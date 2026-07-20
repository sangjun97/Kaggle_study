(venv_new) PS D:\python\CUH\growing_apc> & $PY scripts\model\02_refine_primary.py --set-name $name --gen GEN5
📊 Primary slopes (x_max=±0.002):
 GEN5_1: 11230.0745
 GEN5_2: 11213.9501
 GEN5_3: 10693.1535
 GEN5_4: 10076.0096
 GEN5_5: 10239.2442
 GEN5_6: 10073.9414
 GEN5_7: 9827.8364
 GEN5_8: 9466.8945
✅ slope JSON 저장: config/slopes\main\GEN5-pull_speed_t200_delta.json
 🖼️ 분석 플롯 저장: outputs\refine\plot_analysis_GEN5_pull_speed_t200_delta.png
✅ refine primary 완료 → outputs\refine
(venv_new) PS D:\python\CUH\growing_apc> & $PY scripts\model\03_refine_base.py --set-name $name --gen GEN5
📊 Base slopes [ftir_oi_center_delta] (x_max=±0.5):
  GEN5_1: 25.0514
  GEN5_2: 30.3361
  GEN5_3: 41.0027
  GEN5_4: 55.4653
  GEN5_5: 62.0755
  GEN5_6: 60.7913
  GEN5_7: 58.7584
  GEN5_8: 56.4421
✅ slope JSON 저장: config/slopes\main\GEN5-ftir_oi_center_delta.json
📊 Base slopes [dia_l50_delta] (x_max=±0.5):
  GEN5_1: -10.5936
  GEN5_2: -8.8655
  GEN5_3: -9.3270
  GEN5_4: -7.8646
  GEN5_5: -7.9580
  GEN5_6: -7.7777
  GEN5_7: -7.9051
  GEN5_8: -7.7350
✅ slope JSON 저장: config/slopes\main\GEN5-dia_l50_delta.json
✅ refine base 완료 → outputs\refine
(venv_new) PS D:\python\CUH\growing_apc> & $PY scripts\model\04_build_models_json.py --set-name $name
📋 release_params.csv 기반 mapping 자동 생성: 10 개 position
✅ 최종 slopes.json 생성: config/models_MID_pw25.json
✅ saved: config/models_MID_pw25.json
(venv_new) PS D:\python\CUH\growing_apc> 
(venv_new) PS D:\python\CUH\growing_apc> # slope 진폭 비교
(venv_new) PS D:\python\CUH\growing_apc> python -c "import json; d=json.load(open('config/models_$name.json',encoding='utf-8')); sl=[round(m['slope_main']['pull_speed_t200_delta']) for m in d['GEN5']['models']]; print('MID_pw25:',sl); print('진폭',max(sl)-min(sl),'peak@m',sl.index(max(sl))+1)"
MID_pw25: [11230, 11214, 10693, 10076, 10239, 10074, 9828, 9467]
진폭 1763 peak@m 1
