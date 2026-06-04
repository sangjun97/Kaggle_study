============================================================
Raw shape : (2392, 80)
Y 매칭 1/1 : ['구분']
X 매칭 38/38
============================================================
중복제거: 2392 -> 2392 rows

[Y 원본 분포]
구분
미발생    2326
발생       66
Name: count, dtype: int64

발생 66건 / 유효 2392건 → 발생율 2.76%

연속형 X 22개
범주형 X 16개 → ['DSP(6100)_LGV장비', 'FP(6500)장비', 'PCS(6700)SUBLOT_ID', 'PCS(6700)장비', 'FP_bath', 'FP(6500)_CAR_ID', 'FP(6500)_LGV 장비', 'FP(6500)_LGV 투입 시점', 'DSPC(6200)SUBLOT_ID', 'DSCP SC1-2 Processtime', 'PCS SC1-1_NH4OH 농도', 'PCS SC1-2_NH4OH 농도', 'PCS SC1-2_Processtime', 'PCS SC1-1_Temp', 'PCS SC1-2_Temp', 'PCS 고온DIW_Temp']

[결측율 상위 10]
DSCP SC1-1_H2O2 농도        6.4
DSCP SC1-1 Temp           6.4
DSCP SC1-2 Temp           6.4
DSCP SC1-1_NH4OH 농도       6.4
DSCP SC1-2_H2O2 농도        6.4
DSCP 액 Counter            6.4
DSCP SC1-2 Processtime    6.4
DSCP SC1-2_NH4OH 농도       6.4
DSCP SC1-1 Processtime    6.4
PCS SC1-1_H2O2 농도         5.1
dtype: float64

[월별 발생율]
           샘플  발생   발생율
_month
2026-05  1156  33  2.85
2026-06  1236  33  2.67
