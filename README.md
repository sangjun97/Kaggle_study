======================================================================
# Low OI dataset 분포 / clip 제안
# data/dataset/delta_train.csv (행 1063)
======================================================================

[1] cuh_bsw_all BP 변환 검증 (145 천장 + 150 덩어리 기대)
 cuh_bsw_all_pres: n=1063, =145(BSW천장) 19건, =150(BP개입) 10건, 145~150 사이 0건
 cuh_bsw_all_prev: n=1063, =145(BSW천장) 30건, =150(BP개입) 23건, 145~150 사이 0건

[2] 분포표
column                            n      min       p1      p50      p99      max
--------------------------------------------------------------------------------
pull_speed_t200_delta          1063  -0.0079  -0.0064  -0.0001   0.0068   0.0081
ftir_oi_center_delta           1063  -0.9140  -0.4384   0.0120   0.4978   0.6340
dia_l50_delta                  1063  -2.7315  -1.2825   0.1060   1.3172   3.1202
cuh_bsw_all_prev               1063   5.0000   5.0000  70.0000 150.0000 150.0000
cuh_bsw_all_pres               1063   5.0000   5.0000  70.0000 145.0000 150.0000
cuh_bsw_all_prev               1063   5.0000   5.0000  70.0000 150.0000 150.0000
cuh_bsw_all_delta              1063-140.0000-126.9000  -5.0000 115.0000 140.0000

[3] clip_config 제안 (percentile 1~99)
 복사용 ↓

clip_config:
 cuh_bsw_all_prev: {min: 5.0, max: 150}
 pull_speed_t200_delta: {min: -0.0068, max: 0.0068}
 ftir_oi_center_delta:  {min: -0.4978, max: 0.4978}
 dia_l50_delta:         {min: -1.3172, max: 1.3172}

 ※ delta 는 대칭(|min|=|max|), cuh_bsw_all_prev 상한은 BP(150) 보존 위해 150 고정.
 ※ PS↔CUH 계수가 Mid 와 같다면 pull_speed delta 는 Mid 값(±0.007) 유지도 가능.
