(venv_new) PS D:\python\CUH\growing_apc_LOW> python .\scripts\release\quant.py
delta 축소 계수 k = 0.254 (1이면 완벽, <1이면 중앙회귀)
 -> 모델이 실제 변화의 25%만 반영 (나머지는 중앙으로 당김)

비포화 n=140: MAE=32.94

저농도(<=30) 중 prev<60 인데 과대예측: n=26
 실제 16.7 / prev 36.4 / 예측 40.3
 -> prev 낮은데 과대면 delta 예측 자체 오류 (데이터/학습으로 개선 가능)
