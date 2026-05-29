# 하이실트론 사번
# 비밀번호
# 입력 요청드립니다.
# %% [markdown]
# # SK Siltron - Lot Basis Data Extraction & Join (Code Input Mode)
# # 방식: 코드 내 TARGET_LOT_NUMBERS 변수에 직접 입력 후 실행

# %%
## 초기 셋팅 코드
import trino
import pandas as pd
import json
import sys
from datetime import datetime
import warnings
from pathlib import Path
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
warnings.filterwarnings('ignore', message='Unverified HTTPS request')

# ==============================================================================
# [사용자 입력 구역] - 여기에 조회할 Lot Number 를 입력하세요
# ==============================================================================
# 형식: ['Lot1', 'Lot2', 'Lot3'] 처럼 쉼표로 구분하여 입력
# 예시: ['24A001', '24A002', '24B005']
# 주의: 공란으로 두면 데이터가 추출되지 않습니다. 반드시 값을 입력해주세요.
TARGET_LOT_NUMBERS = [ '6B520', '6B521', '6B522', '6B523', '6B524', '6B525', '6B526', '6B527'

    # 여기에 Lot 번호를 입력하세요 (예: '26A001', '26A002')
]
# ==============================================================================

# 접속 정보 설정
HOST = 'aidp-trino-analysis.sksiltron.co.kr'
PORT = 31085
USER = '257103'               # 하이실트론 사번
PASSWORD = 'lwj2110!'         # 비밀번호
CATALOG = 'iceberg'
SCHEMA = 'ibg_lake'

# ==============================================================================
# Trino 연결 생성 함수
# ==============================================================================
def create_trino_connection():
    """Trino DB 에 안전하게 연결"""
    return trino.dbapi.connect(
        host=HOST,
        port=PORT,
        user=USER,
        http_scheme='https',
        auth=trino.auth.BasicAuthentication(USER, PASSWORD),
        catalog=CATALOG,
        schema=SCHEMA,
        verify=False
    )

# ==============================================================================
# EXPLAIN 으로 IO 통계 확인 및 대용량 경고
# ==============================================================================
def safe_float(val, default=0.0):
    if isinstance(val, (int, float)):
        return float(val)
    if isinstance(val, str):
        if val.strip().lower() == "nan":
            return float('nan')
        try:
            return float(val)
        except ValueError:
            return default
    return default

def check_data_size_before_query(conn, query):
    """쿼리의 예상 출력 크기를 확인"""
    explain_query = f"EXPLAIN (TYPE IO, FORMAT JSON) {query}"
    cur = conn.cursor()

    try:
        print("🔍 EXPLAIN 쿼리 실행 중...")
        cur.execute(explain_query)
        explain_result = cur.fetchall()

        json_str = explain_result[0][0]
        io_stats = json.loads(json_str)

        input_tables = io_stats.get("inputTableColumnInfos", [])
        total_input_gb = 0.0
        
        global_estimate = io_stats.get("estimate", {})
        output_size_bytes = safe_float(global_estimate.get("outputSizeInBytes"), float('nan'))
        output_size_gb = output_size_bytes / (1024 ** 3) if not pd.isna(output_size_bytes) else float('nan')

        # 입력 기반 추정 (출력 추정이 안 될 경우)
        if pd.isna(output_size_gb):
            for table_info in input_tables:
                estimate = table_info.get("estimate", {})
                size_bytes = safe_float(estimate.get("outputSizeInBytes"), 0)
                total_input_gb += size_bytes / (1024 ** 3)
            check_size = total_input_gb
            print(f"📊 예상 스캔 크기 (입력 기준): {check_size:.3f} GB")
        else:
            check_size = output_size_gb
            print(f"📊 예상 출력 크기: {check_size:.3f} GB")

        if check_size > 1.0:
            print(f"\n🚨 경고: 예상 데이터 크기가 {check_size:.3f} GB 로 1GB 를 초과합니다.")
            confirm = input("계속 진행하시겠습니까? (y/N): ").strip().lower()
            if confirm not in ['y', 'yes']:
                print("⛔ 사용자에 의해 쿼리 취소됨.")
                sys.exit(0)
        else:
            print("✅ 용량 확인 완료.")

    except Exception as e:
        print(f"❌ EXPLAIN 분석 중 오류 발생: {e}")
        confirm = input("EXPLAIN 실패. 그래도 쿼리 실행하시겠습니까? (y/N): ").strip().lower()
        if confirm not in ['y', 'yes']:
            sys.exit(0)
    finally:
        cur.close()

# ==============================================================================
# 데이터 추출 헬퍼 함수
# ==============================================================================
def fetch_data_to_df(query):
    """Trino 쿼리 실행 후 DataFrame 으로 반환"""
    conn = create_trino_connection()
    cursor = conn.cursor()
    try:
        cursor.execute(query)
        rows = cursor.fetchall()
        columns = [desc[0] for desc in cursor.description]
        return pd.DataFrame(rows, columns=columns)
    finally:
        cursor.close()
        conn.close()

# ==============================================================================
# 1 단계: dw_fa_eq_fdcxygr_i 에서 Lot Number 기반 데이터 추출
# ==============================================================================
def extract_fdc_data_by_lots(lot_list):
    if not lot_list:
        raise ValueError("추출할 Lot Number 리스트가 비어 있습니다.")

    exclude_columns = [
        'ftir_oi_center', 'ftir_oi_edge1', 'ftir_oi_edge2', 'ftir_oi_edge3', 'ftir_oi_edge4',
        'ftir_eqp_id', 'cuh_bp_center', 'cuh_bp_edge', 'cuh_bp_all',
        'cuh_bsw_center', 'cuh_bsw_edge', 'cuh_bsw_all',
        'fpd_ar', 'fpd_score', 'ldp_ar', 'ldp_score'
    ]
    
    all_columns = [
        'gr_out', 'area', 'grower', 'lot_number', 'runtime_event', 'runtime_all',
        'work_time', 'mscode', 'customer_grp_1', 'customer_grp_2', 'cust_nm',
        'current_event', 'attempt', 'max_attempt', 'ingot_status', 'st_loss_pos',
        'st_loss_label', 'length', 'dia', 'adc_filter', 'adc_position_nor',
        'ssrp_sho_dia', 'ssrp_sho_dia_error', 'a_chamber_flow', 'b_chamber_flow',
        'a_chamber_temp', 'b_chamber_temp', 'water_jacket_1_flow', 'water_jacket_2_flow',
        'water_jacket_1_temp', 'water_jacket_2_temp', 'ar_flow', 'ar_gas_no2',
        'a_chamber_pressure', 'throttle_valve_open', 'actual_pull_speed',
        'actual_pull_speed_gap', 'actual_clr', 'crucible_lift', 'crucible_position',
        'crucible_rotation', 'seed_position', 'seed_rotation', 'melt_gap',
        'melt_gap_delta', 'melt_temp', 'atc_actual_value', 'body_profile',
        'body_profile_gap', 'heater_power', 'heater_resistance', 'st_heater_power',
        'actual_magnet_intensity', 'all_cleaning', 'exhaust_line_cleaning',
        'crucible_2d', 'heater_cap', 'nop_inner', 'nop_outer_lower', 'nop_outer_upper',
        'nop_shield', 'nop_tube', 'side_heater', 'st_heater', 'top_ring',
        'uls_heat_cap', 'upper_ring', 'upper_tube_l', 'upper_tube_u', 'frs_oi',
        'frs_eqp_id', 'pull_speed', 'magnet_position', 'ar_gas_no3',
        'hsys_upd_dtm', 'pt_d', 'renamed_grower', 'renamed_lot_number'
    ]
    
    columns_sql = ', '.join([f'"{col}"' for col in all_columns])
    lots_sql = ", ".join([f"'{lot.strip()}'" for lot in lot_list if lot.strip()])
    
    query = f"""
    SELECT {columns_sql}
    FROM iceberg.ibg_lake.dw_fa_eq_fdcxygr_i
    WHERE lot_number IN ({lots_sql})
      AND lot_number IS NOT NULL
    """
    
    print(f"🔍 [1 단계] dw_fa_eq_fdcxygr_i 데이터 추출 중... (대상 Lot 수: {len(lot_list)})")
    
    try:
        df = fetch_data_to_df(query)
        print(f"✅ 전체 데이터 로드 완료 | 행: {len(df):,}, 열: {len(df.columns)}")
        
        if df.empty:
            print("⚠️ 조회된 데이터가 없습니다. Lot 번호를 확인해주세요.")
            return df, []
            
        unique_lots = df['lot_number'].dropna().astype(str).unique().tolist()
        print(f"✅ 실제 포함된 고유 lot_number 수: {len(unique_lots):,}")
        
        return df, sorted(unique_lots)
    except Exception as e:
        print(f"❌ 데이터 추출 실패: {e}")
        raise

# ==============================================================================
# 2 단계: Oracle 에서 3340_data 추출
# ==============================================================================
def extract_3340_data_precise(lot_prefix_list):
    if not lot_prefix_list:
        raise ValueError("lot_prefix_list 가 비어 있습니다.")

    like_conditions = " OR ".join([f'b."SAMP_ID" LIKE \'{lot}%\'' for lot in lot_prefix_list if lot])
    
    query = f"""
    WITH 
    ftir_data AS (
        SELECT 
            b."SAMP_ID",
            MAX(CASE WHEN b."PARAM_NM" = 'OI'  AND b."PARAM_PRMPT_NM" = 'CENTER'   THEN TRIM(b."PARAM_VAL") END) AS FTIR_OI_CENTER,
            MAX(CASE WHEN b."PARAM_NM" = 'OI'  AND b."PARAM_PRMPT_NM" = 'EDGE1-1'  THEN TRIM(b."PARAM_VAL") END) AS FTIR_OI_EDGE1_1,
            MAX(CASE WHEN b."PARAM_NM" = 'OI'  AND b."PARAM_PRMPT_NM" = 'EDGE1-2'  THEN TRIM(b."PARAM_VAL") END) AS FTIR_OI_EDGE1_2,
            MAX(CASE WHEN b."PARAM_NM" = 'OI'  AND b."PARAM_PRMPT_NM" = 'EDGE1-3'  THEN TRIM(b."PARAM_VAL") END) AS FTIR_OI_EDGE1_3,
            MAX(CASE WHEN b."PARAM_NM" = 'OI'  AND b."PARAM_PRMPT_NM" = 'EDGE1-4'  THEN TRIM(b."PARAM_VAL") END) AS FTIR_OI_EDGE1_4,
            MAX(CASE WHEN b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" = 'CENTER'   THEN TRIM(b."PARAM_VAL") END) AS CUH_BP_CENTER,
            MAX(CASE WHEN b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" = 'EDGE'     THEN TRIM(b."PARAM_VAL") END) AS CUH_BP_EDGE,
            MAX(CASE WHEN b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" = 'ALL'      THEN TRIM(b."PARAM_VAL") END) AS CUH_BP_ALL,
            MAX(CASE WHEN b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" = 'BSW_CENTER' THEN TRIM(b."PARAM_VAL") END) AS CUH_BSW_CENTER,
            MAX(CASE WHEN b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" = 'BSW_EDGE'   THEN TRIM(b."PARAM_VAL") END) AS CUH_BSW_EDGE,
            MAX(CASE WHEN b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" = 'BSW_ALL'    THEN TRIM(b."PARAM_VAL") END) AS CUH_BSW_ALL
        FROM ORACLE.OGGMGR.TN_SAMP_PARAM b
        WHERE 
            b."PARAM_SET_ID" = 'RESN'
            AND b."PARAM_NM" IN ('OI', 'CUH')
            AND (
                (b."PARAM_NM" = 'OI'  AND b."PARAM_PRMPT_NM" IN ('CENTER', 'EDGE1-1', 'EDGE1-2', 'EDGE1-3', 'EDGE1-4'))
                OR 
                (b."PARAM_NM" = 'CUH' AND b."PARAM_PRMPT_NM" IN ('CENTER', 'EDGE', 'ALL', 'BSW_CENTER', 'BSW_EDGE', 'BSW_ALL'))
            )       
            AND ({like_conditions})
            AND TRIM(b."PARAM_VAL") != '*'
            AND TRIM(b."PARAM_VAL") != ''
            AND REGEXP_LIKE(TRIM(b."PARAM_VAL"), '^-?[0-9]+(\\.[0-9]+)?$')
        GROUP BY b."SAMP_ID"
    ),
    fpd_ldp_data AS (
        SELECT 
            fpd."SAMP_ID", fpd.FPD_MAX1_VALUE, ldp.LDP_sum, bband.B_BAND_VALUE
        FROM (
            SELECT a."SAMP_ID", CAST(TRIM(REPLACE(REPLACE(a."PARAM_VAL", '*', ''), ' ', '')) AS DECIMAL(18,10)) AS FPD_MAX1_VALUE
            FROM oracle.oggmgr.TN_SAMP_PARAM a 
            WHERE UPPER(a."PARAM_NM") = 'FPD' AND UPPER(a."PARAM_PRMPT_NM") LIKE '%MAX%1%'
              AND ({like_conditions.replace('b."SAMP_ID"', 'a."SAMP_ID"')})
              AND TRIM(a."PARAM_VAL") IS NOT NULL AND TRIM(a."PARAM_VAL") != ''
              AND REGEXP_LIKE(TRIM(REPLACE(REPLACE(a."PARAM_VAL", '*', ''), ' ', '')), '^-?[0-9]+\\.?[0-9]*$')
        ) fpd
        INNER JOIN (
            SELECT a."SAMP_ID", SUM(CAST(TRIM(REPLACE(REPLACE(a."PARAM_VAL", '*', ''), ' ', '')) AS DECIMAL(18,10))) AS LDP_sum
            FROM oracle.oggmgr.TN_SAMP_PARAM a
            WHERE UPPER(a."PARAM_NM") = 'LDP' AND ({like_conditions.replace('b."SAMP_ID"', 'a."SAMP_ID"')})
              AND TRIM(a."PARAM_VAL") IS NOT NULL AND TRIM(a."PARAM_VAL") != ''
              AND REGEXP_LIKE(TRIM(REPLACE(REPLACE(a."PARAM_VAL", '*', ''), ' ', '')), '^-?[0-9]+\\.?[0-9]*$')
            GROUP BY a."SAMP_ID"
        ) ldp ON fpd."SAMP_ID" = ldp."SAMP_ID"
        LEFT JOIN (
            SELECT "SAMP_ID", B_BAND_VALUE FROM (
                SELECT a."SAMP_ID", TRIM(REPLACE(REPLACE(a."PARAM_VAL", '*', ''), ' ', '')) AS B_BAND_VALUE,
                ROW_NUMBER() OVER (PARTITION BY a."SAMP_ID" ORDER BY CASE WHEN a."CRT_DT" IS NOT NULL THEN 0 ELSE 1 END, a."CRT_DT" DESC NULLS LAST) AS rn
                FROM oracle.oggmgr.TN_SAMP_PARAM a
                WHERE UPPER(a."PARAM_NM") = 'B-BAND' AND ({like_conditions.replace('b."SAMP_ID"', 'a."SAMP_ID"')})
                  AND a."PARAM_VAL" IS NOT NULL AND TRIM(a."PARAM_VAL") != ''
            ) ranked WHERE rn = 1
        ) bband ON fpd."SAMP_ID" = bband."SAMP_ID"
    ),
    pos_data AS (
        SELECT 
            t."SAMP_ID", t."BLCK_LOT_ID", t."ORDER_PSTN_VAL", t."SAMP_RLTY_PSTN_VAL",
            ((CASE SUBSTR(blck_last2, 1, 1)
                WHEN 'A' THEN 10 WHEN 'B' THEN 11 WHEN 'C' THEN 12 WHEN 'D' THEN 13
                WHEN 'E' THEN 14 WHEN 'F' THEN 15 WHEN 'G' THEN 16 WHEN 'H' THEN 17
                WHEN 'J' THEN 18 WHEN 'K' THEN 19 WHEN 'L' THEN 20 WHEN 'M' THEN 21
                WHEN 'N' THEN 22 ELSE CAST(SUBSTR(blck_last2, 1, 1) AS INTEGER) END * 10)
             + CAST(SUBSTR(blck_last2, 2, 1) AS INTEGER)) * 10 AS block_pos,
            ((CASE SUBSTR(blck_last2, 1, 1)
                WHEN 'A' THEN 10 WHEN 'B' THEN 11 WHEN 'C' THEN 12 WHEN 'D' THEN 13
                WHEN 'E' THEN 14 WHEN 'F' THEN 15 WHEN 'G' THEN 16 WHEN 'H' THEN 17
                WHEN 'J' THEN 18 WHEN 'K' THEN 19 WHEN 'L' THEN 20 WHEN 'M' THEN 21
                WHEN 'N' THEN 22 ELSE CAST(SUBSTR(blck_last2, 1, 1) AS INTEGER) END * 10)
             + CAST(SUBSTR(blck_last2, 2, 1) AS INTEGER)) * 10 + t."ORDER_PSTN_VAL" AS order_pos,
            ((CASE SUBSTR(blck_last2, 1, 1)
                WHEN 'A' THEN 10 WHEN 'B' THEN 11 WHEN 'C' THEN 12 WHEN 'D' THEN 13
                WHEN 'E' THEN 14 WHEN 'F' THEN 15 WHEN 'G' THEN 16 WHEN 'H' THEN 17
                WHEN 'J' THEN 18 WHEN 'K' THEN 19 WHEN 'L' THEN 20 WHEN 'M' THEN 21
                WHEN 'N' THEN 22 ELSE CAST(SUBSTR(blck_last2, 1, 1) AS INTEGER) END * 10)
             + CAST(SUBSTR(blck_last2, 2, 1) AS INTEGER)) * 10 + t."SAMP_RLTY_PSTN_VAL" AS actual_pos
        FROM (
            SELECT a.*, SUBSTR(a."BLCK_LOT_ID", -2) AS blck_last2,
            ROW_NUMBER() OVER (PARTITION BY a."SAMP_ID" ORDER BY a."CRT_DT" DESC NULLS LAST) AS rn
            FROM oracle.oggmgr.TN_SAMP_MATCH a
            WHERE a."SAMP_USG_CD" = 'RESN' AND ({like_conditions.replace('b."SAMP_ID"', 'a."INGT_LOT_ID"')})
              AND a."BLCK_LOT_ID" IS NOT NULL AND LENGTH(a."BLCK_LOT_ID") >= 2
        ) t WHERE t.rn = 1
    )
    SELECT 
        p."SAMP_ID", SUBSTR(p."SAMP_ID", 1, 5) AS Iot_number, p."BLCK_LOT_ID",
        p."ORDER_PSTN_VAL", p."SAMP_RLTY_PSTN_VAL", p.block_pos,
        CASE WHEN p."SAMP_RLTY_PSTN_VAL" = 999 THEN p.order_pos ELSE p.actual_pos END AS actual_pos,
        p.order_pos, f.FTIR_OI_CENTER, f.FTIR_OI_EDGE1_1, f.FTIR_OI_EDGE1_2, f.FTIR_OI_EDGE1_3, f.FTIR_OI_EDGE1_4,
        f.CUH_BP_CENTER, f.CUH_BP_EDGE, f.CUH_BP_ALL, f.CUH_BSW_CENTER, f.CUH_BSW_EDGE, f.CUH_BSW_ALL,
        fl.FPD_MAX1_VALUE AS FPD_SCORE, fl.LDP_sum AS LDP_SCORE, fl.B_BAND_VALUE,
        CASE WHEN fl.FPD_MAX1_VALUE = 0 THEN 'A' WHEN fl.FPD_MAX1_VALUE IS NULL THEN NULL ELSE 'R' END AS FPD_AR,
        CASE WHEN fl.LDP_sum = 0 THEN 'A' WHEN fl.LDP_sum IS NULL THEN NULL ELSE 'R' END AS LDP_AR,
        CASE WHEN fl.B_BAND_VALUE IS NOT NULL THEN 'Y' ELSE 'N' END AS has_b_band
    FROM pos_data p
    LEFT JOIN ftir_data f ON p."SAMP_ID" = f."SAMP_ID"
    LEFT JOIN fpd_ldp_data fl ON p."SAMP_ID" = fl."SAMP_ID"
    ORDER BY p."SAMP_ID"
    """
    print(f"🔍 [2 단계] 3340_data 정밀 추출 중... 대상 lot 수: {len(lot_prefix_list)}")
    
    try:
        df_3340 = fetch_data_to_df(query)
        print(f"✅ 3340_data 정밀 로드 완료 | 행: {len(df_3340):,}")
        return df_3340
    except Exception as e:
        print(f"❌ 3340_data 추출 실패: {e}")
        raise

# ==============================================================================
# 3 단계: 조인 및 저장
# ==============================================================================
def join_and_save(fdc_df, data_3340_df, output_path, filename_prefix):
    if fdc_df.empty:
        print("⚠️ 조인할 FDC 데이터가 없어 작업을 종료합니다.")
        return

    print("🔍 [3 단계] 조인 시작: dw_fa_eq_fdcxygr_i 기준 LEFT JOIN")
    
    data_3340_df.columns = [col.lower() for col in data_3340_df.columns]
    
    data_3340_df['iot_number'] = data_3340_df['iot_number'].astype(str)
    data_3340_df['order_pos'] = pd.to_numeric(data_3340_df['order_pos'], errors='coerce')
    
    fdc_df['lot_number'] = fdc_df['lot_number'].astype(str)
    fdc_df['length'] = pd.to_numeric(fdc_df['length'], errors='coerce')
    
    merged = fdc_df.merge(
        data_3340_df,
        left_on=['lot_number', 'length'],
        right_on=['iot_number', 'order_pos'],
        how='left',
        suffixes=('', '_3340')
    )
    
    print(f"✅ 조인 완료 | 최종 행 수: {len(merged):,}")
    
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    final_filename = f"{filename_prefix}_{timestamp}.csv"
    final_path = output_path / final_filename
    
    merged.to_csv(final_path, index=False, encoding='utf-8-sig')
    print(f"💾 데이터가 '{final_path}'로 저장되었습니다.")

# ==============================================================================
# 메인 실행 흐름
# ==============================================================================
if __name__ == "__main__":
    print("="*60)
    print("SK Siltron Lot-Based Data Extraction Tool (SONI)")
    print("="*60)
    
    # 1. 코드 내 입력된 Lot 목록 확인
    if not TARGET_LOT_NUMBERS:
        print("\n❌ 오류: TARGET_LOT_NUMBERS 리스트가 비어 있습니다.")
        print("   코드 상단의 [사용자 입력 구역] 에서 Lot 번호를 입력한 후 다시 실행해주세요.")
        print("   예: TARGET_LOT_NUMBERS = ['24A001', '24A002']")
        sys.exit(1)
        
    print(f"\n✅ 입력된 Lot 수: {len(TARGET_LOT_NUMBERS)}개")
    print(f"   샘플: {TARGET_LOT_NUMBERS[:5]}{'...' if len(TARGET_LOT_NUMBERS) > 5 else ''}")

    # 2. 설정 경로
    output_dir = Path('C:/Users/257103/Downloads') # 필요 시 수정
    output_dir.mkdir(exist_ok=True)
    file_prefix = 'fdc_3340_lot_basis'

    try:
        # Step 1: FDC 데이터 추출 (Lot 기준)
        fdc_df, extracted_lots = extract_fdc_data_by_lots(TARGET_LOT_NUMBERS)
        
        if fdc_df.empty:
            print("🛑 추출된 FDC 데이터가 없어 종료합니다.")
            sys.exit(0)

        # Step 2: 3340 데이터 추출
        data_3340 = extract_3340_data_precise(extracted_lots)
        
        # Step 3: 조인 및 저장
        join_and_save(fdc_df, data_3340, output_dir, file_prefix)
        
        print("\n🎉 모든 작업이 성공적으로 완료되었습니다.")
        
    except Exception as e:
        print(f"\n❌ 치명적 오류 발생: {e}")
        import traceback
        traceback.print_exc()
        sys.exit(1)
