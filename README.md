"""스무딩 / 파생변수 함수 모음.

data2_grower_data.py 와 release/utils.py 에 중복으로 정의되어 있던
process_avg, process_avg_nan, process_set, process_oi, process_avg_const 통합.
"""

import numpy as np
import pandas as pd


def process_avg(df: pd.DataFrame, df_process: pd.DataFrame,
                ref_col: str, sub_col: str,
                windows: list[int], cols: list[str], suffix: str,
                min_periods: int = 1, verbose: bool = False) -> None:
    """ref_col 기준 정렬 후 이동평균 → df_process에 `{col}_{suffix}{window}` 추가.

    디버깅: verbose=True 또는 환경변수 GROWING_APC_VERBOSE=1 일 때 skip 사유 출력.
    """
    import os
    if os.environ.get("GROWING_APC_VERBOSE") == "1":
        verbose = True

    if ref_col not in df.columns:
        if verbose:
            print(f"    [process_avg] SKIP: ref_col '{ref_col}' not in df.columns")
        return
    if sub_col not in df.columns:
        if verbose:
            print(f"    [process_avg] SKIP: sub_col '{sub_col}' not in df.columns")
        return

    df_sorted = df.sort_values(by=[ref_col, sub_col]).reset_index(drop=False)
    df_clean  = df_sorted.drop_duplicates(subset=ref_col, keep="first")

    for window in windows:
        for col in cols:
            if col not in df_clean.columns:
                if verbose:
                    print(f"    [process_avg] SKIP col='{col}': not in df.columns")
                continue
            data = df_clean[["index", ref_col, col]].copy()
            data[col] = pd.to_numeric(data[col], errors="coerce")

            if len(data) < window + 1:
                if verbose:
                    print(f"    [process_avg] SKIP col='{col}' w={window}: "
                          f"len(data)={len(data)} < window+1={window+1}")
                continue

            new_col = f"{col}_{suffix}{window}"
            data[new_col] = data[col].rolling(window=window+1, center=True,
                                              min_periods=min_periods).mean()

            result_series = data.set_index("index")[new_col]
            df_process[new_col] = result_series

            # ref_col 그룹 내 첫 유효값으로 NaN 채움
            temp = df_process[[ref_col, new_col]].copy()
            temp = temp.sort_values(by=[ref_col, new_col])
            group_valid = temp.groupby(ref_col)[new_col].apply(
                lambda x: x.dropna().iloc[0] if not x.dropna().empty else np.nan
            )
            fill_values = df_process[ref_col].map(group_valid)
            df_process[new_col] = df_process[new_col].fillna(fill_values)


def process_avg_nan(df: pd.DataFrame, df_process: pd.DataFrame,
                    ref_col: str, sub_col: str,
                    windows: list[int], cols: list[str], suffix: str,
                    min_periods: int = 1, max_fill_steps: int = 1000) -> None:
    """NA를 ffill+bfill로 보간 후 이동평균 (DIA용)."""
    df_sorted = df.sort_values(by=[ref_col, sub_col]).reset_index(drop=False)
    df_clean  = df_sorted.drop_duplicates(subset=ref_col, keep="first")

    for window in windows:
        for col in cols:
            if col not in df_clean.columns:
                continue
            data = df_clean[["index", ref_col, col]].copy()
            data[col] = pd.to_numeric(data[col], errors="coerce")

            # ffill+bfill 반복 보간
            s = data[col].copy()
            if s.isna().any():
                for _ in range(max_fill_steps):
                    s_prev = s.copy()
                    s = s.ffill(limit=1)
                    s = s.bfill(limit=1)
                    if s.equals(s_prev):
                        break
            data[col] = s

            if len(data) < window + 1:
                continue

            new_col = f"{col}_{suffix}{window}"
            data[new_col] = data[col].rolling(window=window+1, center=True, min_periods=min_periods).mean()

            result_series = data.set_index("index")[new_col]
            df_process[new_col] = result_series

            temp = df_process[[ref_col, new_col]].copy()
            temp = temp.sort_values(by=[ref_col, new_col])
            group_valid = temp.groupby(ref_col)[new_col].apply(
                lambda x: x.dropna().iloc[0] if not x.dropna().empty else np.nan
            )
            fill_values = df_process[ref_col].map(group_valid)
            df_process[new_col] = df_process[new_col].fillna(fill_values)


def process_avg_const(df_process: pd.DataFrame, target_cols: list[str],
                      ref_col: str, ref_range: list[float]) -> None:
    """ref_col이 [min, max] 범위 내일 때 target_cols 각각의 평균을 상수 컬럼으로 추가."""
    ref_min, ref_max = ref_range
    cols_to_check = list(target_cols) + [ref_col]
    temp = df_process[cols_to_check].apply(pd.to_numeric, errors="coerce")
    mask = (temp[ref_col] >= ref_min) & (temp[ref_col] <= ref_max)

    for col in target_cols:
        subset = temp.loc[mask, col]
        const_value = subset.mean() if (not subset.empty and not subset.isna().all()) else np.nan
        df_process[f"{col}_avg_const"] = const_value


def process_set(df_process: pd.DataFrame, base_col: str, gap_col: str) -> None:
    """`{base_col}_set` = base_col - gap_col."""
    if base_col not in df_process.columns or gap_col not in df_process.columns:
        return
    temp = df_process[[base_col, gap_col]].apply(pd.to_numeric, errors="coerce")
    df_process[f"{base_col}_set"] = temp[base_col] - temp[gap_col]


def process_oi(df_process: pd.DataFrame) -> None:
    """FTIR 가장자리 4점 평균 + 중앙 포함 면적 평균 등 도메인 파생변수."""
    edge_cols = ["ftir_oi_edge1_1", "ftir_oi_edge1_2",
                 "ftir_oi_edge1_3", "ftir_oi_edge1_4"]
    if all(c in df_process.columns for c in edge_cols):
        temp = df_process[edge_cols].apply(pd.to_numeric, errors="coerce")
        df_process["ftir_oi_edge_avg"] = temp.mean(axis=1)

    area_cols = edge_cols + ["ftir_oi_center"]
    if all(c in df_process.columns for c in area_cols):
        temp = df_process[area_cols].apply(pd.to_numeric, errors="coerce")
        df_process["ftir_oi_area_avg"] = temp.mean(axis=1)

    if all(c in df_process.columns for c in ["ftir_oi_edge_avg", "ftir_oi_center"]):
        temp = df_process[["ftir_oi_edge_avg", "ftir_oi_center"]].apply(pd.to_numeric, errors="coerce")
        df_process["ftir_oi_avg"] = temp.mean(axis=1)
