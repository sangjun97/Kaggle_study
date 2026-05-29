(venv_new) PS D:\python\CUH\전달용> python .\scripts\diag_fpd.py 6B527
D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
  warnings.warn(

======================================================================
[A. FPD 의 모든 PARAM_PRMPT_NM 분포]
SELECT a."PARAM_PRMPT_NM", COUNT(*) AS cnt
        FROM oracle.oggmgr.TN_SAMP_PARAM a
        WHERE UPPER(a."PARAM_NM") = 'FPD'
          AND a."SAMP_ID" LIKE '6B527%'
        GROUP BY a."PARAM_PRMPT_NM"
        ORDER BY cnt DESC
----------------------------------------------------------------------
행수: 0

======================================================================
[B. FPD MAX 류 PARAM_VAL 샘플]
SELECT a."SAMP_ID", a."PARAM_PRMPT_NM", a."PARAM_VAL"
        FROM oracle.oggmgr.TN_SAMP_PARAM a
        WHERE UPPER(a."PARAM_NM") = 'FPD'
          AND UPPER(a."PARAM_PRMPT_NM") LIKE '%MAX%'
          AND a."SAMP_ID" LIKE '6B527%'
        ORDER BY a."SAMP_ID"
----------------------------------------------------------------------
행수: 0

======================================================================
[C. LDP 의 PARAM_PRMPT_NM 분포]
SELECT a."PARAM_PRMPT_NM", COUNT(*) AS cnt
        FROM oracle.oggmgr.TN_SAMP_PARAM a
        WHERE UPPER(a."PARAM_NM") = 'LDP'
          AND a."SAMP_ID" LIKE '6B527%'
        GROUP BY a."PARAM_PRMPT_NM"
        ORDER BY cnt DESC
----------------------------------------------------------------------
행수: 0

======================================================================
[D. TN_SAMP_MATCH (position) SAMP_ID 개수]
SELECT a."SAMP_ID", a."INGT_LOT_ID", a."BLCK_LOT_ID",
               a."ORDER_PSTN_VAL", a."SAMP_RLTY_PSTN_VAL", a."SAMP_USG_CD"
        FROM oracle.oggmgr.TN_SAMP_MATCH a
        WHERE a."INGT_LOT_ID" LIKE '6B527%'
          AND a."SAMP_USG_CD" = 'RESN'
        ORDER BY a."SAMP_ID"
----------------------------------------------------------------------
행수: 0

======================================================================
[E. FPD ∩ LDP 모두 가진 SAMP_ID]
SELECT DISTINCT fpd."SAMP_ID"
        FROM (
          SELECT DISTINCT a."SAMP_ID"
          FROM oracle.oggmgr.TN_SAMP_PARAM a
          WHERE UPPER(a."PARAM_NM") = 'FPD'
            AND UPPER(a."PARAM_PRMPT_NM") LIKE '%MAX%1%'
            AND a."SAMP_ID" LIKE '6B527%'
            AND TRIM(a."PARAM_VAL") IS NOT NULL
            AND TRIM(a."PARAM_VAL") != ''
        ) fpd
        INNER JOIN (
          SELECT DISTINCT a."SAMP_ID"
          FROM oracle.oggmgr.TN_SAMP_PARAM a
          WHERE UPPER(a."PARAM_NM") = 'LDP'
            AND a."SAMP_ID" LIKE '6B527%'
            AND TRIM(a."PARAM_VAL") IS NOT NULL
            AND TRIM(a."PARAM_VAL") != ''
        ) ldp ON fpd."SAMP_ID" = ldp."SAMP_ID"
        ORDER BY fpd."SAMP_ID"
----------------------------------------------------------------------
행수: 0

======================================================================
[F. SAMP_ID 첫 5자리 vs 전체 길이]
SELECT a."SAMP_ID", LENGTH(a."SAMP_ID") AS samp_len,
               SUBSTR(a."SAMP_ID", 1, 5) AS prefix5
        FROM oracle.oggmgr.TN_SAMP_PARAM a
        WHERE UPPER(a."PARAM_NM") = 'FPD'
          AND a."SAMP_ID" LIKE '6B527%'
        GROUP BY a."SAMP_ID", LENGTH(a."SAMP_ID"), SUBSTR(a."SAMP_ID", 1, 5)
        ORDER BY a."SAMP_ID"
----------------------------------------------------------------------
행수: 0
