피처 172개 / 발생 66건
CV recall: 0.44 ± 0.22

[ 분리 규칙 ]
|--- DSP~DSPC Leadtime(min) <= 20.50
|   |--- DSCP 액 Counter <= 8.50
|   |   |--- PCS SC1-1_Temp=60.1 <= 0.50
|   |   |   |--- class: 0
|   |   |--- PCS SC1-1_Temp=60.1 >  0.50
|   |   |   |--- DSCP SC1-2 Temp <= 64.95
|   |   |   |   |--- class: 1
|   |   |   |--- DSCP SC1-2 Temp >  64.95
|   |   |   |   |--- class: 0
|   |--- DSCP 액 Counter >  8.50
|   |   |--- DSP(6100)_LGV장비=P3PLGV08 <= 0.50
|   |   |   |--- FP(6500)_CAR_ID=CD0043 <= 0.50
|   |   |   |   |--- class: 0
|   |   |   |--- FP(6500)_CAR_ID=CD0043 >  0.50
|   |   |   |   |--- class: 1
|   |   |--- DSP(6100)_LGV장비=P3PLGV08 >  0.50
|   |   |   |--- DSCP SC1-2 Temp <= 65.15
|   |   |   |   |--- class: 1
|   |   |   |--- DSCP SC1-2 Temp >  65.15
|   |   |   |   |--- class: 0
|--- DSP~DSPC Leadtime(min) >  20.50
|   |--- DSCP 액 Counter <= 31.50
|   |   |--- PCS SC1-1_NH4OH 농도=0.76 <= 0.50
|   |   |   |--- DSP(6100)_LGV장비=P3PLGV07 <= 0.50
|   |   |   |   |--- class: 1
|   |   |   |--- DSP(6100)_LGV장비=P3PLGV07 >  0.50
|   |   |   |   |--- class: 0
|   |   |--- PCS SC1-1_NH4OH 농도=0.76 >  0.50
|   |   |   |--- PCS 고온DIW_Temp=50.3 <= 0.50
|   |   |   |   |--- class: 0
|   |   |   |--- PCS 고온DIW_Temp=50.3 >  0.50
|   |   |   |   |--- class: 0
|   |--- DSCP 액 Counter >  31.50
|   |   |--- FP_Pad Life <= 286.00
|   |   |   |--- DSCP SC1-1_NH4OH 농도 <= 1.64
|   |   |   |   |--- class: 1
|   |   |   |--- DSCP SC1-1_NH4OH 농도 >  1.64
|   |   |   |   |--- class: 0
|   |   |--- FP_Pad Life >  286.00
|   |   |   |--- FP(6500)장비=P3PFPM04 <= 0.50
|   |   |   |   |--- class: 0
|   |   |   |--- FP(6500)장비=P3PFPM04 >  0.50
|   |   |   |   |--- class: 0


[ 중요 인자 Top 12 ]
DSP~DSPC Leadtime(min)      0.297
DSCP 액 Counter              0.262
PCS SC1-1_NH4OH 농도=0.76     0.073
DSP(6100)_LGV장비=P3PLGV08    0.061
DSP(6100)_LGV장비=P3PLGV07    0.057
FP(6500)_CAR_ID=CD0043      0.055
PCS SC1-1_Temp=60.1         0.053
DSCP SC1-2 Temp             0.052
FP_Pad Life                 0.048
DSCP SC1-1_NH4OH 농도         0.041
PCS 고온DIW_Temp=50.3         0.000
FP(6500)장비=P3PFPM04         0.000
dtype: float64
