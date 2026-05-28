
(venv_new) PS D:\python\CUH\전달용> python .\scripts\release\run_apc.py --lot 6H825
D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
  warnings.warn(
📋 release_params: 10 positions, target=80.0

============================================================
📡 Lot 기반 데이터 수집: target=6H825, n_runs=5
============================================================
 [PRES] 6H825 FDC+3340 조회...
 → 6,728 행
 [PREV] 후보 24 개 (prefix=6H8)
 ✗ 6H824: cuh 없음 → skip
 ✓ 6H823: cuh 유효 (6,697 행) [1/5]
 ✓ 6H822: cuh 유효 (6,703 행) [2/5]
 ✓ 6H821: cuh 유효 (7,244 행) [3/5]
 ✓ 6H820: cuh 유효 (9,041 행) [4/5]
 ✓ 6H819: cuh 유효 (6,759 행) [5/5]

 📊 수집 완료: PRES 1개 + PREV 5개 (요청 5개)

######################################################################
# [LOT 모드] lot=6H825, grower=TG128, PREV 5개
######################################################################

=== PREV 후보 (blend=5개, mode=weighted) ===
 rank lot_number   gr_out  usage                role
    1      6H823 20260514      1 blend + ps_set_prev
    2      6H822 20260509      2               blend
    3      6H821 20260504      3               blend
    4      6H820 20260429      4               blend
    5      6H819 20260423      5               blend
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0014 ~ 0.0021] (clip: [-0.0070 ~ 0.0070], 밖 0/10건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [-0.9079 ~ -0.8190] (clip: [-3.0000 ~ 3.0000], 밖 0/10건)
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0020 ~ 0.0017] (clip: [-0.0070 ~ 0.0070], 밖 0/10건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [-0.6487 ~ -0.4715] (clip: [-3.0000 ~ 3.0000], 밖 0/10건)
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0012 ~ 0.0027] (clip: [-0.0070 ~ 0.0070], 밖 0/10건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [-1.1110 ~ -1.0207] (clip: [-3.0000 ~ 3.0000], 밖 0/10건)
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0010 ~ 0.0035] (clip: [-0.0070 ~ 0.0070], 밖 0/10건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [-0.7211 ~ -0.5182] (clip: [-3.0000 ~ 3.0000], 밖 0/10건)
  📐 입력 X 값 vs clip 범위:
       pull_speed_t100_delta: [-0.0008 ~ 0.0023] (clip: [-0.0070 ~ 0.0070], 밖 0/10건)
       ftir_oi_center_delta: [nan ~ nan] (clip: [-0.8000 ~ 0.8000], 밖 0/0건)
       dia_l50_delta: [0.4383 ~ 0.6363] (clip: [-3.0000 ~ 3.0000], 밖 0/10건)

=== APC 추천 (Zero baseline, target=80.0) ===
grower: TG128
pres_lot: 6H825
prev (K=5 weighted):
 rank lot_number   gr_out                role
    1      6H823 20260514 blend + ps_set_prev
    2      6H822 20260509               blend
    3      6H821 20260504               blend
    4      6H820 20260429               blend
    5      6H819 20260423               blend

결과:
 position  ps_set_prev  ps_delta_zero  ps_correction_value  ps_set_proposed
      270      0.45730       -0.00661              0.00000          0.45069
      390      0.46121       -0.00661              0.00000          0.45460
      590      0.47363       -0.00700              0.00000          0.46663
      790      0.47798       -0.00700              0.00000          0.47098
      990      0.48105       -0.00700              0.00000          0.47405
     1190      0.48079       -0.00700              0.00000          0.47379
     1390      0.48394       -0.00700              0.00000          0.47694
     1590      0.48472       -0.00700              0.00000          0.47772
     1790      0.48382       -0.00700              0.00000          0.47682
     1990      0.48342       -0.00442              0.00000          0.47900

✅ 저장: outputs\release_apc\6H825_apc_recommendation.csv

======================================================================
📊 LOT 모드 완료: 성공 (lot=6H825)
======================================================================
