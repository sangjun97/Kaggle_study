#MID_run_apc.py
"""실무자용 APC 추천 스크립트.

입력은 run_dev.py 와 동일. 출력:
  - outputs/release_apc/<LOT>_apc_recommendation.csv
    컬럼: position, ps_set_prev, ps_delta_zero, ps_set_proposed
  - 잉곳별 plot (run_dev 동일)
  - 콘솔 출력에 표 형태 요약
"""

import sys
import os
import argparse
from pathlib import Path
import numpy as np
import pandas as pd

sys.path.insert(0, str(Path(__file__).resolve().parents[2]))
from src.core.config import load_config
from src.core.io import read_csv, write_csv, ensure_dir
from src.release.params import load_release_params
from src.release.pipeline import (preprocess_ingot, find_valid_prevs,
                                    build_delta_table, run_inference_for_ingot)
from src.release.postprocess import transform_cuh_prev, calc_per_position
from src.model.slope_io import load_slope_config
from src.preprocess.matching import extract_at_references


APC_CTRL = "pull_speed_t200_delta"


def blend_recommendations(per_prev_results: list[dict], mode: str) -> dict:
    """K개 PREV 별 추천을 blend.

    Args:
        per_prev_results: 각 PREV 의 결과 dict 리스트. 키:
            "ps_prev", "apc_delta_zero", "apc_prof_zero", "ref_pos", "run_delta"
        mode: "weighted" (1/run_delta), "mean", "median"

    Returns:
        blend 된 {ref_pos, ps_prev (rank 1 그대로), ps_delta_zero, ps_set_proposed}
    """
    if not per_prev_results:
        return {}
    if len(per_prev_results) == 1:
        r = per_prev_results[0]
        return {
            "ref_pos": r["ref_pos"],
            "ps_prev": r["ps_prev"],
            "apc_delta_zero": r["apc_delta_zero"],
            "apc_prof_zero": r["apc_prof_zero"],
        }

    # rank 1 (가장 최근 PREV) 의 ref_pos 와 ps_prev 는 그대로 사용
    base = per_prev_results[0]
    ref_pos = base["ref_pos"]
    n_pos = len(ref_pos)

    # 각 PREV 의 추천을 동일 length 의 array 로 정렬 (ref_pos 위치별로)
    delta_arr = []
    prof_arr = []
    weights = []
    for r in per_prev_results:
        # 만약 ref_pos 길이가 base 와 다르면 skip (안전)
        if len(r["apc_delta_zero"]) != n_pos:
            continue
        delta_arr.append(np.asarray(r["apc_delta_zero"], dtype=float))
        prof_arr.append(np.asarray(r["apc_prof_zero"], dtype=float))
        w = 1.0 / max(r["run_delta"], 1) if mode == "weighted" else 1.0
        weights.append(w)

    if not delta_arr:
        return {"ref_pos": ref_pos, "ps_prev": base["ps_prev"],
                "apc_delta_zero": base["apc_delta_zero"],
                "apc_prof_zero": base["apc_prof_zero"]}

    M = np.stack(delta_arr) # (K, n_pos)
    P = np.stack(prof_arr) # (K, n_pos)
    W = np.asarray(weights) # (K,)

    if mode == "median":
        delta_blend = np.nanmedian(M, axis=0)
        prof_blend = np.nanmedian(P, axis=0)
    else:
        # weighted 또는 mean — NaN 제외 가중 평균
        mask = ~np.isnan(M)
        Wm = (W[:, None] * mask).astype(float)
        delta_blend = np.nansum(M * Wm, axis=0) / np.where(Wm.sum(axis=0) == 0, 1, Wm.sum(axis=0))
        prof_blend = np.nansum(P * Wm, axis=0) / np.where(Wm.sum(axis=0) == 0, 1, Wm.sum(axis=0))

    return {
        "ref_pos": ref_pos,
        "ps_prev": base["ps_prev"],
        "apc_delta_zero": delta_blend.tolist(),
        "apc_prof_zero": prof_blend.tolist(),
    }


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--paths-config", default="config/paths.yaml")
    parser.add_argument("--preprocess-config", default="config/preprocess.yaml")
    parser.add_argument("--model-config", default="config/model.yaml")
    parser.add_argument("--release-params", default="config/release_params.csv")
    parser.add_argument("--ingot-csv", required=True)
    parser.add_argument("--grower-dir", required=True)
    parser.add_argument("--file-version", choices=["v1", "v2"], default="v1")
    parser.add_argument("--usage-relaxed", action="store_true")
    parser.add_argument("--skip-customer-filter", action="store_true")
    parser.add_argument("--set-name", default=None,
                        help="모델 SET 이름. 지정 시 config/models_<set>.json 자동 사용")
    # K개 PREV blending 옵션 (학습 시 사용한 5 run 과 일관성)
    parser.add_argument("--blend-prevs", type=int, default=5,
                        help="추론 시 사용할 PREV 후보 수 (기본 5). "
                             "1=가장 최근만, 5=학습 데이터와 일관된 K=5 blend.")
    parser.add_argument("--blend-mode", choices=["weighted", "mean", "median"],
                        default="weighted",
                        help="blend 방식: weighted=run_delta 작을수록 weight ↑ (1/run_delta) [기본], "
                             "mean=단순 평균, median=중앙값.")
    args = parser.parse_args()

    paths = load_config(args.paths_config)
    pcfg = load_config(args.preprocess_config)
    model_cfg = load_config(args.model_config)

    # set-name 분기
    if args.set_name:
        models_path = f"config/models_{args.set_name}.json"
        if not os.path.exists(models_path):
            print(f"❌ {models_path} 없음")
            return
        paths.paths["models_json"] = models_path

    # release_params.csv 적용
    rel_cfg = dict(pcfg.release)
    inf_cfg = dict(model_cfg.inference)
    clip_cfg = dict(model_cfg.clip_config)
    if os.path.exists(args.release_params):
        rp = load_release_params(args.release_params)
        if rp.get("positions"):
            rel_cfg["position_reference"] = rp["positions"]
        if "apc_cuh_target" in rp:
            inf_cfg["apc_cuh_target"] = float(rp["apc_cuh_target"])
        print(f"📋 release_params: {len(rel_cfg['position_reference'])} positions, "
              f"target={inf_cfg['apc_cuh_target']}")

    customer_filter = pcfg.customer_filter_values
    x_cols_raw = model_cfg.data["x_columns"]
    x_cols = list(x_cols_raw[args.file_version]) if isinstance(x_cols_raw, dict) else list(x_cols_raw)

    out_dir = ensure_dir(os.path.join(paths.paths["release_out"] + "_apc"))

    # === 잉곳 로드 + 전처리 ===
    df_ingot = read_csv(args.ingot_csv, normalize=True)
    grower = df_ingot["grower"].value_counts().idxmax()
    pres_lot = df_ingot["lot_number"].value_counts().idxmax()

    customer_value = df_ingot["customer_grp_2"].value_counts().idxmax()
    if customer_value not in customer_filter and not args.skip_customer_filter:
        print(f"[ERROR] customer_grp_2='{customer_value}' 가 대상이 아님")
        return

    df_pres = preprocess_ingot(
        df_ingot,
        filters={"length": pcfg.length, "current_event": rel_cfg["current_event"]},
        params={"position_reference": rel_cfg["position_reference"]},
        preprocess_config=args.preprocess_config,
    )

    path_grower = os.path.join(args.grower_dir, f"{grower}.csv")
    df_grower = read_csv(path_grower)
    pres_date = int(df_pres["gr_out"].iloc[0])

    data_prevs = find_valid_prevs(df_pres, df_grower, pres_date,
                                   rel_cfg["max_delta"],
                                   position_reference=rel_cfg["position_reference"],
                                   usage_strict=not args.usage_relaxed)
    if not data_prevs:
        print(f"⚠️ PREV 후보 없음. APC 산출 불가.")
        return

    # ===== PREV 후보 분리: CUH 측정 완료 (페어용) vs 가장 최근 (ps_set_prev 출처) =====
    def _is_all_cuh_empty(df_ingot: pd.DataFrame) -> bool:
        """잉곳 데이터가 CUH 측정 전 상태인지 (center/edge 둘 다 전부 NaN)."""
        cen = df_ingot.get("cuh_bsw_center")
        edg = df_ingot.get("cuh_bsw_edge")
        if cen is None and edg is None:
            return True
        cen_empty = cen is None or cen.isna().all()
        edg_empty = edg is None or edg.isna().all()
        return cen_empty and edg_empty

    # ps_set_prev 의 출처: 절대 시간상 가장 최근 PREV (CUH 유무 무관)
    ps_source_info = data_prevs[0]
    ps_source_lot = ps_source_info["data"]["lot_number"].value_counts().idxmax()
    ps_source_gr_out = int(ps_source_info["data"]["gr_out"].iloc[0])

    # Delta 페어용: CUH 가 측정 완료된 후보만 (전부 NaN 인 잉곳 제외)
    # + 학습 범위 (run_delta <= max_delta) 안의 후보만
    max_run_delta = rel_cfg["max_delta"]
    cuh_ready_prevs = [p for p in data_prevs
                        if not _is_all_cuh_empty(p["data"])
                        and p.get("usage", 0) <= max_run_delta]
    n_skipped_no_cuh = sum(1 for p in data_prevs if _is_all_cuh_empty(p["data"]))
    n_skipped_out_range = sum(1 for p in data_prevs
                               if not _is_all_cuh_empty(p["data"])
                               and p.get("usage", 0) > max_run_delta)
    if n_skipped_no_cuh > 0:
        print(f" CUH 미측정 PREV 후보 {n_skipped_no_cuh}개 페어 제외 "
              f"(ps_set_prev 는 rank=0 의 PS 사용)")
    if n_skipped_out_range > 0:
        print(f" run_delta > {max_run_delta} 후보 {n_skipped_out_range}개 페어 제외 "
              f"(학습 범위 밖)")

    # ===== K=N PREV blending (K 미만이면 가용한 만큼만 사용) =====
    K_HISTORY = max(args.blend_prevs, 1)
    used_prevs = cuh_ready_prevs[:K_HISTORY]
    if not used_prevs:
        print(f"⚠️ CUH 측정 완료된 PREV 후보 없음. APC 산출 불가.")
        return
    K_ACTUAL = len(used_prevs)
    if K_ACTUAL < K_HISTORY:
        print(f" PREV 가용 후보 {K_ACTUAL}개 (요청 {K_HISTORY}개) → K={K_ACTUAL} 로 blend")

    # 참고용 history 메타
    # rank 0: ps_set_prev 출처 (CUH 무관, 가장 최근)
    # rank 1~K: Delta 페어 (CUH 측정 완료)
    history_meta = []
    ps_source_same_as_rank1 = (ps_source_gr_out == int(used_prevs[0]["data"]["gr_out"].iloc[0]))
    if not ps_source_same_as_rank1:
        # ps 출처가 별도일 때만 rank 0 추가 (CUH 미측정 잉곳)
        history_meta.append({
            "rank": 0,
            "lot_number": ps_source_lot,
            "gr_out": ps_source_gr_out,
            "usage": ps_source_info.get("usage"),
            "role": "ps_set_prev only (CUH 미측정)",
        })
    K_META = max(K_HISTORY, 5)
    for i, info in enumerate(cuh_ready_prevs[:K_META]):
        df_p = info["data"]
        try:
            lot = df_p["lot_number"].value_counts().idxmax()
        except Exception:
            lot = None
        try:
            gr_out = int(df_p["gr_out"].iloc[0])
        except Exception:
            gr_out = None
        role = "blend" if i < K_ACTUAL else "reference"
        if i == 0 and ps_source_same_as_rank1:
            role = "blend + ps_set_prev"
        history_meta.append({
            "rank": i + 1,
            "lot_number": lot,
            "gr_out": gr_out,
            "usage": info.get("usage"),
            "role": role,
        })
    df_history = pd.DataFrame(history_meta)
    print(f"\n=== PREV 후보 (blend={K_ACTUAL}개, mode={args.blend_mode}) ===")
    print(df_history.to_string(index=False))

    pres_ref, _ = extract_at_references(df_pres, rel_cfg["position_reference"])
    slope_config = load_slope_config(paths.paths["models_json"])

    # K개 PREV 각각 inference + per-run 보정
    per_prev_results = []
    for i, prev_info in enumerate(used_prevs):
        df_prev_i = prev_info["data"]
        delta_usage_i = prev_info["usage"]
        rank = i + 1

        prev_ref_i, _ = extract_at_references(df_prev_i, rel_cfg["position_reference"])

        # PREV 잉곳에서 위치별 boundary 컬럼 추출 (이 run 만의 메타)
        meta_rows_i = []
        for ref_pos, df_at in prev_ref_i.items():
            if df_at.empty:
                continue
            r = df_at.iloc[0]
            meta_rows_i.append({
                "ref_length": ref_pos,
                "b_band_value": r.get("b_band_value", ""),
                "fpd_ar": r.get("fpd_ar", ""),
                "ldp_ar": r.get("ldp_ar", ""),
            })
        df_meta_i = pd.DataFrame(meta_rows_i)

        df_delta_i = build_delta_table(pres_ref, prev_ref_i, rel_cfg["position_reference"],
                                        pres_date, int(df_prev_i["gr_out"].iloc[0]),
                                        delta_usage_i)
        if df_delta_i.empty:
            print(f" ⚠️ rank={rank}: Δ 테이블 비어있음 → skip")
            continue
        df_delta_i.attrs["grower"] = grower

        # === Boundary 룰: 모델 입력의 cuh_bsw_all_prev 변환 + per-position 보정값 산출 ===
        position_corrections = {} # {ref_length: correction}
        position_rules = [] # 발동 라벨 (출력용)
        if not df_meta_i.empty:
            meta_idx = df_meta_i.set_index("ref_length")
            for idx, row in df_delta_i.iterrows():
                ref = row.get("ref_length")
                if ref is None or ref not in meta_idx.index:
                    continue
                m = meta_idx.loc[ref]
                if isinstance(m, pd.DataFrame):
                    m = m.iloc[0]
                row_input = {
                    "cuh_bsw_all_prev": row["cuh_bsw_all_prev"],
                    "b_band_value": m["b_band_value"],
                    "fpd_ar": m["fpd_ar"],
                    "ldp_ar": m["ldp_ar"],
                }
                model_input, correction, rule = calc_per_position(row_input)
                position_corrections[ref] = correction
                if rule != "none" or correction != 0.0:
                    position_rules.append((int(ref), rule, correction,
                                           row["cuh_bsw_all_prev"], model_input))
                # 모델 입력의 cuh_prev/delta 갱신 (boundary 발동 OR score>150 clip 시)
                if model_input is not None and model_input != row["cuh_bsw_all_prev"]:
                    df_delta_i.at[idx, "cuh_bsw_all_prev"] = model_input
                    if "cuh_bsw_all_pres" in df_delta_i.columns:
                        df_delta_i.at[idx, "cuh_bsw_all_delta"] = (
                            df_delta_i.at[idx, "cuh_bsw_all_pres"] - model_input
                        )

        # 룰 발동 위치 출력 (rule 와 보정값까지)
        if position_rules:
            print(f"\n rank={rank} 룰 발동:")
            for pos, rule, corr, orig, after in position_rules:
                print(f" {pos}: rule={rule}, score {orig:.1f}→{after:.1f}, "
                      f"correction={corr:+.5f}")

        ingot_data_i = run_inference_for_ingot(
            df_delta_i, slope_config,
            x_columns=x_cols,
            y_column=model_cfg.data["y_column"],
            c_column="cuh_bsw_all_prev",
            apc_control_column=inf_cfg["apc_control_column"],
            clip_config=clip_cfg,
            inference_cfg=inf_cfg,
        )

        # 이 run 의 raw 모델 예측 + 위치별 보정값을 별도 저장 (blend 후 합산)
        ref_pos_i = ingot_data_i["REF_POS"]
        apc_delta_raw = ingot_data_i.get(f"APC_Delta_Zero_{APC_CTRL}", [])
        apc_prof_raw = ingot_data_i.get(f"APC_Profile_Zero_{APC_CTRL}", [])
        ps_prev_raw_i = ingot_data_i.get("X_PREV", [])

        corrections_list = [position_corrections.get(ref, 0.0) for ref in ref_pos_i]

        per_prev_results.append({
            "rank": rank,
            "run_delta": delta_usage_i,
            "ref_pos": ref_pos_i,
            "ps_prev": ps_prev_raw_i,
            "apc_delta_zero": apc_delta_raw, # raw 모델 예측 (보정 전)
            "apc_prof_zero": apc_prof_raw, # raw profile (보정 전)
            "corrections": corrections_list, # 위치별 보정값
        })

    if not per_prev_results:
        print(f"⚠️ 모든 PREV 후보의 추론 실패")
        return

    # blend (raw delta 들의 weighted blend)
    blended = blend_recommendations(per_prev_results, args.blend_mode)
    ref_pos = blended["ref_pos"]
    apc_delta_zero = blended["apc_delta_zero"] # raw blended (보정 전)

    # ps_set_prev: ps_source_info (절대 시간상 가장 최근 잉곳) 의 위치별 PS 사용
    # cuh_ready_prevs[0] 와 다를 수 있음 (CUH 측정 대기 중인 최신 잉곳)
    if ps_source_same_as_rank1:
        ps_prev_raw = blended["ps_prev"]
    else:
        ps_source_ref, _ = extract_at_references(ps_source_info["data"],
                                                  rel_cfg["position_reference"])
        ps_prev_raw = []
        for ref in ref_pos:
            if ref in ps_source_ref and not ps_source_ref[ref].empty:
                val = ps_source_ref[ref].iloc[0].get("pull_speed_t200", None)
                ps_prev_raw.append(float(val) if val is not None and not pd.isna(val) else None)
            else:
                ps_prev_raw.append(None)

    # corrections 도 같은 방식으로 blend
    n_pos = len(ref_pos)
    corr_arrs = []
    weights = []
    for r in per_prev_results:
        if len(r.get("corrections", [])) != n_pos:
            continue
        corr_arrs.append(np.asarray(r["corrections"], dtype=float))
        w = 1.0 / max(r["run_delta"], 1) if args.blend_mode == "weighted" else 1.0
        weights.append(w)
    if corr_arrs:
        C = np.stack(corr_arrs)
        W = np.asarray(weights)
        if args.blend_mode == "median":
            corr_blend = np.median(C, axis=0)
        else:
            corr_blend = (C * W[:, None]).sum(axis=0) / W.sum()
    else:
        corr_blend = np.zeros(n_pos)

    # ps_set_proposed = ps_set_prev + ps_delta_zero (raw blended) + ps_correction_value
    ps_set_proposed = []
    for j in range(n_pos):
        prev_v = ps_prev_raw[j] if ps_prev_raw and j < len(ps_prev_raw) else None
        delta_v = apc_delta_zero[j] if apc_delta_zero and j < len(apc_delta_zero) else None
        corr_v = corr_blend[j]
        if prev_v is not None and delta_v is not None:
            ps_set_proposed.append(prev_v + delta_v + corr_v)
        else:
            ps_set_proposed.append(None)

    df_out = pd.DataFrame({
        "position": ref_pos,
        "ps_set_prev": ps_prev_raw if ps_prev_raw else [None] * n_pos,
        "ps_delta_zero": apc_delta_zero if apc_delta_zero else [None] * n_pos,
        "ps_correction_value": corr_blend.tolist(),
        "ps_set_proposed": ps_set_proposed,
    })

    # 콘솔 표 출력
    df_history_show = df_history[["rank", "lot_number", "gr_out", "role"]].copy()
    print(f"\n=== APC 추천 (Zero baseline, target={inf_cfg['apc_cuh_target']}) ===")
    print(f"grower: {grower}")
    print(f"pres_lot: {pres_lot}")
    print(f"prev (K={len(per_prev_results)} {args.blend_mode}):")
    print(df_history_show.to_string(index=False))
    print(f"\n결과:")
    print(df_out.to_string(index=False, float_format=lambda x: f"{x:.5f}"))

    # CSV 저장 (단순화: 헤더 + 결과 표만)
    out_csv = os.path.join(str(out_dir), f"{pres_lot}_apc_recommendation.csv")
    with open(out_csv, "w", encoding="utf-8-sig") as f:
        f.write(f"# grower: {grower}\n")
        f.write(f"# pres_lot: {pres_lot}\n")
        f.write(f"# prev_history (K={len(per_prev_results)} {args.blend_mode}):\n")
        for _, h in df_history.iterrows():
            f.write(f"# rank={h['rank']}, lot={h['lot_number']}, "
                    f"gr_out={h['gr_out']}, role={h['role']}\n")
        df_out.to_csv(f, index=False, float_format="%.5f")
    print(f"\n✅ 저장: {out_csv}")


if __name__ == "__main__":
    main()




# LOW_run_apc.py
"""실무자용 APC 추천 스크립트.

입력은 run_dev.py 와 동일. 출력:
  - outputs/release_apc/<LOT>_apc_recommendation.csv
    컬럼: position, ps_set_prev, ps_delta_zero, ps_set_proposed
  - 잉곳별 plot (run_dev 동일)
  - 콘솔 출력에 표 형태 요약
"""

import sys
import os
import argparse
from pathlib import Path
import numpy as np
import pandas as pd

sys.path.insert(0, str(Path(__file__).resolve().parents[2]))
from src.core.config import load_config
from src.core.io import read_csv, write_csv, ensure_dir
from src.release.params import load_release_params
from src.release.pipeline import (preprocess_ingot, find_valid_prevs,
                                    build_delta_table, run_inference_for_ingot)
from src.release.postprocess import transform_cuh_prev, calc_per_position
from src.model.slope_io import load_slope_config
from src.preprocess.matching import extract_at_references


def _save_apc_xlsx(out_path, grower, pres_lot, blend_mode,
                   df_history, df_out):
    """APC 결과를 보기 좋은 xlsx 로 저장 (열 너비 자동, 헤더 스타일, 숫자 포맷)."""
    from openpyxl import Workbook
    from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
    from openpyxl.utils import get_column_letter

    wb = Workbook()
    ws = wb.active
    ws.title = "APC"

    hdr_font = Font(name="Arial", bold=True, color="FFFFFF")
    hdr_fill = PatternFill("solid", fgColor="305496")
    sub_fill = PatternFill("solid", fgColor="D6DCE4")
    base_font = Font(name="Arial")
    center = Alignment(horizontal="center", vertical="center")
    thin = Side(style="thin", color="BFBFBF")
    border = Border(left=thin, right=thin, top=thin, bottom=thin)

    r = 1
    # 메타 헤더
    for label, val in [("grower", grower), ("pres_lot", pres_lot),
                       ("blend_mode", blend_mode)]:
        ws.cell(r, 1, label).font = Font(name="Arial", bold=True)
        ws.cell(r, 2, val).font = base_font
        r += 1
    r += 1

    # PREV history 블록
    ws.cell(r, 1, "PREV history").font = Font(name="Arial", bold=True, size=12)
    r += 1
    hist_cols = ["rank", "lot_number", "gr_out", "run_delta", "비율(%)", "role"]
    hist_src = df_history.rename(columns={"usage": "run_delta"})
    for c, name in enumerate(hist_cols, 1):
        cell = ws.cell(r, c, name)
        cell.font, cell.fill, cell.alignment, cell.border = hdr_font, hdr_fill, center, border
    r += 1
    hist_start = r
    for _, row in hist_src.iterrows():
        for c, name in enumerate(hist_cols, 1):
            cell = ws.cell(r, c, row.get(name))
            cell.font, cell.alignment, cell.border = base_font, center, border
        r += 1
    r += 1

    # 결과 표 블록
    ws.cell(r, 1, "결과 (APC 추천)").font = Font(name="Arial", bold=True, size=12)
    r += 1
    res_start = r
    cols = list(df_out.columns)
    for c, name in enumerate(cols, 1):
        cell = ws.cell(r, c, name)
        cell.font, cell.fill, cell.alignment, cell.border = hdr_font, hdr_fill, center, border
    r += 1
    ps_cols = {"ps_set_pres", "ps_set_proposed", "Target P/S",
               "ps_delta_pres", "ps_correction_value"}
    for _, row in df_out.iterrows():
        for c, name in enumerate(cols, 1):
            v = row[name]
            v = None if (isinstance(v, float) and pd.isna(v)) else v
            cell = ws.cell(r, c, v)
            cell.font, cell.alignment, cell.border = base_font, center, border
            if name in ps_cols:
                cell.number_format = "0.00000"
            elif name == "position":
                cell.number_format = "0"
            elif name.startswith("cuh_r"):
                cell.number_format = "0.0"
        r += 1

    # 열 너비 자동맞춤 (헤더/값 최대 길이 기준)
    for c in range(1, len(cols) + 1):
        letter = get_column_letter(c)
        maxlen = 0
        for row_cells in ws.iter_rows(min_col=c, max_col=c):
            for cell in row_cells:
                if cell.value is not None:
                    maxlen = max(maxlen, len(str(cell.value)))
        ws.column_dimensions[letter].width = min(max(maxlen + 3, 10), 28)

    ws.freeze_panes = ws.cell(res_start + 1, 1) # 결과 헤더 고정
    wb.save(out_path)


# APC 제어 델타 컬럼은 model.yaml 의 inference.apc_control_column 에서
# 함수 내 apc_ctrl 로 구성한다 (t100/t200 전환을 config 로만 제어).


def blend_recommendations(per_prev_results: list[dict], mode: str) -> dict:
    """K개 PREV 별 추천을 '위치(ref_pos) 기준' 으로 정렬 후 blend.

    각 PREV 가 커버하는 위치가 달라도, 같은 위치끼리만 모아 blend 한다.
    위치마다 참여 run 수가 다를 수 있음 (그 위치를 가진 run 만 참여).

    Args:
        per_prev_results: 각 PREV 의 결과 dict. 키:
            "ref_pos", "ps_prev", "apc_delta_zero", "apc_prof_zero", "run_delta"
        mode: "weighted" (1/run_delta), "mean", "median"

    Returns:
        {ref_pos, ps_prev, apc_delta_zero, apc_prof_zero}
        ref_pos 는 모든 PREV 위치의 합집합 (오름차순).
    """
    if not per_prev_results:
        return {}
    if len(per_prev_results) == 1:
        r = per_prev_results[0]
        return {
            "ref_pos": r["ref_pos"],
            "ps_prev": r["ps_prev"],
            "apc_delta_zero": r["apc_delta_zero"],
            "apc_prof_zero": r["apc_prof_zero"],
        }

    # 위치별 수집: ref_pos → list of (delta, prof, weight, ps_prev)
    by_pos: dict[float, dict] = {}
    for r in per_prev_results:
        rp = r["ref_pos"]
        d_arr = r["apc_delta_zero"]
        p_arr = r["apc_prof_zero"]
        ps_arr = r.get("ps_prev", [None] * len(rp))
        w = 1.0 / max(r["run_delta"], 1) if mode == "weighted" else 1.0
        # rank (per_prev_results 순서) 가 작을수록 최신 → ps_prev 우선권
        for j, pos in enumerate(rp):
            if j >= len(d_arr) or j >= len(p_arr):
                continue
            dv, pv = d_arr[j], p_arr[j]
            slot = by_pos.setdefault(pos, {"delta": [], "prof": [], "w": [],
                                           "ps_prev": None})
            slot["delta"].append(float(dv) if dv is not None else np.nan)
            slot["prof"].append(float(pv) if pv is not None else np.nan)
            slot["w"].append(w)
            # ps_prev: 가장 먼저 등장한 run (= 최신 rank) 값 채택
            if slot["ps_prev"] is None and j < len(ps_arr) and ps_arr[j] is not None:
                slot["ps_prev"] = ps_arr[j]

    ref_pos = sorted(by_pos.keys())
    delta_blend, prof_blend, ps_prev_out = [], [], []
    for pos in ref_pos:
        slot = by_pos[pos]
        d = np.asarray(slot["delta"], dtype=float)
        p = np.asarray(slot["prof"], dtype=float)
        w = np.asarray(slot["w"], dtype=float)

        if mode == "median":
            db = np.nanmedian(d)
            pb = np.nanmedian(p)
        else:
            dm = ~np.isnan(d)
            pm = ~np.isnan(p)
            wd = (w * dm)
            wp = (w * pm)
            db = np.nansum(d * wd) / (wd.sum() if wd.sum() else 1.0)
            pb = np.nansum(p * wp) / (wp.sum() if wp.sum() else 1.0)

        delta_blend.append(db)
        prof_blend.append(pb)
        ps_prev_out.append(slot["ps_prev"])

    return {
        "ref_pos": ref_pos,
        "ps_prev": ps_prev_out,
        "apc_delta_zero": delta_blend,
        "apc_prof_zero": prof_blend,
    }


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--paths-config", default="config/paths.yaml")
    parser.add_argument("--preprocess-config", default="config/preprocess.yaml")
    parser.add_argument("--model-config", default="config/model.yaml")
    parser.add_argument("--release-params", default="config/release_params.csv")
    parser.add_argument("--ingot-csv", required=True)
    parser.add_argument("--grower-dir", required=True)
    parser.add_argument("--file-version", choices=["v1", "v2"], default="v1")
    parser.add_argument("--usage-relaxed", action="store_true")
    parser.add_argument("--skip-customer-filter", action="store_true")
    parser.add_argument("--set-name", default=None,
                        help="모델 SET 이름. 지정 시 config/models_<set>.json 자동 사용")
    # K개 PREV blending 옵션 (학습 시 사용한 5 run 과 일관성)
    parser.add_argument("--blend-prevs", type=int, default=5,
                        help="추론 시 사용할 PREV 후보 수 (기본 5). "
                             "1=가장 최근만, 5=학습 데이터와 일관된 K=5 blend.")
    parser.add_argument("--blend-mode", choices=["weighted", "mean", "median"],
                        default="weighted",
                        help="blend 방식: weighted=run_delta 작을수록 weight ↑ (1/run_delta) [기본], "
                             "mean=단순 평균, median=중앙값.")
    args = parser.parse_args()

    paths = load_config(args.paths_config)
    pcfg = load_config(args.preprocess_config)
    model_cfg = load_config(args.model_config)

    # set-name 분기
    if args.set_name:
        models_path = f"config/models_{args.set_name}.json"
        if not os.path.exists(models_path):
            print(f"❌ {models_path} 없음")
            return
        paths.paths["models_json"] = models_path

    # release_params.csv 적용
    rel_cfg = dict(pcfg.release)
    inf_cfg = dict(model_cfg.inference)
    clip_cfg = dict(model_cfg.clip_config)
    if os.path.exists(args.release_params):
        rp = load_release_params(args.release_params)
        if rp.get("positions"):
            rel_cfg["position_reference"] = rp["positions"]
        if "apc_cuh_target" in rp:
            inf_cfg["apc_cuh_target"] = float(rp["apc_cuh_target"])
        print(f"📋 release_params: {len(rel_cfg['position_reference'])} positions, "
              f"target={inf_cfg['apc_cuh_target']}")

    customer_filter = pcfg.customer_filter_values
    x_cols_raw = model_cfg.data["x_columns"]
    x_cols = list(x_cols_raw[args.file_version]) if isinstance(x_cols_raw, dict) else list(x_cols_raw)

    out_dir = ensure_dir(os.path.join(paths.paths["release_out"] + "_apc"))

    # === 잉곳 파일 로드 + (grower, lot_number) 분리 ===
    # raw_daily csv 는 여러 grower × 여러 lot 을 포함할 수 있으므로 잉곳 단위로 분리
    df_raw = read_csv(args.ingot_csv, normalize=True)
    if "grower" not in df_raw.columns or "lot_number" not in df_raw.columns:
        print(f"[ERROR] grower 또는 lot_number 컬럼 없음")
        return

    # 잉곳 단위 그룹핑 (grower, lot_number) - 데이터 행 수 기준 내림차순
    ingot_groups = []
    for (grower_id, lot_num), df_one in df_raw.groupby(["grower", "lot_number"]):
        if pd.isna(grower_id) or pd.isna(lot_num):
            continue
        ingot_groups.append((str(grower_id), str(lot_num), df_one.copy()))
    # 데이터 많은 잉곳 우선 (가장 정상적으로 자란 잉곳)
    ingot_groups.sort(key=lambda x: len(x[2]), reverse=True)

    n_ingots = len(ingot_groups)
    print(f"\n{'='*70}")
    print(f"📦 입력 파일에서 {n_ingots} 개 잉곳 발견")
    for i, (g, l, df_i) in enumerate(ingot_groups, 1):
        print(f" [{i}/{n_ingots}] grower={g}, lot={l}, 행수={len(df_i)}")
    print(f"{'='*70}")

    n_success = 0
    n_failed = 0
    for i, (grower, pres_lot, df_ingot) in enumerate(ingot_groups, 1):
        print(f"\n{'#'*70}")
        print(f"# [{i}/{n_ingots}] grower={grower}, pres_lot={pres_lot}")
        print(f"{'#'*70}")
        try:
            ok = _process_single_ingot(
                df_ingot, grower, pres_lot,
                args, paths, pcfg, model_cfg,
                rel_cfg, inf_cfg, clip_cfg,
                customer_filter, x_cols, out_dir
            )
            if ok:
                n_success += 1
            else:
                n_failed += 1
        except Exception as e:
            print(f"\n[ERROR] grower={grower}, lot={pres_lot} 처리 실패: {e}")
            import traceback
            traceback.print_exc()
            n_failed += 1

    print(f"\n{'='*70}")
    print(f"📊 처리 완료: 성공 {n_success}, 실패 {n_failed}, 전체 {n_ingots}")
    print(f"{'='*70}")


def _process_single_ingot_with_grower(df_ingot, grower, pres_lot, df_grower,
                                      args, paths, pcfg, model_cfg,
                                      rel_cfg, inf_cfg, clip_cfg,
                                      customer_filter, x_cols, out_dir) -> bool:
    """lot 단위 APC 용 wrapper: df_grower 를 외부(5 run)에서 주입.

    run_apc_by_lot.py 가 호출. 기존 _process_single_ingot 에 df_grower 만 전달.
    """
    return _process_single_ingot(
        df_ingot, grower, pres_lot,
        args, paths, pcfg, model_cfg,
        rel_cfg, inf_cfg, clip_cfg,
        customer_filter, x_cols, out_dir,
        df_grower=df_grower,
    )


def _process_single_ingot(df_ingot, grower, pres_lot,
                          args, paths, pcfg, model_cfg,
                          rel_cfg, inf_cfg, clip_cfg,
                          customer_filter, x_cols, out_dir,
                          df_grower=None) -> bool:
    """단일 잉곳 APC 처리. 성공 시 True 반환."""
    # apc_control_column 은 이미 완전한 델타 컬럼명 (예: pull_speed_t200_delta).
    # run_inference_for_ingot 이 APC_Delta_Zero_{apc_control_column} 키를 만들므로
    # apc_ctrl 은 그 값을 그대로 사용. base 는 _delta 를 떼서 raw PS 컬럼명.
    apc_ctrl = inf_cfg["apc_control_column"] # pull_speed_t200_delta
    apc_ctrl_base = apc_ctrl[:-6] if apc_ctrl.endswith("_delta") else apc_ctrl # pull_speed_t200

    customer_value = df_ingot["customer_grp_2"].value_counts().idxmax()
    if customer_value not in customer_filter and not args.skip_customer_filter:
        print(f"[SKIP] customer_grp_2='{customer_value}' 가 대상이 아님")
        return False

    df_pres = preprocess_ingot(
        df_ingot,
        filters={"length": pcfg.length, "current_event": rel_cfg["current_event"]},
        params={"position_reference": rel_cfg["position_reference"]},
        preprocess_config=args.preprocess_config,
        skip_validation=True, # PRES 잉곳은 아직 측정 안 됨 (fpd_ar/cuh_* NaN)
    )

    if df_pres.empty:
        print(f"\n[ERROR] preprocess_ingot 후 빈 DataFrame")
        print(f" 진단:")
        print(f" - 원본 행 수: {len(df_ingot)}")
        print(f" - length 범위 필터: [{pcfg.length['min']}, {pcfg.length['max']}]")
        print(f" - current_event 필터: '{rel_cfg['current_event']}'")
        print(f" - position_reference: {rel_cfg['position_reference']}")
        if len(df_ingot) > 0:
            print(f" - 원본 length 범위: "
                  f"[{df_ingot['length'].min():.0f}, {df_ingot['length'].max():.0f}]")
            if "current_event" in df_ingot.columns:
                events = df_ingot["current_event"].value_counts().head(3).to_dict()
                print(f" - 원본 current_event: {events}")
            # filter_valid_data 의 컬럼 상태 점검 (PRES 잉곳에서 NaN 가능성)
            for col in ["fpd_ar", "ldp_ar", "cuh_bp_center", "cuh_bsw_center"]:
                if col in df_ingot.columns:
                    n_na = df_ingot[col].isna().sum()
                    n_empty = (df_ingot[col].astype(str).str.strip() == "").sum()
                    sample = df_ingot[col].dropna().head(3).tolist()
                    print(f" - {col}: NaN={n_na}, 빈문자열={n_empty}, 샘플={sample}")
        print(f" 원인: filter_valid_data 의 컬럼 (fpd_ar/ldp_ar/cuh_*) 이 모두 NaN 이거나")
        print(f" 기대값과 다른 것 같습니다. PRES 잉곳은 아직 측정 안 된 상태라")
        print(f" 이 필터를 PRES 에 적용하면 모두 제거됩니다.")
        return False

    # df_grower 가 주입되면(lot 단위 APC) 파일 로드 생략, 아니면 기존 grower-dir 사용
    if df_grower is None:
        path_grower = os.path.join(args.grower_dir, f"{grower}.csv")
        df_grower = read_csv(path_grower)
    pres_date = int(df_pres["gr_out"].iloc[0])

    data_prevs = find_valid_prevs(df_pres, df_grower, pres_date,
                                   rel_cfg["max_delta"],
                                   position_reference=rel_cfg["position_reference"],
                                   usage_strict=not args.usage_relaxed)
    if not data_prevs:
        print(f"⚠️ PREV 후보 없음. APC 산출 불가.")
        return False

    # ===== PREV 후보 분리: CUH 측정 완료 (페어용) vs 가장 최근 (ps_set_prev 출처) =====
    def _is_all_cuh_empty(df_ingot: pd.DataFrame) -> bool:
        """잉곳 데이터가 CUH 측정 전 상태인지 (center/edge 둘 다 전부 NaN)."""
        cen = df_ingot.get("cuh_bsw_center")
        edg = df_ingot.get("cuh_bsw_edge")
        if cen is None and edg is None:
            return True
        cen_empty = cen is None or cen.isna().all()
        edg_empty = edg is None or edg.isna().all()
        return cen_empty and edg_empty

    # ps_set_prev 의 출처: 절대 시간상 가장 최근 PREV (CUH 유무 무관)
    ps_source_info = data_prevs[0]
    ps_source_lot = ps_source_info["data"]["lot_number"].value_counts().idxmax()
    ps_source_gr_out = int(ps_source_info["data"]["gr_out"].iloc[0])

    # Delta 페어용: CUH 가 측정 완료된 후보만 (전부 NaN 인 잉곳 제외)
    # + 학습 범위 (run_delta <= max_delta) 안의 후보만
    max_run_delta = rel_cfg["max_delta"]
    cuh_ready_prevs = [p for p in data_prevs
                        if not _is_all_cuh_empty(p["data"])
                        and p.get("usage", 0) <= max_run_delta]
    n_skipped_no_cuh = sum(1 for p in data_prevs if _is_all_cuh_empty(p["data"]))
    n_skipped_out_range = sum(1 for p in data_prevs
                               if not _is_all_cuh_empty(p["data"])
                               and p.get("usage", 0) > max_run_delta)
    if n_skipped_no_cuh > 0:
        print(f" CUH 미측정 PREV 후보 {n_skipped_no_cuh}개 페어 제외 "
              f"(ps_set_prev 는 rank=0 의 PS 사용)")
    if n_skipped_out_range > 0:
        print(f" run_delta > {max_run_delta} 후보 {n_skipped_out_range}개 페어 제외 "
              f"(학습 범위 밖)")

    # ===== K=N PREV blending (K 미만이면 가용한 만큼만 사용) =====
    K_HISTORY = max(args.blend_prevs, 1)
    used_prevs = cuh_ready_prevs[:K_HISTORY]
    if not used_prevs:
        print(f"⚠️ CUH 측정 완료된 PREV 후보 없음. APC 산출 불가.")
        return False
    K_ACTUAL = len(used_prevs)
    if K_ACTUAL < K_HISTORY:
        print(f" PREV 가용 후보 {K_ACTUAL}개 (요청 {K_HISTORY}개) → K={K_ACTUAL} 로 blend")

    # 참고용 history 메타
    # rank 0: ps_set_prev 출처 (CUH 무관, 가장 최근)
    # rank 1~K: Delta 페어 (CUH 측정 완료)
    history_meta = []
    ps_source_same_as_rank1 = (ps_source_gr_out == int(used_prevs[0]["data"]["gr_out"].iloc[0]))
    if not ps_source_same_as_rank1:
        # ps 출처가 별도일 때만 rank 0 추가 (CUH 미측정 잉곳)
        history_meta.append({
            "rank": 0,
            "lot_number": ps_source_lot,
            "gr_out": ps_source_gr_out,
            "usage": ps_source_info.get("usage"),
            "role": "ps_set_prev only (CUH 미측정)",
        })
    K_META = max(K_HISTORY, 5)
    for i, info in enumerate(cuh_ready_prevs[:K_META]):
        df_p = info["data"]
        try:
            lot = df_p["lot_number"].value_counts().idxmax()
        except Exception:
            lot = None
        try:
            gr_out = int(df_p["gr_out"].iloc[0])
        except Exception:
            gr_out = None
        role = "blend" if i < K_ACTUAL else "reference"
        if i == 0 and ps_source_same_as_rank1:
            role = "blend + ps_set_prev"
        history_meta.append({
            "rank": i + 1,
            "lot_number": lot,
            "gr_out": gr_out,
            "usage": info.get("usage"),
            "role": role,
        })
    df_history = pd.DataFrame(history_meta)

    # blend 비율 계산 → df_history 에 컬럼으로 추가 (별도 표 없이)
    blend_mask = df_history["role"].str.contains("blend", na=False)
    ratios = {}
    if args.blend_mode == "weighted":
        raw = [(idx, 1.0 / max(row["usage"] or 1, 1))
               for idx, row in df_history[blend_mask].iterrows()]
        tot = sum(w for _, w in raw) or 1.0
        ratios = {idx: w / tot * 100 for idx, w in raw}
    elif args.blend_mode == "mean":
        n = int(blend_mask.sum()) or 1
        ratios = {idx: 100.0 / n for idx in df_history[blend_mask].index}
    # median 은 비율 개념이 없어 '-'
    df_history["비율(%)"] = [
        (round(ratios[idx], 1) if idx in ratios else "-")
        for idx in df_history.index
    ]

    print(f"\n=== PREV 후보 (blend={K_ACTUAL}개, mode={args.blend_mode}) ===")
    print(df_history.to_string(index=False))

    pres_ref, _ = extract_at_references(df_pres, rel_cfg["position_reference"])
    slope_config = load_slope_config(paths.paths["models_json"])

    # K개 PREV 각각 inference + per-run 보정
    per_prev_results = []
    for i, prev_info in enumerate(used_prevs):
        df_prev_i = prev_info["data"]
        delta_usage_i = prev_info["usage"]
        rank = i + 1

        prev_ref_i, _ = extract_at_references(df_prev_i, rel_cfg["position_reference"])

        # PREV 잉곳에서 위치별 boundary 컬럼 추출 (이 run 만의 메타)
        meta_rows_i = []
        for ref_pos, df_at in prev_ref_i.items():
            if df_at.empty:
                continue
            r = df_at.iloc[0]
            meta_rows_i.append({
                "ref_length": ref_pos,
                "b_band_value": r.get("b_band_value", ""),
                "fpd_ar": r.get("fpd_ar", ""),
                "ldp_ar": r.get("ldp_ar", ""),
            })
        df_meta_i = pd.DataFrame(meta_rows_i)

        df_delta_i = build_delta_table(pres_ref, prev_ref_i, rel_cfg["position_reference"],
                                        pres_date, int(df_prev_i["gr_out"].iloc[0]),
                                        delta_usage_i,
                                        bp_override=bool(getattr(pcfg, "bp_override", False)))
        if df_delta_i.empty:
            print(f" ⚠️ rank={rank}: Δ 테이블 비어있음 → skip")
            continue
        df_delta_i.attrs["grower"] = grower

        # === Boundary 룰: 모델 입력의 cuh_bsw_all_prev 변환 + per-position 보정값 산출 ===
        # 주의: cuh NaN 제거를 boundary 변환 '뒤'에 한다.
        # R/Y boundary 위치는 cuh 가 NaN 이어도 transform 이 score 0/150 으로
        # 채우므로, 변환 후엔 유효값이 되어 살아남는다. 정상(rule=none)인데
        # cuh NaN 인 위치(TAIL 미측정)만 이후 제거된다.
        position_corrections = {} # {ref_length: correction}
        position_rules = [] # 발동 라벨 (출력용)
        if not df_meta_i.empty:
            meta_idx = df_meta_i.set_index("ref_length")
            for idx, row in df_delta_i.iterrows():
                ref = row.get("ref_length")
                if ref is None or ref not in meta_idx.index:
                    continue
                m = meta_idx.loc[ref]
                if isinstance(m, pd.DataFrame):
                    m = m.iloc[0]
                row_input = {
                    "cuh_bsw_all_prev": row["cuh_bsw_all_prev"],
                    "b_band_value": m["b_band_value"],
                    "fpd_ar": m["fpd_ar"],
                    "ldp_ar": m["ldp_ar"],
                }
                model_input, correction, rule = calc_per_position(row_input)
                position_corrections[ref] = correction
                if rule != "none" or correction != 0.0:
                    position_rules.append((int(ref), rule, correction,
                                           row["cuh_bsw_all_prev"], model_input))
                # 모델 입력의 cuh_prev/delta 갱신 (boundary 발동 OR score>150 clip 시)
                # R/Y 면 model_input 이 0/150 으로 채워져 NaN 이 아니게 됨.
                if model_input is not None and model_input != row["cuh_bsw_all_prev"]:
                    df_delta_i.at[idx, "cuh_bsw_all_prev"] = model_input
                    if "cuh_bsw_all_pres" in df_delta_i.columns:
                        df_delta_i.at[idx, "cuh_bsw_all_delta"] = (
                            df_delta_i.at[idx, "cuh_bsw_all_pres"] - model_input
                        )

        # boundary 변환 후에도 cuh_bsw_all_prev 가 NaN 인 위치 제거.
        # = 정상(rule=none)인데 측정 안 된 위치 (예: TAIL 1590~1990).
        # R/Y 위치는 위에서 0/150 으로 채워졌으므로 보존된다.
        if "cuh_bsw_all_prev" in df_delta_i.columns:
            before = len(df_delta_i)
            df_delta_i = df_delta_i[
                pd.to_numeric(df_delta_i["cuh_bsw_all_prev"], errors="coerce").notna()
            ].reset_index(drop=True)
            dropped = before - len(df_delta_i)
            if dropped > 0:
                print(f" rank={rank}: cuh 없는 정상 위치 {dropped}개 제외")
            if df_delta_i.empty:
                print(f" ⚠️ rank={rank}: 유효 위치 없음 → skip")
                continue

        # 룰 발동 위치 출력 (rule 와 보정값까지)
        if position_rules:
            print(f"\n rank={rank} 룰 발동:")
            for pos, rule, corr, orig, after in position_rules:
                print(f" {pos}: rule={rule}, score {orig:.1f}→{after:.1f}, "
                      f"correction={corr:+.5f}")

        ingot_data_i = run_inference_for_ingot(
            df_delta_i, slope_config,
            x_columns=x_cols,
            y_column=model_cfg.data["y_column"],
            c_column="cuh_bsw_all_prev",
            apc_control_column=inf_cfg["apc_control_column"],
            clip_config=clip_cfg,
            inference_cfg=inf_cfg,
        )

        # 이 run 의 raw 모델 예측 + 위치별 보정값을 별도 저장 (blend 후 합산)
        ref_pos_i = ingot_data_i["REF_POS"]
        apc_delta_raw = ingot_data_i.get(f"APC_Delta_Zero_{apc_ctrl}", [])
        apc_prof_raw = ingot_data_i.get(f"APC_Profile_Zero_{apc_ctrl}", [])
        ps_prev_raw_i = ingot_data_i.get("X_PREV", [])

        corrections_list = [position_corrections.get(ref, 0.0) for ref in ref_pos_i]

        prev_lot_i = (df_prev_i["lot_number"].value_counts().idxmax()
                      if "lot_number" in df_prev_i.columns else f"rank{rank}")

        per_prev_results.append({
            "rank": rank,
            "lot": prev_lot_i,
            "run_delta": delta_usage_i,
            "ref_pos": ref_pos_i,
            "ps_prev": ps_prev_raw_i,
            "apc_delta_zero": apc_delta_raw, # raw 모델 예측 (보정 전)
            "apc_prof_zero": apc_prof_raw, # raw profile (보정 전)
            "corrections": corrections_list, # 위치별 보정값
            "cuh_prev": ingot_data_i.get("Y_PREV", []), # 위치별 cuh (참고용)
        })

    if not per_prev_results:
        print(f"⚠️ 모든 PREV 후보의 추론 실패")
        return False

    # blend (raw delta 들의 weighted blend)
    blended = blend_recommendations(per_prev_results, args.blend_mode)
    ref_pos = blended["ref_pos"]
    apc_delta_zero = blended["apc_delta_zero"] # raw blended (보정 전)

    # ps_set_prev: ps_source_info (절대 시간상 가장 최근 잉곳) 의 위치별 PS 사용
    # cuh_ready_prevs[0] 와 다를 수 있음 (CUH 측정 대기 중인 최신 잉곳)
    if ps_source_same_as_rank1:
        ps_prev_raw = blended["ps_prev"]
    else:
        ps_source_ref, _ = extract_at_references(ps_source_info["data"],
                                                  rel_cfg["position_reference"])
        ps_prev_raw = []
        for ref in ref_pos:
            if ref in ps_source_ref and not ps_source_ref[ref].empty:
                val = ps_source_ref[ref].iloc[0].get(apc_ctrl_base, None)
                ps_prev_raw.append(float(val) if val is not None and not pd.isna(val) else None)
            else:
                ps_prev_raw.append(None)

    # corrections 도 위치(ref_pos) 기준으로 blend (blend_recommendations 와 동일 정렬)
    n_pos = len(ref_pos)
    pos_index = {pos: j for j, pos in enumerate(ref_pos)}
    corr_sum = np.zeros(n_pos)
    corr_w = np.zeros(n_pos)
    corr_vals: list[list[float]] = [[] for _ in range(n_pos)] # median 용
    for r in per_prev_results:
        rp = r["ref_pos"]
        carr = r.get("corrections", [])
        w = 1.0 / max(r["run_delta"], 1) if args.blend_mode == "weighted" else 1.0
        for k, pos in enumerate(rp):
            if pos not in pos_index or k >= len(carr):
                continue
            j = pos_index[pos]
            cv = float(carr[k])
            corr_sum[j] += cv * w
            corr_w[j] += w
            corr_vals[j].append(cv)

    if args.blend_mode == "median":
        corr_blend = np.array([np.median(v) if v else 0.0 for v in corr_vals])
    else:
        corr_blend = np.where(corr_w > 0, corr_sum / np.where(corr_w == 0, 1, corr_w), 0.0)

    # ps_set_proposed = ps_set_prev + ps_delta_zero (raw blended) + ps_correction_value
    ps_set_proposed = []
    for j in range(n_pos):
        prev_v = ps_prev_raw[j] if ps_prev_raw and j < len(ps_prev_raw) else None
        delta_v = apc_delta_zero[j] if apc_delta_zero and j < len(apc_delta_zero) else None
        corr_v = corr_blend[j]
        if prev_v is not None and delta_v is not None:
            ps_set_proposed.append(prev_v + delta_v + corr_v)
        else:
            ps_set_proposed.append(None)

    # 타겟(PRES) 잉곳의 위치별 실측 PS (비교 기준)
    ps_pres_raw = []
    target_ps_raw = [] # actual_pull_speed_set = actual_pull_speed - actual_pull_speed_gap (raw)
    for ref in ref_pos:
        if ref in pres_ref and not pres_ref[ref].empty:
            row0 = pres_ref[ref].iloc[0]
            v = row0.get(apc_ctrl_base, None)
            ps_pres_raw.append(float(v) if v is not None and not pd.isna(v) else None)
            tv = row0.get("actual_pull_speed_set", None)
            target_ps_raw.append(float(tv) if tv is not None and not pd.isna(tv) else None)
        else:
            ps_pres_raw.append(None)
            target_ps_raw.append(None)

    # 표시용 델타 = 추천 - 타겟실측 (PRES 기준). 모델 계산은 그대로 PREV 기준.
    ps_delta_pres = []
    for j in range(n_pos):
        prop = ps_set_proposed[j]
        pres = ps_pres_raw[j]
        if prop is not None and pres is not None:
            ps_delta_pres.append(prop - pres)
        else:
            ps_delta_pres.append(None)

    df_out = pd.DataFrame({
        "position": ref_pos,
        "ps_set_pres": ps_pres_raw, # 타겟 lot 실측 PS (기준)
        "ps_delta_pres": ps_delta_pres, # 추천 - 타겟 (PRES 기준 변화량)
        "ps_correction_value": corr_blend.tolist(),
        "ps_set_proposed": ps_set_proposed, # 최종 추천값
        "Target P/S": target_ps_raw, # actual_pull_speed - gap (raw)
    })

    # 참고: 위치별 이전 5 run 의 CUH score (실무자 판단 참고용)
    pos_index_full = {pos: j for j, pos in enumerate(ref_pos)}
    for r in per_prev_results:
        col = f"cuh_r{r['rank']}({r['lot']})"
        vals = [None] * n_pos
        cuh_arr = r.get("cuh_prev", [])
        for k, pos in enumerate(r["ref_pos"]):
            if pos in pos_index_full and k < len(cuh_arr):
                cv = cuh_arr[k]
                vals[pos_index_full[pos]] = (round(float(cv), 1)
                                             if cv is not None and not pd.isna(cv) else None)
        df_out[col] = vals

    # 콘솔 표 출력 (history 에 run_delta=usage + 비율 포함)
    df_history_show = df_history[["rank", "lot_number", "gr_out",
                                  "usage", "비율(%)", "role"]].copy()
    df_history_show = df_history_show.rename(columns={"usage": "run_delta"})
    print(f"\n=== APC 추천 (Zero baseline, target={inf_cfg['apc_cuh_target']}) ===")
    print(f"grower: {grower}")
    print(f"pres_lot: {pres_lot}")
    print(f"prev (K={len(per_prev_results)} {args.blend_mode}):")
    print(df_history_show.to_string(index=False))
    print(f"\n결과: (ps_set_pres=타겟 실측, ps_delta_pres=추천-타겟, "
          f"cuh_rN=이전 run 위치별 CUH 참고)")
    print(df_out.to_string(index=False, float_format=lambda x: f"{x:.5f}"))

    # xlsx 저장 (열 너비 자동, 헤더 스타일, 숫자 포맷)
    out_xlsx = os.path.join(str(out_dir), f"{pres_lot}_apc_recommendation.xlsx")
    _save_apc_xlsx(out_xlsx, grower, pres_lot, args.blend_mode,
                   df_history, df_out)
    print(f"\n✅ 저장: {out_xlsx}")
    return True


if __name__ == "__main__":
    main()

