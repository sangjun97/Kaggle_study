
[3] filter_valid_data(keep_boundary=True) 후: 18행
 ※ R 통과, cuh 컷 주석, ftir_oi 0~20 컷, length 중복제거
 length   runtime_event ldp_ar fpd_ar cuh_bsw_center cuh_bsw_edge
  390.0  824.0000000000      R      A             40           25
  590.0 1268.0000000000      R      A             20           15
  790.0 1703.0000000000      R      A             20           15
  990.0 2137.0000000000      R      A             25           20
 1190.0 2567.0000000000      R      A             25           10
 1390.0 2995.0000000000      R      A             25           25
 1410.0 3038.0000000000      R      A             25           25

[4] generate_reference_position 후 (ref_length 부여):
 ※ 각 위치 ±25 안 '가장 가까운 length' 1행에 ref_length 부여 (cuh 무관)
 length  ref_length cuh_bsw_center cuh_bsw_edge
  390.0       390.0             40           25
  590.0       590.0             20           15
  790.0       790.0             20           15
  990.0       990.0             25           20
 1190.0      1190.0             25           10
 1390.0      1390.0             25           25

[4-b] 위치별 매칭 분석:
 pos=390: ±25 내 1행, cuh있는행 1개, 매칭행 cuh=40
 pos=590: ±25 내 1행, cuh있는행 1개, 매칭행 cuh=20
 pos=790: ±25 내 1행, cuh있는행 1개, 매칭행 cuh=20
 pos=990: ±25 내 1행, cuh있는행 1개, 매칭행 cuh=25
 pos=1190: ±25 내 1행, cuh있는행 1개, 매칭행 cuh=25
 pos=1390: ±25 내 2행, cuh있는행 2개, 매칭행 cuh=25

[5] extract_at_references: 추출 10 / skip 0
 pos=390: length=390.0, center=40, edge=25
 pos=590: length=590.0, center=20, edge=15
 pos=790: length=790.0, center=20, edge=15
 pos=990: length=990.0, center=25, edge=20
 pos=1190: length=1190.0, center=25, edge=10
 pos=1390: length=1390.0, center=25, edge=25
