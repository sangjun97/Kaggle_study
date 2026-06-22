(dsstar) PS D:\python\기타과제\20260622_DS_STAR> uv sync --active --native-tls
warning: The `tool.uv.dev-dependencies` field (used in `pyproject.toml`) is deprecated and will be removed in a future release; use `dependency-groups.dev` instead
warning: The `--native-tls` flag is deprecated and will be removed in a future release. Use `--system-certs` instead.
warning: The `UV_NATIVE_TLS` environment variable is deprecated and will be removed in a future release. Use `UV_SYSTEM_CERTS` instead.
error: Querying Python at `D:\python\기타과제\20260622_DS_STAR\dsstar\Scripts\python.exe` failed with exit status exit code: 1

[stderr]
Fatal Python error: init_import_site: Failed to import the site module
Python runtime state: initialized
Traceback (most recent call last):
  File "<frozen importlib._bootstrap>", line 1176, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1147, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 690, in _load_unlocked
  File "<frozen importlib._bootstrap>", line 980, in exec_module
  File "<frozen site>", line 626, in <module>
  File "<frozen site>", line 609, in main
  File "<frozen site>", line 541, in venv
  File "<frozen site>", line 394, in addsitepackages
  File "<frozen site>", line 236, in addsitedir
  File "<frozen site>", line 188, in addpackage
UnicodeDecodeError: 'cp949' codec can't decode byte 0x83 in position 14: illegal multibyte sequence
