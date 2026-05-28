(venv_new) PS D:\python\CUH\전달용> python .\scripts\release\run_apc.py --lot 6H823
D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
  warnings.warn(
📋 release_params: 10 positions, target=80.0

============================================================
📡 Lot 기반 데이터 수집: target=6H823, n_runs=5
============================================================
 [PRES] 6H823 FDC+3340 조회...
 → 6,697 행
Traceback (most recent call last):
  File "D:\python\CUH\전달용\scripts\release\run_apc.py", line 631, in <module>
    main()
  File "D:\python\CUH\전달용\scripts\release\run_apc.py", line 168, in main
    _run_lot_mode(args, paths, pcfg, model_cfg,
  File "D:\python\CUH\전달용\scripts\release\run_apc.py", line 240, in _run_lot_mode
    collected = collect_lot_runs(
                ^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\전달용\src\release\lot_fetch.py", line 155, in collect_lot_runs
    candidates = fetch_prev_lot_candidates(target_lot, prefix_len=prefix_len)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\전달용\src\release\lot_fetch.py", line 67, in fetch_prev_lot_candidates
    df_prev = fetch_df(q_prev)
              ^^^^^^^^^^^^^^^^
  File "D:\python\CUH\전달용\scripts\fetch\fetch_from_db.py", line 101, in fetch_df
    cur.execute(query)
  File "D:\python\virtualenv\venv_new\Lib\site-packages\trino\dbapi.py", line 640, in execute
    self._iterator = iter(self._query.execute())
                          ^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\trino\client.py", line 909, in execute
    self._result.rows += self.fetch()
                         ^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\trino\client.py", line 929, in fetch
    status = self._request.process(response)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\trino\client.py", line 698, in process
    raise self._process_error(response["error"], response.get("id"))
trino.exceptions.TrinoUserError: TrinoUserError(type=USER_ERROR, name=TYPE_MISMATCH, message="line 5:18: Cannot apply operator: decimal(20,0) < varchar(16)", query_id=20260528_090531_20764_tugj6)
