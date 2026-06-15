 [77/137] 20250130.csv: 잉곳 +20 (누적 1795)
Traceback (most recent call last):
  File "D:\python\CUH\growing_apc\scripts\diagnostics\diag_distribution.py", line 145, in collect
    df = read_csv(fp, normalize=True)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\growing_apc\src\core\io.py", line 21, in read_csv
    df = pd.read_csv(path, encoding=encoding, **kwargs)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1026, in read_csv
    return _read(filepath_or_buffer, kwds)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 626, in _read
    return parser.read(nrows)
           ^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1923, in read
    ) = self._engine.read(  # type: ignore[attr-defined]
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\c_parser_wrapper.py", line 239, in read
    data = self._reader.read(nrows)
           ^^^^^^^^^^^^^^^^^^^^^^^^
  File "pandas/_libs/parsers.pyx", line 820, in pandas._libs.parsers.TextReader.read
  File "pandas/_libs/parsers.pyx", line 914, in pandas._libs.parsers.TextReader._read_rows
  File "pandas/_libs/parsers.pyx", line 891, in pandas._libs.parsers.TextReader._check_tokenize_status
  File "pandas/_libs/parsers.pyx", line 2061, in pandas._libs.parsers.raise_parser_error
pandas.errors.ParserError: Error tokenizing data. C error: out of memory

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "D:\python\CUH\growing_apc\scripts\diagnostics\diag_distribution.py", line 393, in <module>
    main()
  File "D:\python\CUH\growing_apc\scripts\diagnostics\diag_distribution.py", line 275, in main
    df = collect(raw_dir, pcfg,
         ^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\growing_apc\scripts\diagnostics\diag_distribution.py", line 147, in collect
    df = read_csv(fp, normalize=True,encoding='cp949')
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\growing_apc\src\core\io.py", line 21, in read_csv
    df = pd.read_csv(path, encoding=encoding, **kwargs)
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1026, in read_csv
    return _read(filepath_or_buffer, kwds)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 626, in _read
    return parser.read(nrows)
           ^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1923, in read
    ) = self._engine.read(  # type: ignore[attr-defined]
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\c_parser_wrapper.py", line 239, in read
    data = self._reader.read(nrows)
           ^^^^^^^^^^^^^^^^^^^^^^^^
  File "pandas/_libs/parsers.pyx", line 820, in pandas._libs.parsers.TextReader.read
  File "pandas/_libs/parsers.pyx", line 914, in pandas._libs.parsers.TextReader._read_rows
  File "pandas/_libs/parsers.pyx", line 891, in pandas._libs.parsers.TextReader._check_tokenize_status
  File "pandas/_libs/parsers.pyx", line 2053, in pandas._libs.parsers.raise_parser_error
UnicodeDecodeError: 'cp949' codec can't decode byte 0xeb in position 53423: illegal multibyte sequence
