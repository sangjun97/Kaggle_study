"""filter_valid_data: 물리적 범위 컷 + 중복 제거."""

import pandas as pd


def filter_valid_data(df: pd.DataFrame) -> pd.DataFrame:
    if df.empty:
        return df

    out = df.copy().sort_values(by=["length", "runtime_event"])
    if "fpd_ar" in out.columns and "ldp_ar" in out.columns:
        out = out[(out["fpd_ar"] == "A") & (out["ldp_ar"] == "A")]

    # for col in ["cuh_bp_center", "cuh_bsw_center"]:
    #     if col in out.columns:
    #         vals = pd.to_numeric(out[col], errors="coerce")
    #         out  = out[(vals >= 0) & (vals <= 100)]

    # for col in ["cuh_bp_edge", "cuh_bsw_edge"]:
    #     if col in out.columns:
    #         vals = pd.to_numeric(out[col], errors="coerce")
    #         out  = out[(vals >= 0) & (vals <= 50)]

    for col in ["ftir_oi_center"]:
        if col in out.columns:
            vals = pd.to_numeric(out[col], errors="coerce")
            out  = out[(vals >= 0) & (vals <= 20)]

    out = out.drop_duplicates(subset="length", keep="first")
    return out
