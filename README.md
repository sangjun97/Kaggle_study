(venv_new) PS D:\python\CUH\growing_apc> Get-ChildItem -Path src\ -Filter *.py -Recurse | Select-String -Pattern "keep_boundary"

src\preprocess\pipeline.py:78:    df_ingot = filter_valid_data(df_p, keep_boundary=True, dedup_col="actual_pos")
src\preprocess\validation.py:6:def filter_valid_data(df: pd.DataFrame, keep_boundary: bool = False,
src\preprocess\validation.py:11:        keep_boundary: True 면 fpd_ar/ldp_ar 가 "R"(boundary reject)인 행도
src\preprocess\validation.py:26:        if keep_boundary:
