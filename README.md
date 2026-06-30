(venv_new) PS D:\python\CUH> python .\diagnose_cuh_dist.py --data .\growing_apc\data\dataset\delta_train.csv --label Mid --out outputs/dist_mid --model-config .\growing_apc\config\model.yaml

============================================================
# Mid OI — CUH(cuh_bsw_all_prev) 분포 (n=15179)
============================================================
     n: 15179
  mean: 88.44
   std: 40.08
   min: 5.00
   max: 145.00
    p1: 10.00
   p25: 60.00
   p50: 90.00
   p75: 125.00
   p99: 145.00

 [양자화] 5단위 정수 비율: 100.0%

 [포화 구조]
 145 (진짜 포화) : 1872 건
 146~149 (빈 구간) : 0 건
 150 (BP/포화) : 0 건
 >=100 : 6470 건 (42.6%)

 ⚠️ 원본(center/edge) 없음 → 변환 전후 비교 생략. g1~g3 저장.

============================================================
# Mid OI — PS delta(pull_speed_t200_delta) 분포 (n=15179)
============================================================
     n: 15179
  mean: 0.00035
   std: 0.00224
   min: -0.01250
   max: 0.01446
    p1: -0.00495
   p25: -0.00124
   p50: 0.00056
   p75: 0.00191
   p99: 0.00564
 [|delta|<0.0005] 비율: 14.1% (제어 거의 없음)
 [|delta|<0.001] 비율: 29.6% (제어 거의 없음)
 [clip -0.007~0.007] 포함 비율: 99.6% (밖 0.4% 는 clip 으로 잘림)
 ✅ PS 그래프 저장: outputs/dist_mid/g5_ps_delta_Mid.png
 ✅ 수치 저장: outputs/dist_mid/summary_Mid.csv (+ summary_ps)
(venv_new) PS D:\python\CUH> python .\diagnose_cuh_dist.py --data .\growing_apc_LOW\data\dataset\delta_train.csv --label Low --out outputs/dist_low --model-config .\growing_apc_LOW\config\model.yaml

============================================================
# Low OI — CUH(cuh_bsw_all_prev) 분포 (n=1109)
============================================================
     n: 1109
  mean: 83.49
   std: 56.94
   min: 5.00
   max: 150.00
    p1: 5.00
   p25: 30.00
   p50: 75.00
   p75: 150.00
   p99: 150.00

 [양자화] 5단위 정수 비율: 100.0%

 [포화 구조]
 145 (진짜 포화) : 0 건
 146~149 (빈 구간) : 0 건
 150 (BP/포화) : 422 건
 >=100 : 422 건 (38.1%)

 ⚠️ 원본(center/edge) 없음 → 변환 전후 비교 생략. g1~g3 저장.

============================================================
# Low OI — PS delta(pull_speed_t200_delta) 분포 (n=1109)
============================================================
     n: 1109
  mean: -0.00003
   std: 0.00278
   min: -0.00786
   max: 0.00815
    p1: -0.00641
   p25: -0.00204
   p50: -0.00002
   p75: 0.00182
   p99: 0.00682
 [|delta|<0.0005] 비율: 12.5% (제어 거의 없음)
 [|delta|<0.001] 비율: 25.9% (제어 거의 없음)
 [clip -0.007~0.007] 포함 비율: 98.7% (밖 1.3% 는 clip 으로 잘림)
 ✅ PS 그래프 저장: outputs/dist_low/g5_ps_delta_Low.png
 ✅ 수치 저장: outputs/dist_low/summary_Low.csv (+ summary_ps)
