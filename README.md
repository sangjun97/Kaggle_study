(venv_new) PS D:\python\CUH\growing_apc_LOW> python .\scripts\release\sat_c.py     
포화 비율: 43.5% (110/253)
ROC-AUC: 0.670

              precision    recall  f1-score   support

         비포화      0.669     0.678     0.674       143
          포화      0.574     0.564     0.569       110

    accuracy                          0.628       253
   macro avg      0.622     0.621     0.621       253
weighted avg      0.628     0.628     0.628       253

혼동행렬 [행=실제, 열=예측]:
[[97 46]
 [48 62]]

피처 중요도 top5:
cuh_prev           0.339739
ftir_oi_center     0.175936
nop_shield         0.096464
all_cleaning       0.079896
pull_speed_t200    0.073520

실제 분류기 적용 MAE: 40.90 (oracle 18.74, 현재 37.17)
