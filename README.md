======================================================================
# APC by lot: 6B527
======================================================================
🎯 target lot 조회: 6B527
 target 4698행, USAGE 컬럼 9개, gr_out=20260528, customer=Smart_SK hynix/GF/CXMT 440
🔎 lot 시퀀스 조회: prefixes=['6B5', '5B5']
 이전 lot 후보 89개 (최신순)
 ✅ 6B526: 수집 (1/5, 6947행)
 ✅ 6B525: 수집 (2/5, 6672행)
 ✅ 6B524: 수집 (3/5, 8123행)
 ✅ 6B523: 수집 (4/5, 7102행)
 ✅ 6B522: 수집 (5/5, 6630행)
 skip 통계: FDC없음=0, cuh없음=0, gr_out=0, customer=0, USAGE탈락=0
 grower DF 구성: 51행, 잉곳 5개 (요청 run 5개)

📊 컨텍스트: grower=TG75, prev 5/5 run (lots=['6B526', '6B525', '6B524', '6B523', '6B522']), target 4698행
  PREV 후보 수집: 6 개 (검사 6 개, skip 0 개)

=== PREV 후보 (blend=5개, mode=weighted) ===
 rank lot_number   gr_out  usage                role
    1      6B526 20260525      1 blend + ps_set_prev
    2      6B525 20260520      2               blend
    3      6B524 20260514      3               blend
    4      6B524 20260514      3               blend
    5      6B523 20260509      4               blend
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0012 ~ 0.0011] (clip: [-0.0070 ~ 0.0070], 밖 0/4건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [-0.2168 ~ -0.0570] (clip: [-3.0000 ~ 3.0000], 밖 0/4건)
 ⚠️ rank=2: Δ 테이블 비어있음 → skip
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0023 ~ 0.0032] (clip: [-0.0070 ~ 0.0070], 밖 0/3건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [0.4673 ~ 0.8942] (clip: [-3.0000 ~ 3.0000], 밖 0/3건)
 ⚠️ rank=4: Δ 테이블 비어있음 → skip
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0009 ~ 0.0025] (clip: [-0.0070 ~ 0.0070], 밖 0/3건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [-0.4639 ~ -0.0128] (clip: [-3.0000 ~ 3.0000], 밖 0/3건)

=== APC 추천 (Zero baseline, target=80.0) ===
grower: TG75
pres_lot: 6B527
prev (K=3 weighted):
 rank lot_number   gr_out                role
    1      6B526 20260525 blend + ps_set_prev
    2      6B525 20260520               blend
    3      6B524 20260514               blend
    4      6B524 20260514               blend
    5      6B523 20260509               blend

결과:
 position  ps_set_prev  ps_delta_zero  ps_correction_value  ps_set_proposed
      270      0.45045        0.00129              0.00000          0.45173
      390      0.46218       -0.00214              0.00000          0.46004
      590      0.47164       -0.00371              0.00000          0.46793
      790      0.47669       -0.00500              0.00000          0.47169
      990      0.47838       -0.00200              0.00000          0.47638
     1190      0.48106       -0.00200              0.00000          0.47906
     1390      0.48469        0.00000              0.00000          0.48469

✅ 저장: outputs\release_apc\6B527_apc_recommendation.csv
