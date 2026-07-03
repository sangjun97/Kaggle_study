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
(venv_new) PS D:\python\CUH\growing_apc_LOW> # 01_train 단독
(venv_new) PS D:\python\CUH\growing_apc_LOW> Select-String -Path scripts\model\01_train.py -Pattern "clip_config|clip" -Context 0,2
(venv_new) PS D:\python\CUH\growing_apc_LOW> 
(venv_new) PS D:\python\CUH\growing_apc_LOW> # validate_cuh 가 clip 을 쓰는지 slope 만 쓰는지
(venv_new) PS D:\python\CUH\growing_apc_LOW> Select-String -Path scripts\release\validate_cuh.py -Pattern "clip|global_max_abs|slope" -Context 0,1

> scripts\release\validate_cuh.py:40:from src.model.slope_io import load_slope_config
  scripts\release\validate_cuh.py:41:from src.preprocess.matching import extract_at_references
> scripts\release\validate_cuh.py:89:    def _reduce(bp, do_clip):
  scripts\release\validate_cuh.py:90:        out = {}
> scripts\release\validate_cuh.py:98:            out[ref] = float(min(150.0, max(0.0, v))) if do_clip else float(v)
  scripts\release\validate_cuh.py:99:        return out
> scripts\release\validate_cuh.py:101:    # cuh_pred 는 물리범위 clip, cuh_prev 는 실측 기준이라 clip 안 함
  scripts\release\validate_cuh.py:102:    return _reduce(by_pos, True), _reduce(by_pos_prev, False)
> scripts\release\validate_cuh.py:106:                     clip_cfg, customer_filter, x_cols, valid_status):
  scripts\release\validate_cuh.py:107:    """단일 lot 의 위치별 cuh_pred 산출. {ref_length: cuh_pred} 반환."""
> scripts\release\validate_cuh.py:158:    slope_config = load_slope_config(_mj)
  scripts\release\validate_cuh.py:159:
> scripts\release\validate_cuh.py:173:            df_delta, slope_config,
  scripts\release\validate_cuh.py:174:            x_columns=x_cols,
> scripts\release\validate_cuh.py:178:            clip_config=clip_cfg,
  scripts\release\validate_cuh.py:179:            inference_cfg=inf_cfg,
> scripts\release\validate_cuh.py:267:    clip_cfg = dict(model_cfg.clip_config)
  scripts\release\validate_cuh.py:268:    if os.path.exists(args.release_params):
> scripts\release\validate_cuh.py:294:                clip_cfg, customer_filter, x_cols, valid_status)
  scripts\release\validate_cuh.py:295:            if target_df is None or not cuh_pred_at:
