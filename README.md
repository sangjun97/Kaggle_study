python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:97 문자:9
+         python scripts\release\eval_validation_mae.py --in $valout *> ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\release\eval_validation_mae.py", line 151, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\release\eval_validation_mae.py", line 48, in main
    df = pd.read_csv(args.inp)
         ^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1026, in read_csv
    return _read(filepath_or_buffer, kwds)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 620, in _read
    parser = TextFileReader(filepath_or_buffer, **kwds)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1620, in __init__
    self._engine = self._make_engine(f, self.engine)
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1880, in _make_engine
    self.handles = get_handle(
                   ^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\common.py", line 873, in get_handle
    handle = open(
             ^^^^^
FileNotFoundError: [Errno 2] No such file or directory: 'outputs\\retrain_compare\\val_B_baseline.csv'

=== [B_baseline] lr=0.007 pw=7.0 model=config\model.yaml ===
--- 01_train ---
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:72 문자:9
+         python scripts\model\01_train.py `
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\model\01_train.py", line 382, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\model\01_train.py", line 162, in main
    print(f"\U0001f4e6 file_version = {args.file_version}")
UnicodeEncodeError: 'cp949' codec can't encode character '\U0001f4e6' in position 0: illegal multibyte sequence
--- refine ---
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:80 문자:9
+         python scripts\model\02_refine_primary.py --set-name $name -- ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\model\02_refine_primary.py", line 105, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\model\02_refine_primary.py", line 41, in main
    files = find_model_files(train_dir, args.set_name, experiments)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\growing_apc_LOW\src\model\ebm_extract.py", line 22, in find_model_files
    print(f"\u26a0\ufe0f  파일 없음: {path}")
UnicodeEncodeError: 'cp949' codec can't encode character '\u26a0' in position 0: illegal multibyte sequence
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:81 문자:9
+         python scripts\model\02_refine_primary.py --set-name $name -- ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\model\02_refine_primary.py", line 105, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\model\02_refine_primary.py", line 38, in main
    n_exp = len(model_cfg.experiments[args.gen])
                ~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^
KeyError: 'GEN4'
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:82 문자:9
+         python scripts\model\03_refine_base.py    --set-name $name -- ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\model\03_refine_base.py", line 74, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\model\03_refine_base.py", line 35, in main
    files = find_model_files(train_dir, args.set_name, experiments)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\CUH\growing_apc_LOW\src\model\ebm_extract.py", line 22, in find_model_files
    print(f"\u26a0\ufe0f  파일 없음: {path}")
UnicodeEncodeError: 'cp949' codec can't encode character '\u26a0' in position 0: illegal multibyte sequence
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:83 문자:9
+         python scripts\model\03_refine_base.py    --set-name $name -- ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\model\03_refine_base.py", line 74, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\model\03_refine_base.py", line 32, in main
    n_exp       = len(model_cfg.experiments[args.gen])
                      ~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^
KeyError: 'GEN4'
--- build_models_json ---
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:86 문자:9
+         python scripts\model\04_build_models_json.py --set-name $name ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\model\04_build_models_json.py", line 34, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\model\04_build_models_json.py", line 29, in main
    build_final_slopes(args.source_dir, args.output)
  File "D:\python\CUH\growing_apc_LOW\src\model\slope_io.py", line 107, in build_final_slopes
    print(f"\u2705 최종 slopes.json 생성: {output_file}")
UnicodeEncodeError: 'cp949' codec can't encode character '\u2705' in position 0: illegal multibyte sequence
--- validate_cuh ---
python : D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (
2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:90 문자:9
+         python scripts\release\validate_cuh.py `
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (D:\python\virtu...ported version!:String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  warnings.warn(
models_json → config/models_B_baseline.json
meta.csv: 16 lot 처리 시작
 [1/16] 6F108: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [2/16] 6F002: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [3/16] 6W906: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [4/16] 6W911: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [5/16] 6A201: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [6/16] 6A202: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [7/16] 6A204: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [8/16] 6A206: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [9/16] 6A210: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [10/16] 6B307: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [11/16] 6B308: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [12/16] 6C404: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [13/16] 6C405: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [14/16] 6C407: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [15/16] 6D202: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
 [16/16] 6D205: 오류 'cp949' codec can't encode character '\U0001f3af' in position 0: illegal multibyte sequence
Traceback (most recent call last):
  File "D:\python\CUH\growing_apc_LOW\scripts\release\validate_cuh.py", line 314, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\release\validate_cuh.py", line 305, in main
    print("\u274c 저장할 행 없음")
UnicodeEncodeError: 'cp949' codec can't encode character '\u274c' in position 0: illegal multibyte sequence
python : Traceback (most recent call last):
위치 D:\python\CUH\growing_apc_LOW\compare_retrain.ps1:97 문자:9
+         python scripts\release\eval_validation_mae.py --in $valout *> ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (Traceback (most recent call last)::String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
 
  File "D:\python\CUH\growing_apc_LOW\scripts\release\eval_validation_mae.py", line 151, in <module>
    main()
  File "D:\python\CUH\growing_apc_LOW\scripts\release\eval_validation_mae.py", line 48, in main
    df = pd.read_csv(args.inp)
         ^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1026, in read_csv
    return _read(filepath_or_buffer, kwds)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 620, in _read
    parser = TextFileReader(filepath_or_buffer, **kwds)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1620, in __init__
    self._engine = self._make_engine(f, self.engine)
                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\parsers\readers.py", line 1880, in _make_engine
    self.handles = get_handle(
                   ^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\io\common.py", line 873, in get_handle
    handle = open(
             ^^^^^
FileNotFoundError: [Errno 2] No such file or directory: 'outputs\\retrain_compare\\val_B_baseline.csv'
