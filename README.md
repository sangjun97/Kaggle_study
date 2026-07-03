(venv_new) PS D:\python\CUH\growing_apc_LOW> # 1) model.yaml 의 실제 PS clip 표기 확인 (치환 실패 원인)
(venv_new) PS D:\python\CUH\growing_apc_LOW> Select-String -Path config\model.yaml -Pattern "pull_speed_t200_delta" -Context 0,3

> config\model.yaml:3:  primary_column: "pull_speed_t200_delta"
  config\model.yaml:4:
  config\model.yaml:5:  x_columns:
  config\model.yaml:6:    v1:
> config\model.yaml:7:      - pull_speed_t200_delta
  config\model.yaml:8:      - ftir_oi_center_delta
  config\model.yaml:9:      - dia_l50_delta
  config\model.yaml:10:    v2:
> config\model.yaml:11:      - pull_speed_t200_delta
  config\model.yaml:12:      - ftir_oi_center_delta
  config\model.yaml:13:      - dia_l50_delta
  config\model.yaml:14:
> config\model.yaml:79:  apc_control_column: "pull_speed_t200_delta"
  config\model.yaml:80:  test_date_split: 20250606
  config\model.yaml:81:
  config\model.yaml:82:clip_config:
> config\model.yaml:84:  pull_speed_t200_delta:   {min: -0.007, max: 0.007}
  config\model.yaml:85:  ftir_oi_center_delta:    {min: -0.8,   max: 0.8}
  config\model.yaml:86:  dia_l50_delta:           {min: -3.0,   max: 3.0}
  config\model.yaml:87:
> config\model.yaml:89:  pull_speed_t200_delta: 0.010
  config\model.yaml:90:  ftir_oi_center_delta:  1.0
  config\model.yaml:91:  dia_l50_delta:         2.0


(venv_new) PS D:\python\CUH\growing_apc_LOW>
(venv_new) PS D:\python\CUH\growing_apc_LOW> # 2) 01_train 이 --model-config 로 clip_config 를 읽어 쓰는지 확인
(venv_new) PS D:\python\CUH\growing_apc_LOW> python scripts\model\01_train.py --help | Select-String "model-config|clip"

                   [--model-config MODEL_CONFIG] --data-train DATA_TRAIN
  --model-config MODEL_CONFIG


(venv_new) PS D:\python\CUH\growing_apc_LOW> Select-String -Path scripts\model\01_train.py -Pattern "model.config|model_config|clip_config"

scripts\model\01_train.py:53:    parser.add_argument("--model-config", default="config/model.yaml")
scripts\model\01_train.py:105:    model_cfg = load_config(args.model_config)
