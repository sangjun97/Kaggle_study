(venv_new) PS D:\python\CUH\growing_apc_LOW> python .\scripts\release\sim.py --mode train --data .\data\dataset\delta_train.csv   

============================================================
# [A] 학습 데이터 — BSW100 규칙의 포화 기여
============================================================
 ⚠️ cuh_bsw_center/edge_prev 없음 → 원본 복원 불가.
 현재 cuh_bsw_all_prev=150 개수: 422 / 1109
 (원본 center/edge 있는 파일이어야 규칙 분리 분석 가능)
(venv_new) PS D:\python\CUH\growing_apc_LOW> python .\scripts\release\sim.py --mode val --data .\outputs\validation_cuhh_lowoi_2.csv

============================================================
# [B] 검증 데이터 — 규칙 O/X MAE 비교
============================================================

 [규칙 O (현재)]
 실측 포화(150): 110건
 전체 MAE: 42.31 (n=246)
 포화 위치 MAE: 51.28 (n=106)
 비포화 위치 MAE: 35.52 (n=140)

 [규칙 X (BSW100 제거)]
 실측 포화(150): 62건
 전체 MAE: 39.04 (n=246)
 포화 위치 MAE: 51.90 (n=59)
 비포화 위치 MAE: 34.99 (n=187)

 ⚠️ 주의: cuh_pred 는 '규칙 O' 로 학습된 모델 산출물.
 규칙 X MAE 는 실측만 되돌린 근사치 — 방향성 참고용.
 규칙 X 에서 MAE 개선폭이 크면 재학습 가치 있음.
