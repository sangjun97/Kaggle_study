사용자 SQL 그대로 실행
입력 lot: ['6B523']
----------------------------------------------------------------------

행수: 26

fpd_ar 분포: {'A': np.int64(26)}
cuh_bsw_center 유효: 23
ftir_oi_center 유효: 23

처음 15행:
   samp_id iot_number order_pos cuh_bsw_center ftir_oi_center fpd_ar fpd_score
6B52302001      6B523        20           None         13.094      A     0E-10
6B52302067      6B523        90           None         13.101      A     0E-10
6B52302127      6B523       150             45          11.06      A     0E-10
6B52302127      6B523       150             45          11.06      A     0E-10
6B52302129      6B523       152             45           None      A     0E-10
6B52302146      6B523       170             45           None      A     0E-10
6B52302183      6B523       207             45           None      A     0E-10
6B52302245      6B523       270             30          10.83      A     0E-10
6B52302357      6B523       390             70         10.845      A     0E-10
6B52339000      6B523       390             70         10.845      A     0E-10
6B52339176      6B523       570             80         11.167      A     0E-10
6B52339353      6B523       760             80         11.236      A     0E-10
6B52376001      6B523       760             95         11.095      A     0E-10
6B52376176      6B523       940            100         11.265      A     0E-10
6B52376999      6B523      1120             85         11.031      A     0E-10

사용자 SQL 그대로 실행                                                                                                                                                                                           
입력 lot: ['6B520', '6B521', '6B522', '6B523']                                                                                                                                                                   
----------------------------------------------------------------------                                                                                                                                           
                                                                                                                                                                                                                 
행수: 106                                                                                                                                                                                                        
                                                                                                                                                                                                                 
fpd_ar 분포: {'A': np.int64(96), None: np.int64(10)}
cuh_bsw_center 유효: 86
ftir_oi_center 유효: 96

처음 15행:
   samp_id iot_number order_pos cuh_bsw_center ftir_oi_center fpd_ar fpd_score
6B52002001      6B520        20           None         16.424      A     0E-10
6B52002067      6B520        90           None         12.968      A     0E-10
6B52002127      6B520       150           None         11.112      A     0E-10
6B52002128      6B520       151           None         10.991   None      None
6B52002245      6B520       270           None         11.016   None      None
6B52002355      6B520       390           None         11.168   None      None
6B52039001      6B520       390             80         11.189      A     0E-10
6B52039176      6B520       570             80         11.063      A     0E-10
6B52039999      6B520       760             85         11.175      A     0E-10
6B52076001      6B520       760             85         11.175      A     0E-10
6B52076176      6B520       940             70         11.299      A     0E-10
6B52076999      6B520      1120            100         11.022      A     0E-10
6B520B2001      6B520      1120            100         11.022      A     0E-10
6B520B2176      6B520      1300             70         11.007      A     0E-10
6B520B2346      6B520      1480             55         10.824      A     0E-10

>> import pandas as pd
>> df = pd.read_csv('D:/python/CUH/전달용/fdc_3340_lot_basis_20260529_105443.csv', encoding='utf-8-sig', low_memory=False)
>> df.columns = df.columns.str.lower()
>> # fpd_ar 가 채워진 행 (6B520)
>> sub = df[(df['lot_number']=='6B520') & df['fpd_ar'].notna()]
>> print(f'행수: {len(sub)}')
>> print()
>> print('컬럼 목록:', list(df.columns))
>> print()
>> print('첫 10행 (주요 컬럼):')
>> cols = ['lot_number', 'length', 'samp_id', 'iot_number', 'order_pos', 'fpd_ar', 'fpd_score', 'cuh_bsw_center']
>> cols = [c for c in cols if c in sub.columns]
>> print(sub[cols].head(10).to_string())
>> "
행수: 55

컬럼 목록: ['gr_out', 'area', 'grower', 'lot_number', 'runtime_event', 'runtime_all', 'work_time', 'mscode', 'customer_grp_1', 'customer_grp_2', 'cust_nm', 'current_event', 'attempt', 'max_attempt', 'ingot_status', 'st_loss_pos', 'st_loss_label', 'length', 'dia', 'adc_filter', 'adc_position_nor', 'ssrp_sho_dia', 'ssrp_sho_dia_error', 'a_chamber_flow', 'b_chamber_flow', 'a_chamber_temp', 'b_chamber_temp', 'water_jacket_1_flow', 'water_jacket_2_flow', 'water_jacket_1_temp', 'water_jacket_2_temp', 'ar_flow', 'ar_gas_no2', 'a_chamber_pressure', 'throttle_valve_open', 'actual_pull_speed', 'actual_pull_speed_gap', 'actual_clr', 'crucible_lift', 'crucible_position', 'crucible_rotation', 'seed_position', 'seed_rotation', 'melt_gap', 'melt_gap_delta', 'melt_temp', 'atc_actual_value', 'body_profile', 'body_profile_gap', 'heater_power', 'heater_resistance', 'st_heater_power', 'actual_magnet_intensity', 'all_cleaning', 'exhaust_line_cleaning', 'crucible_2d', 'heater_cap', 'nop_inner', 'nop_outer_lower', 'nop_outer_upper', 'nop_shield', 'nop_tube', 'side_heater', 'st_heater', 'top_ring', 'uls_heat_cap', 'upper_ring', 'upper_tube_l', 'upper_tube_u', 'frs_oi', 'frs_eqp_id', 'pull_speed', 'magnet_position', 'ar_gas_no3', 'hsys_upd_dtm', 'pt_d', 'renamed_grower', 'renamed_lot_number', 'samp_id', 'iot_number', 'blck_lot_id', 'order_pstn_val', 'samp_rlty_pstn_val', 'block_pos', 'actual_pos', 'order_pos', 'ftir_oi_center', 'ftir_oi_edge1_1', 'ftir_oi_edge1_2', 'ftir_oi_edge1_3', 'ftir_oi_edge1_4', 'cuh_bp_center', 'cuh_bp_edge', 'cuh_bp_all', 'cuh_bsw_center', 'cuh_bsw_edge', 'cuh_bsw_all', 'fpd_score', 'ldp_score', 'b_band_value', 'fpd_ar', 'ldp_ar', 'has_b_band']

첫 10행 (주요 컬럼):
      lot_number  length     samp_id iot_number  order_pos fpd_ar  fpd_score  cuh_bsw_center
12443      6B520    20.0  6B52002001      6B520       20.0      A        0.0             NaN
12524      6B520    90.0  6B52002067      6B520       90.0      A        0.0             NaN
12525      6B520    90.0  6B52002067      6B520       90.0      A        0.0             NaN
12591      6B520   150.0  6B52002127      6B520      150.0      A        0.0             NaN
12592      6B520   150.0  6B52002127      6B520      150.0      A        0.0             NaN
12621      6B520    20.0  6B52002001      6B520       20.0      A        0.0             NaN
12622      6B520    20.0  6B52002001      6B520       20.0      A        0.0             NaN
12739      6B520    90.0  6B52002067      6B520       90.0      A        0.0             NaN
12858      6B520   150.0  6B52002127      6B520      150.0      A        0.0             NaN
12859      6B520   150.0  6B52002127      6B520      150.0      A        0.0             NaN
