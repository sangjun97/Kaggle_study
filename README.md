delta X: ['pull_speed_t200_delta', 'ftir_oi_center_delta', 'dia_l50_delta']
절대값 X 추정: ['pull_speed_t200', 'ftir_oi_center', 'dia_l50']
[경고] dataset 에 없는 절대값 컬럼: {'dia_l50', 'ftir_oi_center', 'pull_speed_t200'}
 실제 컬럼: ['pull_speed_t200_delta', 'ftir_oi_center_delta', 'dia_l50_delta', 'ftir_oi_center_pres', 'ftir_oi_center_prev', 'actual_pull_speed_set_delta']

[절대값 EBM] feat=['cuh_bsw_all_prev']
 MAE=46.89 Bias=+0.45 k=0.072 (delta: MAE=37.17 k=0.254)

 D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
  warnings.warn(
========================================================================================= test session starts ==========================================================================================
platform win32 -- Python 3.11.9, pytest-9.1.1, pluggy-1.6.0 -- D:\python\virtualenv\venv_new\Scripts\python.exe
cachedir: .pytest_cache
rootdir: D:\python\CUH\growing_apc_LOW
plugins: anyio-4.11.0, dash-4.1.0, langsmith-0.4.42
collected 2 items                                                                                                                                                                                       

scripts/release/test_abs.py::test_abs_columns_exist FAILED
scripts/release/test_abs.py::test_abs_model_reduces_central_pull abs: k=0.072 MAE=46.89 | delta: k=0.254 MAE=37.17
FAILED

=============================================================================================== FAILURES =============================================================================================== 
________________________________________________________________________________________ test_abs_columns_exist ________________________________________________________________________________________ 

    def test_abs_columns_exist():
        df = pd.read_csv(TRAIN, nrows=1)
        feat = _abs_features(df)
>       assert len(feat) >= 2, f"절대값 피처 부족: {feat}"
E       AssertionError: 절대값 피처 부족: ['cuh_bsw_all_prev']
E       assert 1 >= 2
E        +  where 1 = len(['cuh_bsw_all_prev'])

scripts\release\test_abs.py:26: AssertionError
_________________________________________________________________________________ test_abs_model_reduces_central_pull __________________________________________________________________________________ 

    def test_abs_model_reduces_central_pull():
        df = pd.read_csv(TRAIN)
        feat = _abs_features(df)
        d = df[feat + [Y_ABS]].apply(pd.to_numeric, errors="coerce").dropna()

        ebm = ExplainableBoostingRegressor(interactions=0)
        pred = cross_val_predict(ebm, d[feat], d[Y_ABS], cv=5)
        k_abs = np.polyfit(d[Y_ABS], pred, 1)[0]
        mae = np.abs(pred - d[Y_ABS]).mean()

        print(f"abs: k={k_abs:.3f} MAE={mae:.2f} | delta: k=0.254 MAE=37.17")
>       assert k_abs > 0.254, "절대값도 중앙회귀 심하면 EBM/데이터 공통 한계"
E       AssertionError: 절대값도 중앙회귀 심하면 EBM/데이터 공통 한계
E       assert np.float64(0.07242946154877776) > 0.254

scripts\release\test_abs.py:41: AssertionError
======================================================================================= short test summary info ======================================================================================== 
FAILED scripts/release/test_abs.py::test_abs_columns_exist - AssertionError: 절대값 피처 부족: ['cuh_bsw_all_prev']
FAILED scripts/release/test_abs.py::test_abs_model_reduces_central_pull - AssertionError: 절대값도 중앙회귀 심하면 EBM/데이터 공통 한계
===================================================================================== 2 failed in 86.43s (0:01:26) ===================================================================================== 
