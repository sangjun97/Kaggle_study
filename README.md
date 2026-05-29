(venv_new) PS D:\python\CUH\전달용> python .\scripts\diag_fpd.py      
D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
  warnings.warn(
         cnt
0  173619164
Traceback (most recent call last):
  File "D:\python\CUH\전달용\scripts\diag_fpd.py", line 9, in <module>
    print(fetch_df("""
          ^^^^^^^^^^^^
  File "D:\python\CUH\전달용\scripts/fetch\fetch_from_db.py", line 93, in fetch_df
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
trino.exceptions.TrinoUserError: TrinoUserError(type=USER_ERROR, name=COLUMN_NOT_FOUND, message="line 3:11: Column 'rownum' cannot be resolved", query_id=20260529_020110_05566_tugj6)
