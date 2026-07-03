n=1109
|PS| > 0.007 (기존 clip 밖): 14 (1.3%)
0.007 < |PS| <= 0.01 (완화 수혜): 14 (1.3%) <- 이 값들만 예측 입력이 바뀜
|PS| > 0.01 (여전히 잘림): 0

[결론] 14건이 영향받음 -> B=C1 동일하면 validate_cuh 가 wideclip yaml 을 안 읽은 것. --model-config 전달 경로 점검.

(venv_new) PS D:\python\CUH\growing_apc_LOW> Select-String -Path scripts\release\validate_cuh.py -Pattern "model_config|model_cfg\s*=|load_config" -Context 0,1

> scripts\release\validate_cuh.py:35:from src.core.config import load_config
  scripts\release\validate_cuh.py:36:from src.release.params import load_release_params
> scripts\release\validate_cuh.py:252:    paths = load_config(args.paths_config)
> scripts\release\validate_cuh.py:253:    pcfg = load_config(args.preprocess_config)
> scripts\release\validate_cuh.py:254:    model_cfg = load_config(args.model_config)
  scripts\release\validate_cuh.py:255:
