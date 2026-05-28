(venv_new) PS D:\python\CUH\전달용> python .\scripts\preprocess\02_grower_data.py --incremental   
📋 incremental: 처리 대상 잉곳 22 개 (22 grower)
잉곳 처리: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 22/22 [01:01<00:00,  2.78s/it]

📦 grower.csv 갱신 시작: 0 개 grower

============================================================
📊 처리 요약 (incremental 모드)
============================================================
 잉곳 처리/갱신: 0 개
 empty (신규): 6 개
 no summary row: 0 개 ← 01 미반영 의심
 customer 필터 탈락: 16 개
 탈락 customer_grp_2 분포: {'PW_400': 6, 'Multi_Kioxia Double': 1, 'Smart_SEC F3 440': 2, 'Zenith 1.0_SK hynix/GF/CXMT 440': 2, 'Zenith 1.0_Multi_Kioxia Double': 1, 'EPI_440 (일반)': 2, 'Smart_SEC FV3/FV4/Kioxia 440': 2}
 raw 파일 누락: 0 개
 예외 발생: 0 개
 empty (캐시 skip): 0 개
 empty 누적: 506 개
 grower.csv 갱신: 0 개
 grower.csv 유지: 155 개
📝 완료 마커: data\grower\_DONE_02_grower_data.txt
🎉 전체 처리 완료
