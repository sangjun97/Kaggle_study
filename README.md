(venv_new) PS D:\python\CUH\growing_apc> python scripts/check_zero_to_target.py --set-name model_v1_P7_pw7 --target 80

=== Sanity Check: CUH_prev → target 80.0 (Zero baseline) ===
  PS clip: [-0.04, 0.04]
  models in GEN5: 8

model       slope_PS       CUH 0.0→80.0     CUH 150.0→80.0
--------------------------------------------------------
M1           13677.9   PS Δ=+0.00585         PS Δ=-0.00512
M2           14253.9   PS Δ=+0.00561         PS Δ=-0.00491
M3           14283.1   PS Δ=+0.00560         PS Δ=-0.00490
M4           13819.6   PS Δ=+0.00579         PS Δ=-0.00507
M5           14136.7   PS Δ=+0.00566         PS Δ=-0.00495
M6           13838.1   PS Δ=+0.00578         PS Δ=-0.00506
M7           13082.3   PS Δ=+0.00612         PS Δ=-0.00535
M8           12713.3   PS Δ=+0.00629         PS Δ=-0.00551

=== Position 별 앙상블 slope + Zero 추천 (mapping 기반) ===
position     avg_slope       CUH 0.0→80.0     CUH 150.0→80.0
----------------------------------------------------------
270            13677.9   PS Δ=+0.00585         PS Δ=-0.00512
390            13677.9   PS Δ=+0.00585         PS Δ=-0.00512
590            13965.9   PS Δ=+0.00573         PS Δ=-0.00501
790            13965.9   PS Δ=+0.00573         PS Δ=-0.00501
990            14097.7   PS Δ=+0.00567         PS Δ=-0.00497
1190           14097.7   PS Δ=+0.00567         PS Δ=-0.00497
1390           14268.5   PS Δ=+0.00561         PS Δ=-0.00491
1590           14268.5   PS Δ=+0.00561         PS Δ=-0.00491
1790           14283.1   PS Δ=+0.00560         PS Δ=-0.00490
1990           14283.1   PS Δ=+0.00560         PS Δ=-0.00490

====================================================================================================
Baseline: model_v1 | Test_MAE_Pred = 26.042, Test__PredR2 = 0.246
====================================================================================================

SET                                Test_MAE     ΔMAE    Test_R²      ΔR²       판정
--------------------------------------------------------------------------------
model_v1_T9_lr007                    25.536   -0.507      0.287   +0.041        ✅
model_v1_T2_lronly                   25.550   -0.492      0.282   +0.036        ✅
model_v1_T15_lr015_nest200           25.551   -0.491      0.282   +0.036        ✅
model_v1_T11_nest200                 25.560   -0.483      0.286   +0.040        ✅
model_v1_D2_pw5                      25.629   -0.414      0.274   +0.028        ✅
model_v1_T13_nest400                 25.721   -0.321      0.269   +0.023        ✅
model_v1_P6_pw6                      25.768   -0.275      0.264   +0.018        ✅
model_v1_T10_lr015                   25.801   -0.242      0.264   +0.018        ✅
model_v1_T12_nest150                 25.827   -0.215      0.275   +0.029        ✅
model_v1_T5_slow_bag4                25.864   -0.178      0.259   +0.013        ✅
model_v1_T6_slow_bins128             25.865   -0.178      0.260   +0.014        ✅
model_v1_E2_slow                     25.866   -0.177      0.259   +0.013        ✅
model_v1_T4_slow_head05              25.867   -0.176      0.259   +0.013        ✅
model_v1_P7_pw7                      25.920   -0.123      0.254   +0.008        ✅
model_v1_T14_lr007_nest200           25.921   -0.122      0.270   +0.024        ✅
model_v1_T1_slower                   25.935   -0.107      0.253   +0.007        ✅
model_v1_T7_veryslow                 26.010   -0.032      0.248   +0.002        ≈
model_v1_D4_no_stage1w               26.017   -0.025      0.265   +0.019        ≈
model_v1_T8_mid                      26.036   -0.006      0.247   +0.001        ≈
model_v1_E1_head2                    26.050   +0.008      0.245   -0.001        ≈
model_v1_E3_robust                   26.053   +0.010      0.245   -0.001        ≈
model_v1_E3_std                      26.053   +0.011      0.245   -0.001        ≈
model_v1_E1_head05                   26.053   +0.011      0.246   -0.000        ≈
model_v1_P8_pw8                      26.073   +0.030      0.243   -0.003        ≈
model_v1_E2_bins128                  26.073   +0.031      0.245   -0.001        ≈
model_v1_T3_nestonly                 26.076   +0.033      0.243   -0.003        ≈
model_v1_E2_bins32                   26.125   +0.083      0.243   -0.003        ≈
model_v1_T3b_nestmore                26.127   +0.084      0.240   -0.006        ≈
model_v1_P9_pw9                      26.221   +0.179      0.232   -0.014        🚫
model_v1_GEN6_head                   26.246   +0.203      0.250   +0.004        🚫
model_v1_D3_pw10                     26.364   +0.322      0.222   -0.024        🚫
model_v1_w07                         26.524   +0.481      0.211   -0.035        🚫
model_v1_D1_no_stage2                26.845   +0.803      0.211   -0.035        🚫
model_v1_w05                         26.852   +0.810      0.187   -0.059        🚫
model_v1_w03                         27.117   +1.074      0.168   -0.078        🚫
model_v1_clean                       27.368   +1.325      0.150   -0.095        🚫

====================================================================================================
K-fold CV (K=5, seed=42)
 hyperparameter: lr=0.007, n_est=300, pw_high=7.0, outer_bags=2
 실험: ['GEN4_5', 'GEN5_4', 'GEN6_3']
====================================================================================================


--- GEN4_5 (positions=[990, 1020, 1190, 1220], n=2496) ---
 Fold 1/5: train=1996, val=500  [Primary Weight] Stage1 : 189/1996
→ Val MAE=25.284, R²=nan
 Fold 2/5: train=1997, val=499  [Primary Weight] Stage1 : 186/1997
→ Val MAE=24.706, R²=nan
 Fold 3/5: train=1997, val=499  [Primary Weight] Stage1 : 184/1997
→ Val MAE=26.334, R²=nan
 Fold 4/5: train=1997, val=499  [Primary Weight] Stage1 : 193/1997
→ Val MAE=23.606, R²=nan
 Fold 5/5: train=1997, val=499  [Primary Weight] Stage1 : 185/1997
→ Val MAE=25.557, R²=nan

 [GEN4_5 요약 — K=5 fold 평균 ± std]
 Val_MAE_Pred             :  25.098 ± 1.019
 Train_MAE_Pred           :  25.054 ± 0.196

--- GEN5_4 (positions=[790, 830, 990, 1020, 1190, 1220], n=3909) ---
 Fold 1/5: train=3127, val=782  [Primary Weight] Stage1 : 303/3127
→ Val MAE=25.680, R²=nan
 Fold 2/5: train=3127, val=782  [Primary Weight] Stage1 : 304/3127
→ Val MAE=25.675, R²=nan
 Fold 3/5: train=3127, val=782  [Primary Weight] Stage1 : 255/3127
→ Val MAE=26.038, R²=nan
 Fold 4/5: train=3127, val=782  [Primary Weight] Stage1 : 308/3127
→ Val MAE=26.251, R²=nan
 Fold 5/5: train=3128, val=781  [Primary Weight] Stage1 : 262/3128
→ Val MAE=26.243, R²=nan

 [GEN5_4 요약 — K=5 fold 평균 ± std]
 Val_MAE_Pred             :  25.977 ± 0.287
 Train_MAE_Pred           :  25.921 ± 0.067

--- GEN6_3 (positions=[390, 440, 590, 630], n=3169) ---
 Fold 1/5: train=2535, val=634  [Primary Weight] Stage1 : 194/2535
→ Val MAE=28.326, R²=nan
 Fold 2/5: train=2535, val=634  [Primary Weight] Stage1 : 197/2535
→ Val MAE=26.483, R²=nan
 Fold 3/5: train=2535, val=634  [Primary Weight] Stage1 : 208/2535
→ Val MAE=27.619, R²=nan
 Fold 4/5: train=2535, val=634  [Primary Weight] Stage1 : 220/2535
→ Val MAE=27.357, R²=nan
 Fold 5/5: train=2536, val=633  [Primary Weight] Stage1 : 205/2536
→ Val MAE=28.565, R²=nan

 [GEN6_3 요약 — K=5 fold 평균 ± std]
 Val_MAE_Pred             :  27.670 ± 0.828
 Train_MAE_Pred           :  27.613 ± 0.155

✅ 저장: outputs\cv\cv_results_20260519_085824.csv

================================================================================
📊 전체 CV 요약
================================================================================
EXP            Val_MAE_mean    Val_MAE_std    Val_R²_mean     Val_R²_std
--------------------------------------------------------------------------------
Index(['Train_MAE_Pred', 'Train_PredR2', 'Train_TA10', 'Train_TA20',
       'Val_MAE_Pred', 'Val_PredR2', 'Val_TA10', 'Val_TA20', 'n_train',
       'n_val', 'EXP', 'FOLD'],
      dtype='object')
GEN4_5               25.098          1.019          0.285          0.036
GEN5_4               25.977          0.287          0.265          0.025
GEN6_3               27.670          0.828          0.202          0.054
