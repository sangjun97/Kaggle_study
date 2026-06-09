
"""APC 후처리 보정 (boundary 룰 + score 선형 보간 누적 차감).

룰:
  Boundary (cuh_prev 변환 + PS 보정):
    ldp_ar == "R" → score=0, PS +0.002 (ldp 우선, b_band/fpd 와 동시 시)
    b_band_value == "Y" → score=0 (PS 보정 없음)
    fpd_ar == "R" → score=150, PS -0.005 (ldp 와 동시 발생 없음)

  학습 범위 clip:
    모델 inference 입력은 max(0, min(150, score)).
    보정 산출은 clip 전 (boundary 변환 후) 원본 score 기준.

  Score 선형 보간 누적 차감 (변환 후 score 기준, ldp 발동 시 미적용):
    150 < score ≤ 170 : -0.003 × (score-150)/20 (구간 내 선형, max -0.003)
    170 < score ≤ 230 : 150~170 통과 -0.003 + -0.002 × (score-170)/60
    230 < score ≤ 300 : 통과 -0.005 + -0.002 × (score-230)/70

  검증 예시:
    score=160 → -0.0015
    score=170 → -0.003
    score=200 → -0.003 - 0.001 = -0.004
    score=230 → -0.005
    score=265 → -0.005 - 0.001 = -0.006
    score=300 → -0.005 - 0.002 = -0.007
"""

from typing import Tuple
import numpy as np
import pandas as pd


BOUNDARY_COLS = ["b_band_value", "fpd_ar", "ldp_ar"]
MODEL_INPUT_CLIP = (0.0, 150.0) # 학습 범위


def transform_cuh_prev(row: pd.Series) -> Tuple[float, str]:
    """boundary 룰 적용 후 변환된 score + 발동 룰 라벨 반환.

    Returns:
        (new_score, rule)
          rule ∈ {"ldp_r", "b_band_y", "fpd_r", "none"}
    """
    score = row.get("cuh_bsw_all_prev", None)
    if score is None or pd.isna(score):
        return score, "none"

    # ldp_ar=R 가 b_band 와 동시 시 ldp 우선 (사용자 명세)
    if str(row.get("ldp_ar", "")).strip().upper() == "R":
        return 0.0, "ldp_r"
    if str(row.get("b_band_value", "")).strip().upper() == "Y":
        return 0.0, "b_band_y"
    if str(row.get("fpd_ar", "")).strip().upper() == "R":
        return 150.0, "fpd_r"

    return float(score), "none"


def _segment_offset(score: float) -> float:
    """score 의 선형 보간 누적 차감.

    score ≤ 150 → 0
    150 < score ≤ 170 → -0.003 × (score-150)/20
    170 < score ≤ 230 → -0.003 + -0.002 × (score-170)/60
    230 < score ≤ 300 → -0.005 + -0.002 × (score-230)/70
    score > 300 → -0.007 (capping, 명세상 발생 안 함)
    """
    if score is None or pd.isna(score) or score <= 150:
        return 0.0
    if score <= 170:
        return -0.003 * (score - 150) / 20.0
    if score <= 230:
        return -0.003 + (-0.002 * (score - 170) / 60.0)
    if score <= 300:
        return -0.005 + (-0.002 * (score - 230) / 70.0)
    return -0.007


def calc_correction(score_new: float, rule: str) -> float:
    """단일 위치의 PS 보정값 산출.

    Args:
        score_new: boundary 변환 후 score
        rule: transform_cuh_prev 가 반환한 룰 라벨

    Returns:
        ps_correction_value (모델 PS Δ 예측치에 더해질 값)
    """
    if rule == "ldp_r":
        # ldp 우선: 구간 룰 미적용, +0.002 만
        return 0.002
    if rule == "fpd_r":
        # fpd_r: score=150 변환 후 -0.005 보정
        # 150 시점에는 구간 보간 = 0 이므로 -0.005 만
        return -0.005
    if rule == "b_band_y":
        # b_band: score=0 변환 후 구간 보정 0 → 보정 없음
        return 0.0
    # 정상: 변환 후 score 기준 구간 보간
    return _segment_offset(score_new)


def calc_per_position(df_prev_at_pos: dict) -> Tuple[float, float, str]:
    """단일 (run, position) 의 변환 + 보정 산출.

    Args:
        df_prev_at_pos: 해당 위치의 PREV 데이터 한 행 (dict-like).
            필수 키: cuh_bsw_all_prev, b_band_value, fpd_ar, ldp_ar

    Returns:
        (model_input_score, ps_correction, rule)
          model_input_score : 학습 범위 clip 적용된 모델 입력값 (0~150)
          ps_correction : 이 run/position 에 더할 PS 보정값
          rule : 발동된 룰 라벨
    """
    score_orig = df_prev_at_pos.get("cuh_bsw_all_prev")
    if score_orig is None or pd.isna(score_orig):
        return None, 0.0, "none"

    new_score, rule = transform_cuh_prev(pd.Series(df_prev_at_pos))
    correction = calc_correction(new_score, rule)
    # 모델 입력은 학습 범위 clip
    lo, hi = MODEL_INPUT_CLIP
    model_input = max(lo, min(hi, new_score))
    return model_input, correction, rule

