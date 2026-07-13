(venv_new) PS D:\python\CUH\growing_apc_LOW> Get-ChildItem -Path src\ -Filter *.py -Recurse |
>>     Select-String -Pattern "model_indices|mapping" |
>>     Select-Object Path, LineNumber, Line

Path                                                  LineNumber Line
----                                                  ---------- ----
D:\python\CUH\growing_apc_LOW\src\model\apc.py               119 def blend_slopes(model_indices: list[int],
D:\python\CUH\growing_apc_LOW\src\model\apc.py               127     weights = get_blending_weights(len(model_indices), base)
D:\python\CUH\growing_apc_LOW\src\model\apc.py               133     missing = [i for i in model_indices if i not in by_idx]
D:\python\CUH\growing_apc_LOW\src\model\apc.py               138     first = by_idx[model_indices[0]]
D:\python\CUH\growing_apc_LOW\src\model\apc.py               141     for w, idx in zip(weights, model_indices):
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           49     추가로 release_params.csv 의 position 정보를 읽어 GEN5 의 mapping 자동 생성.
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           74     # release_params 에서 position 읽고 GEN5 mapping 자동 생성
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           75     auto_mapping = None
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           79                                              build_position_to_models_mapping)
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           82                 auto_mapping = build_position_to_models_mapping(
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           84                 print(f"📋 release_params.csv 기반 mapping 자동 생성: "
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           85                       f"{len(auto_mapping)} 개 position")
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           91         # mapping 먼저, models 뒤 순서 (사용자 요청 형식)
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           93         if gen_key == "GEN5" and auto_mapping:
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py           94             gen_dict["mapping"] = auto_mapping
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py          117 def get_model_indices(ref_length: int, gen: str, config: dict) -> list | None:
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py          118     """REF_LENGTH → 적용 모델 인덱스 리스트 (mapping 필드)."""
D:\python\CUH\growing_apc_LOW\src\model\slope_io.py          119     return config.get(gen, {}).get("mapping", {}).get(str(ref_length))
D:\python\CUH\growing_apc_LOW\src\release\params.py            5   - models.json 의 GEN5 mapping 자동 재구성
D:\python\CUH\growing_apc_LOW\src\release\params.py           54 def build_position_to_models_mapping(positions: list[int],
D:\python\CUH\growing_apc_LOW\src\release\params.py           56     """N 개 position 에 대해 인수인계서의 GEN5 규칙으로 model_idx mapping 생성.
D:\python\CUH\growing_apc_LOW\src\release\params.py           82     mapping: dict[str, list[int]] = {}
D:\python\CUH\growing_apc_LOW\src\release\params.py           95         mapping[str(p)] = indices
D:\python\CUH\growing_apc_LOW\src\release\params.py           97     return mapping
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py         15 from src.model.slope_io import get_model_indices
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        343         model_indices = get_model_indices(ref_length, pos_type, slope_config)
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        344         print(model_indices)
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        345         if model_indices is None:
D:\python\CUH\growing_apc_LOW\src\release\pipeline.py        348         avg_main = blend_slopes(model_indices, slope_config[pos_type]["models"], blend_base)

