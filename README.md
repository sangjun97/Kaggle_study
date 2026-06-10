======================================================================
# 진단: APC 위치별 cuh / 비율 — lot 5U462
======================================================================
🎯 target lot 조회: 5U462
 target 6412행, USAGE 컬럼 9개, gr_out=20260108, customer=Smart_SK hynix/GF/CXMT 440
🔎 lot 시퀀스 조회: prefixes=['5U4', '4U4']
 이전 lot 후보 118개 (최신순)
 ✅ 5U461: 수집 (1/5, 5050행)
 ✅ 5U460: 수집 (2/5, 5066행)
 ✅ 5U459: 수집 (3/5, 5073행)
 ✅ 5U458: 수집 (4/5, 5064행)
 ✅ 5U457: 수집 (5/5, 5070행)
 skip 통계: FDC없음=0, 필터=0, cuh없음=0, gr_out=0, customer=0, USAGE탈락=0
 grower DF 구성: 60행, 잉곳 5개 (요청 run 5개)
  PREV 후보 수집: 5 개 (검사 5 개, skip 0 개)
grower=TG14, prev lots=['5U461', '5U460', '5U459', '5U458', '5U457'], blend K=5, target=80.0

=== 위치별 진단 (lot 5U462) ===
 model_cuh_in : boundary 룰 적용 후 모델에 들어간 cuh (run 평균)
 cuh_need(Δ) : target - model_cuh_in
 ps_delta : blended ps 변화량
 cuhΔ:psΔ : cuh_need / ps_delta (ps 1 단위당 cuh 변화의 역수)

 position  n_run  model_cuh_in  target  cuh_need(Δ)  ps_prev  ps_delta  cuhΔ:psΔ
      270      5          88.0    80.0         -8.0  0.44480   0.00121   -6602.4
      390      1           0.0    80.0         80.0  0.45177   0.00500   16000.0
      590      1           0.0    80.0         80.0  0.46510   0.00500   16000.0
      790      1           0.0    80.0         80.0  0.46946   0.00500   16000.0
      990      1           0.0    80.0         80.0  0.47242   0.00500   16000.0
     1190      1           0.0    80.0         80.0  0.47415   0.00500   16000.0
     1390      1           0.0    80.0         80.0  0.47665   0.00500   16000.0
     1790      5         114.0    80.0        -34.0  0.47823  -0.00029  116450.0
     1990      5         122.0    80.0        -42.0  0.47741  -0.00097   43263.2

=== run별 위치별 상세 (center/edge → all → rule) ===
 center_prev/edge_prev : 원본 cuh (둘 다 있어야 all 계산)
 all_prev : cuh_bsw_all_prev (= center+edge). 한쪽 NaN 이면 NaN
 in_pres : 그 위치가 타겟(PRES)에도 있는지 (False 면 교집합서 제외)
 rule : ldp_r/b_band_y→0강제, fpd_r→150강제, DROPPED→delta테이블서 빠짐

 rank   lot  position  center_prev  edge_prev  all_prev  in_pres ldp_ar fpd_ar b_band  rule
    1 5U461       270         35.0       15.0      50.0     True      R      A      Y ldp_r
    1 5U461       390          NaN        NaN       NaN     True      R      A      Y ldp_r
    1 5U461       590          NaN        NaN       NaN     True      R      A      Y ldp_r
    1 5U461       790          NaN        NaN       NaN     True      R      A      Y ldp_r
    1 5U461       990          NaN        NaN       NaN     True      R      A      Y ldp_r
    1 5U461      1190          NaN        NaN       NaN     True      R      A      N ldp_r
    1 5U461      1390          NaN        NaN       NaN     True      R      A      Y ldp_r
    1 5U461      1590          NaN        NaN       NaN     True      A      A      N  none
    1 5U461      1790         10.0        0.0      10.0     True      A      A      N  none
    1 5U461      1990         15.0        0.0      15.0     True      A      A      N  none
    2 5U460       270         65.0       25.0      90.0     True      A      A      N  none
    2 5U460       390          NaN        NaN       NaN     True      A      A      N  none
    2 5U460       590          NaN        NaN       NaN     True      A      A      N  none
    2 5U460       790          NaN        NaN       NaN     True      A      A      N  none
    2 5U460       990          NaN        NaN       NaN     True      A      A      N  none
    2 5U460      1190          NaN        NaN       NaN     True      A      A      N  none
    2 5U460      1390          NaN        NaN       NaN     True      A      A      N  none
    2 5U460      1590          NaN        NaN       NaN     True      A      A      N  none
    2 5U460      1790        100.0       50.0     150.0     True      A      A      N  none
    2 5U460      1990        100.0       50.0     150.0     True      A      A      N  none
    3 5U459       270         60.0       30.0      90.0     True      A      A      N  none
    3 5U459       390          NaN        NaN       NaN     True      A      A      N  none
    3 5U459       590          NaN        NaN       NaN     True      A      A      N  none
    3 5U459       790          NaN        NaN       NaN     True      A      A      N  none
    3 5U459       990          NaN        NaN       NaN     True      A      A      N  none
    3 5U459      1190          NaN        NaN       NaN     True      A      A      N  none
    3 5U459      1390          NaN        NaN       NaN     True      A      A      N  none
    3 5U459      1590          NaN        NaN       NaN     True      A      A      N  none
    3 5U459      1790        100.0       50.0     150.0     True      A      A      N  none
    3 5U459      1990        100.0       50.0     150.0     True      A      A      N  none
    4 5U458       270        100.0       50.0     150.0     True      A      A      N  none
    4 5U458       390          NaN        NaN       NaN     True      A      A      N  none
    4 5U458       590          NaN        NaN       NaN     True      A      A      N  none
    4 5U458       790          NaN        NaN       NaN     True      A      A      N  none
    4 5U458       990          NaN        NaN       NaN     True      A      A      N  none
    4 5U458      1190          NaN        NaN       NaN     True      A      A      N  none
    4 5U458      1390          NaN        NaN       NaN     True      A      A      N  none
    4 5U458      1590          NaN        NaN       NaN     True      A      A      N  none
    4 5U458      1790         75.0       35.0     110.0     True      A      A      N  none
    4 5U458      1990        100.0       50.0     150.0     True      A      A      N  none
    5 5U457       270         75.0       35.0     110.0     True      A      A      N  none
    5 5U457       390          NaN        NaN       NaN     True      A      A      N  none
    5 5U457       590          NaN        NaN       NaN     True      A      A      N  none
    5 5U457       790          NaN        NaN       NaN     True      A      A      N  none
    5 5U457       990          NaN        NaN       NaN     True      A      A      N  none
    5 5U457      1190          NaN        NaN       NaN     True      A      A      N  none
    5 5U457      1390          NaN        NaN       NaN     True      A      A      N  none
    5 5U457      1590          NaN        NaN       NaN     True      A      A      N  none
    5 5U457      1790        100.0       50.0     150.0     True      A      A      N  none
    5 5U457      1990        100.0       45.0     145.0     True      A      A      N  none

 ℹ️ 0 으로 강제된 위치 (7건, 정상 boundary):
 rank1 pos=270: all=50.0 → 0 (ldp_r)
 rank1 pos=390: all=nan → 0 (ldp_r)
 rank1 pos=590: all=nan → 0 (ldp_r)
 rank1 pos=790: all=nan → 0 (ldp_r)
 rank1 pos=990: all=nan → 0 (ldp_r)
 rank1 pos=1190: all=nan → 0 (ldp_r)
 rank1 pos=1390: all=nan → 0 (ldp_r)

✅ 저장: outputs/tmp.csv
✅ 상세 저장: outputs/tmp_detail.csv
