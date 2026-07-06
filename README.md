(venv_new) PS D:\python\CUH\growing_apc_LOW> python .\scripts\release\learnable_breakdown.py
=== 학습가능군 CUH 구간별 MAE ===
   0~30 : n= 36 MAE= 36.45 Bias= +33.89
  30~60 : n= 39 MAE= 28.36 Bias= +13.49
  60~90 : n= 53 MAE= 31.39 Bias=  +8.20
  90~130: n= 10 MAE= 33.63 Bias= -19.14
 130~151: n= 99 MAE= 38.56 Bias= -38.56

학습가능군 중 포화(150): n=99, MAE=38.56
 이 그룹 prev 평균: 110.1 (150 근처면 delta 0 으로 맞춰야)

비포화 학습가능군: n=138, MAE=32.02, Bias=+14.42
 -> 이 값이 Mid(25.7) 대비 높으면 데이터 부족이 근본 원인
