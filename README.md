======================================================================
# APC by lot: 6F626
======================================================================
🎯 target lot 조회: 6F626
 target 7178행, USAGE 컬럼 9개, gr_out=20260526, customer=Smart_SK hynix/GF/CXMT 440
🔎 lot 시퀀스 조회: prefixes=['6F6', '5F6']
 이전 lot 후보 90개 (최신순)
 - 6F625: FDC 없음, skip
 ✅ 6F624: 수집 (1/5, 6175행)
 ✅ 6F623: 수집 (2/5, 5799행)
 ✅ 6F622: 수집 (3/5, 7049행)
 - 6F620: USAGE 페어 탈락 (USAGE 감소 컬럼: ['crucible_2d']) → 이후 후보도 USAGE 더 벌어지므로 탐 색 중단
 skip 통계: FDC없음=1, cuh없음=0, gr_out=0, customer=0, USAGE탈락=1
 ⚠️ 후보 소진: 3/5 run 만 확보
 grower DF 구성: 27행, 잉곳 3개 (요청 run 3개)

📊 컨텍스트: grower=TG106, prev 3/5 run (lots=['6F624', '6F623', '6F622']), target 7178행
 ⚠️ 5 run 미만 — 모은 3개로 진행
 ℹ️ blend_prevs=5 → 3 로 조정
  PREV 후보 수집: 4 개 (검사 4 개, skip 0 개)

=== PREV 후보 (blend=3개, mode=weighted) ===
 rank lot_number   gr_out  usage                role
    1      6F624 20260514      2 blend + ps_set_prev
    2      6F624 20260514      2               blend
    3      6F623 20260510      3               blend
    4      6F622 20260505      4           reference
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [0.0020 ~ 0.0020] (clip: [-0.0070 ~ 0.0070], 밖 0/1건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [1.3089 ~ 1.3089] (clip: [-3.0000 ~ 3.0000], 밖 0/1건)
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [0.0015 ~ 0.0015] (clip: [-0.0070 ~ 0.0070], 밖 0/1건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [1.3811 ~ 1.3811] (clip: [-3.0000 ~ 3.0000], 밖 0/1건)
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [0.0001 ~ 0.0028] (clip: [-0.0070 ~ 0.0070], 밖 0/7건)
       ftir_oi_center_delta: [-0.1300 ~ 0.2400] (clip: [-0.8000 ~ 0.8000], 밖 0/4건)
       dia_l50_delta: [-0.3029 ~ 0.6262] (clip: [-3.0000 ~ 3.0000], 밖 0/7건)

=== APC 추천 (Zero baseline, target=80.0) ===
grower: TG106
pres_lot: 6F626
prev (K=3 weighted):
 rank lot_number   gr_out                role
    1      6F624 20260514 blend + ps_set_prev
    2      6F624 20260514               blend
    3      6F623 20260510               blend
    4      6F622 20260505           reference

결과:
 position  ps_set_prev  ps_delta_zero  ps_correction_value  ps_set_proposed
      270      0.45501       -0.00380              0.00000          0.45121
      390      0.45880       -0.00060              0.00000          0.45820
      590      0.47215        0.00300              0.00000          0.47515
      790      0.47599        0.00100              0.00000          0.47699
      990      0.48280        0.00100              0.00000          0.48380
     1190      0.48270        0.00200              0.00000          0.48470
     1390      0.48180       -0.00700              0.00000          0.47480

✅ 저장: outputs\release_apc\6F626_apc_recommendation.csv
