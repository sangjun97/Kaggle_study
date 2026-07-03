============================================================
# clip ?뚮퉬 吏???ㅼ틪
============================================================

[scripts/model/01_train.py] clip 誘몄궗??

[scripts/model/02_refine_primary.py] clip 誘몄궗??

[scripts/model/03_refine_base.py] clip 誘몄궗??

[scripts/model/04_build_models_json.py] clip 誘몄궗??

[scripts/release/validate_cuh.py] clip ?ъ슜
 L178: clip_config=clip_cfg,
 L267: clip_cfg = dict(model_cfg.clip_config)

============================================================
# src/ ?꾩껜 clip_config 李몄“
============================================================
 src\model\apc.py::L11: def preprocess_clip(row: pd.Series, clip_config: dict) -> tuple[pd.Series, list[str]]:
 src\model\apc.py::L15: for col, cfg in clip_config.items():
 src\model\apc.py::L43: clip_config: dict,
 src\model\apc.py::L57: min_v, max_v = clip_config[ctrl_var]["min"], clip_config[ctrl_var]["max"]
 src\model\apc.py::L76: clip_config: dict,
 src\model\apc.py::L88: min_v, max_v = clip_config[ctrl_var]["min"], clip_config[ctrl_var]["max"]
 src\release\pipeline.py::L282: clip_config: dict,
 src\release\pipeline.py::L290: #     if col not in df.columns or col not in clip_config:
 src\release\pipeline.py::L293: #     cfg = clip_config[col]
 src\release\pipeline.py::L298: # if any(((pd.to_numeric(df[c], errors="coerce") < clip_config[c]["min"])
 src\release\pipeline.py::L299: #         | (pd.to_numeric(df[c], errors="coerce") > clip_config[c]["max"])).any()
 src\release\pipeline.py::L300: #        for c in x_columns if c in clip_config and c in df.columns):
 src\release\pipeline.py::L355: row, clip_config, avg_main, x_columns,
