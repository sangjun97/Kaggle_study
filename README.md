(venv_new) PS D:\python\CUH\전달용> python .\scripts\diag_fpd.py
파일: D:\python\CUH\전달용\fdc_3340_lot_basis_20260529_105443.csv
전체 행수: 53,441

=== fpd_ar 분포 (전체) ===
fpd_ar
NaN    53063
A        378
Name: count, dtype: int64

=== lot 별 fpd_ar 채워진 행수 ===
 6B527: 0 / 4698 행
 6B522: 54 / 6630 행
 6B523: 71 / 7102 행
 6B520: 55 / 6638 행
 6B524: 70 / 8123 행
 6B521: 63 / 6633 행
 6B526: 32 / 6945 행
 6B525: 33 / 6672 행

=== 6B527 에서 samp_id 채워진 행 ===
(6B527 의 samp_id 채워진 행 없음)

======================================================================
[1. SAMP_ID 안에 lot 포함된 행 (전체)]
SELECT DISTINCT a."SAMP_ID", a."PARAM_NM", a."PARAM_SET_ID"
        FROM oracle.oggmgr.TN_SAMP_PARAM a
        WHERE a."SAMP_ID" LIKE '%6B527%'
        LIMIT 30
----------------------------------------------------------------------
행수: 30
   samp_id param_nm param_set_id
9Z016B5272      RES         RESN
9Z016B5272      RRG         RESN
9Z016B5272       OI         RESN
9Z016B5272      ORG         RESN
9Z016B5272       CS         RESN
9Z016B5272    CRACK         RESN
9Z016B5272      CUH         RESN
9Z016B5272      FPD         RESN
9Z016B5272      LDP         RESN
9Z016B5272     SLIP         RESN
9Z016B5272     TWIN         RESN
0Z456B5270      FPD         RESN
0Z456B5270      LDP         RESN
0Z456B5270      CUH         RESN
1F626B5272      RES         RESN
1F626B5272      RRG         RESN
1F626B5272       OI         RESN
1F626B5272      ORG         RESN
1F626B5272       CS         RESN
1F626B5272    CRACK         RESN
1F626B5272      FPD         RESN
1F626B5272      LDP         RESN
1F626B5272     SLIP         RESN
1F626B5272     TWIN         RESN
1F626B5272   B-BAND         RESN
1F626B5272      CUH         RESN
2B936B5279       CS         RESN
2B936B5279    CRACK         RESN
2B936B5279      CUH         RESN
2B936B5279      FPD         RESN

======================================================================
[2. TN_SAMP_MATCH 의 INGT_LOT_ID 안에 lot]
SELECT DISTINCT a."INGT_LOT_ID", a."SAMP_ID", a."SAMP_USG_CD", a."BLCK_LOT_ID"
        FROM oracle.oggmgr.TN_SAMP_MATCH a
        WHERE a."INGT_LOT_ID" LIKE '%6B527%'
        LIMIT 30
----------------------------------------------------------------------
행수: 0

======================================================================
[3. TN_SAMP_MATCH 의 SAMP_ID 안에 lot]
SELECT DISTINCT a."INGT_LOT_ID", a."SAMP_ID", a."SAMP_USG_CD"
        FROM oracle.oggmgr.TN_SAMP_MATCH a
        WHERE a."SAMP_ID" LIKE '%6B527%'
        LIMIT 30
----------------------------------------------------------------------
행수: 9
ingt_lot_id    samp_id samp_usg_cd
      9Z016 9Z016B5272        RESN
      1F626 1F626B5272        RESN
      0Z456 0Z456B5270        RESN
      2B936 2B936B5279        RESN
      3A606 3A606B5275        RESN
      3Z626 3Z626B5270        RESN
      5X126 5X126B5274        OISF
      5X126 5X126B5274        RESN
      5X126 5X126B5273         CSD

======================================================================
[4. 최근 FPD MAX SAMP_ID 샘플 (lot 무관) - 형식 참고]
SELECT a."SAMP_ID", a."PARAM_PRMPT_NM", a."PARAM_VAL"
        FROM oracle.oggmgr.TN_SAMP_PARAM a
        WHERE UPPER(a."PARAM_NM") = 'FPD'
          AND UPPER(a."PARAM_PRMPT_NM") LIKE '%MAX%'
        ORDER BY a."SAMP_ID" DESC
        LIMIT 20
----------------------------------------------------------------------
행수: 20
   samp_id param_prmpt_nm param_val
9Z968J9292           MAX2     0.000
9Z968J9292           MAX1     0.000
9Z968J9203           MAX2     0.000
9Z968J9203           MAX1     0.000
9Z968J9106           MAX2     0.000
9Z968J9106           MAX1     0.000
9Z968J9000           MAX2     0.000
9Z968J9000           MAX1     0.000
9Z968F7305           MAX1     0.000
9Z968F7305           MAX2     0.000
9Z968F7203           MAX2     0.000
9Z968F7203           MAX1     0.000
9Z968F7096           MAX2     0.000
9Z968F7096           MAX1     0.000
9Z968F7001           MAX1     0.000
9Z968F7001           MAX2     0.000
9Z968C5999           MAX1     0.000
9Z968C5999           MAX2     0.000
9Z968C5194           MAX2     0.000
9Z968C5194           MAX1     0.000
