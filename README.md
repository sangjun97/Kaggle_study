"""APC 추론 핵심 (cond 제거 버전).

기여도 계산 + APC 역산 + 가중 블렌딩.
"""

import numpy as np
import pandas as pd
from typing import Dict, List


def preprocess_clip(row: pd.Series, clip_config: dict) -> tuple[pd.Series, list[str]]:
    """row의 각 컬럼을 [min, max] 범위로 클리핑."""
    clipped = row.copy()
    clipped_cols = []
    for col, cfg in clip_config.items():
        if col not in clipped or pd.isna(clipped[col]):
            continue
        if clipped[col] < cfg["min"]:
            clipped[col] = cfg["min"]
            clipped_cols.append(col)
        elif clipped[col] > cfg["max"]:
            clipped[col] = cfg["max"]
            clipped_cols.append(col)
    return clipped, clipped_cols


def compute_slope_main_contrib(slope_main: Dict[str, float], input_row: pd.Series
                               ) -> tuple[dict, float]:
    """f(X) = Σ(slope × x). 기여도 dict + 합계 반환."""
    contrib = 0.0
    contrib_dict = {}
    for feat, slope in slope_main.items():
        if feat in input_row:
            delta = slope * input_row[feat]
            contrib += delta
            contrib_dict[feat] = round(delta, 2)
    return ({"Mains_Contrib": round(contrib, 2),
             **{f"Mains_{k}": v for k, v in contrib_dict.items()}},
            contrib)


def compute_apc_control(clipped_row: pd.Series,
                        clip_config: dict,
                        slope_main: Dict[str, float],
                        x_columns: List[str],
                        y_column: str = "cuh_bsw_all_delta",
                        c_column: str = "cuh_bsw_all_prev",
                        ctrl_var: str = "pull_speed_t200_delta",
                        y_target: float = 80.0,
                        resolution: float = 0.001) -> tuple[float, float, float, float]:
    """APC 역산: PS grid를 스캔하여 |contrib − y_delta_target| 최소가 되는 PS.

    벡터화 버전: 기존 이중 for-loop를 numpy 일괄 평가로 교체.
    """
    y_prev = float(clipped_row[c_column])
    y_delta_target = y_target - y_prev
    min_v, max_v = clip_config[ctrl_var]["min"], clip_config[ctrl_var]["max"]

    # 다른 X 변수들의 기여도는 고정값
    base_contrib = sum(slope_main.get(k, 0.0) * clipped_row[k]
                       for k in slope_main if k != ctrl_var and k in clipped_row)
    slope_ctrl = slope_main.get(ctrl_var, 0.0)

    def _search(res: float) -> tuple[float, float]:
        grid = np.arange(min_v, max_v + res, res)
        contribs = base_contrib + slope_ctrl * grid
        idx = np.argmin(np.abs(contribs - y_delta_target))
        return float(grid[idx]), float(contribs[idx])

    s1_x, s1_y = _search(resolution)
    s2_x, s2_y = _search(resolution / 10)
    return round(s1_x, 5), round(s2_x, 5), s1_y, s2_y


def compute_apc_control_zero_baseline(clipped_row: pd.Series,
                                      clip_config: dict,
                                      slope_main: Dict[str, float],
                                      x_columns: List[str],
                                      y_column: str = "cuh_bsw_all_delta",
                                      c_column: str = "cuh_bsw_all_prev",
                                      ctrl_var: str = "pull_speed_t200_delta",
                                      y_target: float = 80.0,
                                      resolution: float = 0.001
                                      ) -> tuple[float, float, float, float]:
    """release/dev.py 변형: 다른 X 변수를 0으로 가정한 baseline 도 함께 계산."""
    y_prev = float(clipped_row[c_column])
    y_delta_target = y_target - y_prev
    min_v, max_v = clip_config[ctrl_var]["min"], clip_config[ctrl_var]["max"]
    grid = np.arange(min_v, max_v + resolution, resolution)
    slope_ctrl = slope_main.get(ctrl_var, 0.0)

    # Prev 기반: 현재 X 값 그대로
    base_prev = sum(slope_main.get(k, 0.0) * clipped_row[k]
                    for k in slope_main if k != ctrl_var and k in clipped_row)
    contribs_prev = base_prev + slope_ctrl * grid
    idx_p = np.argmin(np.abs(contribs_prev - y_delta_target))

    # Zero 기반: 다른 X = 0 가정 (base_zero = 0)
    contribs_zero = slope_ctrl * grid
    idx_z = np.argmin(np.abs(contribs_zero - y_delta_target))

    return (round(float(grid[idx_p]), 5), round(float(grid[idx_z]), 5),
            float(contribs_prev[idx_p]), float(contribs_zero[idx_z]))


def get_blending_weights(n: int, base: float) -> list[float]:
    """모델 개수별 블렌딩 가중 패턴: [1, b, b², b, 1] 등."""
    patterns = {
        1: [1],
        2: [1, 1],
        3: [1, base, 1],
        4: [1, base, base, 1],
        5: [1, base, base*base, base, 1],
        6: [1, base, base*base, base*base, base, 1],
    }
    return patterns.get(n, [1.0] * n)


def blend_slopes(model_indices: list[int],
                 models: list[dict],
                 base: float = 1.5) -> Dict[str, float]:
    """run4_inference / dev.py 의 가중 평균 블렌딩 (cond 제거).

    models 는 model_idx 키를 가진 dict 리스트. 호출처가 전체 리스트를 넘기든
    필터링된 부분집합을 넘기든 model_idx 로 매칭.
    """
    weights = get_blending_weights(len(model_indices), base)
    sum_w = sum(weights)

    # model_idx → dict 로 lookup 테이블
    by_idx = {m["model_idx"]: m for m in models if "model_idx" in m}

    missing = [i for i in model_indices if i not in by_idx]
    if missing:
        raise KeyError(f"blend_slopes: model_idx {missing} not in models "
                       f"(available: {sorted(by_idx.keys())})")

    first = by_idx[model_indices[0]]
    total_main = {k: 0.0 for k in first["slope_main"].keys()}

    for w, idx in zip(weights, model_indices):
        m = by_idx[idx]
        for k, v in m["slope_main"].items():
            total_main[k] += v * w

    return {k: v / sum_w for k, v in total_main.items()}
