(venv_new) PS D:\python\CUH\growing_apc_LOW> Get-ChildItem -Path src\ -Filter *.py -Recurse |
>>     Select-String -Pattern "clip_config" |
>>     Select-Object Path, LineNumber, Line

Path                                                  LineNumber Line
----                                                  ---------- ----
D:\python\CUH\growing_apc_LOW\src\model\apc.py                11 def preprocess_clip(row: pd.Series, clip_config: dict) -> tuple[pd.Series, list[str]]:
D:\python\CUH\growing_apc_LOW\src\model\apc.py                15     for col, cfg in clip_config.items():
D:\python\CUH\growing_apc_LOW\src\model\apc.py                43                         clip_config: dict,
D:\python\CUH\growing_apc_LOW\src\model\apc.py                57     min_v, max_v = clip_config[ctrl_var]["min"], clip_config[ctrl_var]["max"]
D:\python\CUH\growing_apc_LOW\src\model\apc.py                76                                       clip_config: dict,
D:\python\CUH\growing_apc_LOW\src\model\apc.py                88     min_v, max_v = clip_config[ctrl_var]["min"], clip_config[ctrl_var]["max"]
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        282                             clip_config: dict,
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        290     #     if col not in df.columns or col not in clip_config:
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        293     #     cfg = clip_config[col]
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        298     # if any(((pd.to_numeric(df[c], errors="coerce") < clip_config[c]["min"])
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        299     #         | (pd.to_numeric(df[c], errors="coerce") > clip_config[c]["max"])).any()
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        300     #        for c in x_columns if c in clip_config and c in df.columns):
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        355             row, clip_config, avg_main, x_columns,


(venv_new) PS D:\python\CUH\growing_apc_LOW>
(venv_new) PS D:\python\CUH\growing_apc_LOW> # validate_cuh 예측이 clip 을 쓰는지 slope 만 쓰는지
(venv_new) PS D:\python\CUH\growing_apc_LOW> Select-String -Path scripts\release\validate_cuh.py `
>>     -Pattern "clip|global_max_abs|slope|cuh_pred" -Context 0,1

> scripts\release\validate_cuh.py:5:  모델의 cuh 예측(cuh_pred)을 ref_length 위치별로 나란히 저장.
  scripts\release\validate_cuh.py:6:  → 모델이 실측 cuh 를 얼마나 맞추는지 1년치로 검증.
> scripts\release\validate_cuh.py:10:  cuh_pred = cuh_prev(PREV 실측) + Δcuh 예측(5run blend, PRES 시점).
  scripts\release\validate_cuh.py:11:  → 예측은 PRES 본인 시점. PRES lot 행에 그대로 저장.
> scripts\release\validate_cuh.py:20:  nop_shield, nop_tube, side_heater, st_heater, cuh_pred
  scripts\release\validate_cuh.py:21:
> scripts\release\validate_cuh.py:40:from src.model.slope_io import load_slope_config
  scripts\release\validate_cuh.py:41:from src.preprocess.matching import extract_at_references
> scripts\release\validate_cuh.py:54:    "cuh_prev", "cuh_pred",
  scripts\release\validate_cuh.py:55:]
> scripts\release\validate_cuh.py:60:def _blend_cuh_pred(per_prev_results, mode):
  scripts\release\validate_cuh.py:61:    """위치별 cuh 예측 blend.
> scripts\release\validate_cuh.py:64:    cuh_pred[위치] = Σ w·(Y_PREV + YD_PRED) / Σ w (위치별 참여 run 만)
  scripts\release\validate_cuh.py:65:    weighted: w = 1/run_delta.
> scripts\release\validate_cuh.py:67:    반환: (cuh_pred_dict, cuh_prev_dict)
  scripts\release\validate_cuh.py:68:      cuh_prev_dict = Σ w·Y_PREV / Σ w (같은 가중치로 blend 한 기준 cuh)
> scripts\release\validate_cuh.py:71:    by_pos = {} # ref -> list of (cuh_pred_i, weight)
  scripts\release\validate_cuh.py:72:    by_pos_prev = {} # ref -> list of (y_prev_i, weight)
> scripts\release\validate_cuh.py:85:            cuh_pred_i = float(yp) + float(yd)
> scripts\release\validate_cuh.py:86:            by_pos.setdefault(ref, []).append((cuh_pred_i, w))
  scripts\release\validate_cuh.py:87:            by_pos_prev.setdefault(ref, []).append((float(yp), w))
> scripts\release\validate_cuh.py:89:    def _reduce(bp, do_clip):
  scripts\release\validate_cuh.py:90:        out = {}
> scripts\release\validate_cuh.py:98:            out[ref] = float(min(150.0, max(0.0, v))) if do_clip else float(v)
  scripts\release\validate_cuh.py:99:        return out
> scripts\release\validate_cuh.py:101:    # cuh_pred 는 물리범위 clip, cuh_prev 는 실측 기준이라 clip 안 함
  scripts\release\validate_cuh.py:102:    return _reduce(by_pos, True), _reduce(by_pos_prev, False)
> scripts\release\validate_cuh.py:106:                     clip_cfg, customer_filter, x_cols, valid_status):
> scripts\release\validate_cuh.py:107:    """단일 lot 의 위치별 cuh_pred 산출. {ref_length: cuh_pred} 반환."""
  scripts\release\validate_cuh.py:108:    ctx = collect_lot_context(
> scripts\release\validate_cuh.py:158:    slope_config = load_slope_config(_mj)
  scripts\release\validate_cuh.py:159:
> scripts\release\validate_cuh.py:173:            df_delta, slope_config,
  scripts\release\validate_cuh.py:174:            x_columns=x_cols,
> scripts\release\validate_cuh.py:178:            clip_config=clip_cfg,
  scripts\release\validate_cuh.py:179:            inference_cfg=inf_cfg,
> scripts\release\validate_cuh.py:191:    cuh_pred_at, cuh_prev_at = _blend_cuh_pred(per_prev, args.blend_mode)
  scripts\release\validate_cuh.py:192:    # 저장용으로 전처리된 df_pres 반환 (pull_speed_t200 등 파생 컬럼 포함).
> scripts\release\validate_cuh.py:194:    return cuh_pred_at, cuh_prev_at, df_pres
  scripts\release\validate_cuh.py:195:
> scripts\release\validate_cuh.py:197:def _target_rows_at_refs(target_df, rel_cfg, cuh_pred_at, cuh_prev_at):
> scripts\release\validate_cuh.py:198:    """타겟 잉곳(전처리됨)에서 ref_length 위치별 1행 추출 + 실측 + cuh_pred.
  scripts\release\validate_cuh.py:199:
> scripts\release\validate_cuh.py:226:               if c not in ("cuh_pred", "cuh_prev", "length")}
  scripts\release\validate_cuh.py:227:        row["length"] = r0.get("length", ref)
> scripts\release\validate_cuh.py:228:        row["cuh_pred"] = cuh_pred_at.get(ref, None)
  scripts\release\validate_cuh.py:229:        row["cuh_prev"] = cuh_prev_at.get(ref, None)
> scripts\release\validate_cuh.py:267:    clip_cfg = dict(model_cfg.clip_config)
  scripts\release\validate_cuh.py:268:    if os.path.exists(args.release_params):
> scripts\release\validate_cuh.py:292:            cuh_pred_at, cuh_prev_at, target_df = _predict_one_lot(
  scripts\release\validate_cuh.py:293:                lot, args, paths, pcfg, model_cfg, rel_cfg, inf_cfg,
> scripts\release\validate_cuh.py:294:                clip_cfg, customer_filter, x_cols, valid_status)
> scripts\release\validate_cuh.py:295:            if target_df is None or not cuh_pred_at:
  scripts\release\validate_cuh.py:296:                print(f" [{i}/{len(lots)}] {lot}: 예측 불가 (prev 부족/전처리 실패)", flush=True)
> scripts\release\validate_cuh.py:298:            rows = _target_rows_at_refs(target_df, rel_cfg, cuh_pred_at, cuh_prev_at)
  scripts\release\validate_cuh.py:299:            all_rows.extend(rows)
