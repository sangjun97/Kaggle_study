(venv_new) PS D:\python\CUH\growing_apc> .\run_ab_experiments.ps1

===== 踰좎씠?ㅻ씪??(model_v1) ?대? ?숈뒿?섏뼱 ?덉뼱????=====

===== [E1] HEAD weight = 2.0 =====
📦 file_version = v1
 x_columns = ['pull_speed_t100_delta', 'ftir_oi_center_delta', 'dia_l50_delta']
 scale_values = [0.01, 1.0, 5]
📦 ebm_cfg = lr=0.03, n_est=300, bins=64, outer_bags=4
INFO:__main__: 🎯 GEN4_1: head_weight=2.0 → 3131/3131 행 (ref_length<600)
 📐 scaler=robust, scales=[0.00348, 0.2735, 0.86799]
    [Primary Weight] Stage1 : 291/3131
INFO:interpret.utils._native:EBM lib loading.
INFO:interpret.utils._native:Finding library for Windows, AMD64, bitsize=64, debug=False
INFO:interpret.utils._native:Loading EBM library D:\python\virtualenv\venv_new\Lib\site-packages\interpret\utils\..\root\bld\lib\libebm_win_x64.dll
INFO:interpret.utils._compressed_dataset:Creating native dataset
INFO:interpret.utils._compressed_dataset:Creating native dataset
INFO:interpret.utils._compressed_dataset:Creating native dataset
INFO:__main__: 💾 번들 저장: outputs\train\model_v1_E1_head2_GEN4_1
INFO:__main__: 🎯 GEN4_2: head_weight=2.0 → 2523/3471 행 (ref_length<600)
 📐 scaler=robust, scales=[0.00327, 0.285, 0.84985]
    [Primary Weight] Stage1 : 319/3471
INFO:interpret.utils._compressed_dataset:Creating native dataset
INFO:interpret.utils._compressed_dataset:Creating native dataset
Exception in thread ExecutorManagerThread:
Traceback (most recent call last):
  File "C:\Users\SKsiltron\AppData\Local\Programs\Python\Python311\Lib\threading.py", line 1045, in _bootstrap_inner
    self.run()
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\externals\loky\process_executor.py", line 626, in run
    self.terminate_broken(bpe)
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\externals\loky\process_executor.py", line 833, in terminate_broken
    self.kill_workers(reason="broken executor")
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\externals\loky\process_executor.py", line 866, in kill_workers
    kill_process_tree(p)
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\externals\loky\backend\utils.py", line 19, in kill_process_tree
    _kill_process_tree_with_psutil(process)
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\externals\loky\backend\utils.py", line 35, in _kill_process_tree_with_psutil
    descendants = psutil.Process(process.pid).children(recursive=True)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\psutil\__init__.py", line 977, in children
    ppid_map = _ppid_map()
               ^^^^^^^^^^^
OSError: [WinError 1455] 이 작업을 완료하기 위한 페이징 파일이 너무 작습니다
joblib.externals.loky.process_executor._RemoteTraceback: 
"""
Traceback (most recent call last):
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\externals\loky\process_executor.py", line 453, in _process_worker
    call_item = call_queue.get(block=True, timeout=timeout)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\SKsiltron\AppData\Local\Programs\Python\Python311\Lib\multiprocessing\queues.py", line 122, in get
    return _ForkingPickler.loads(res)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\interpret\glassbox\__init__.py", line 4, in <module>
    from ._aplr import APLRClassifier, APLRRegressor  # noqa: F401
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\interpret\glassbox\_aplr.py", line 8, in <module>
    import pandas as pd
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\__init__.py", line 61, in <module>
    from pandas.core.api import (
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\core\api.py", line 1, in <module>
    from pandas._libs import (
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\_libs\__init__.py", line 18, in <module>
    from pandas._libs.interval import Interval
  File "pandas/_libs/interval.pyx", line 1, in init pandas._libs.interval
  File "pandas/_libs/hashtable.pyx", line 1, in init pandas._libs.hashtable
  File "pandas/_libs/missing.pyx", line 1, in init pandas._libs.missing
  File "D:\python\virtualenv\venv_new\Lib\site-packages\pandas\_libs\tslibs\__init__.py", line 40, in <module>
    from pandas._libs.tslibs.conversion import localize_pydatetime
  File "pandas/_libs/tslibs/conversion.pyx", line 1, in init pandas._libs.tslibs.conversion
  File "pandas/_libs/tslibs/offsets.pyx", line 1, in init pandas._libs.tslibs.offsets
  File "pandas/_libs/tslibs/timestamps.pyx", line 1, in init pandas._libs.tslibs.timestamps
  File "pandas/_libs/tslibs/timedeltas.pyx", line 1, in init pandas._libs.tslibs.timedeltas
ImportError: DLL load failed while importing timezones: 이 작업을 완료하기 위한 페이징 파일이 너무 작습니다.
"""

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "D:\python\CUH\growing_apc\scripts\model\01_train.py", line 304, in <module>
    main()
  File "D:\python\CUH\growing_apc\scripts\model\01_train.py", line 205, in main
    model_primary = train_ebm_model(
                    ^^^^^^^^^^^^^^^^
  File "D:\python\CUH\growing_apc\src\model\two_stage_ebm.py", line 39, in train_ebm_model
    model.fit(X_train, y_train, sample_weight=sample_weight)
  File "D:\python\virtualenv\venv_new\Lib\site-packages\interpret\glassbox\_ebm\_ebm.py", line 1179, in fit
    results = parallel(
              ^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\parallel.py", line 2072, in __call__
    return output if self.return_generator else list(output)
                                                ^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\parallel.py", line 1682, in _get_outputs
    yield from self._retrieve()
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\parallel.py", line 1784, in _retrieve
    self._raise_error_fast()
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\parallel.py", line 1859, in _raise_error_fast
    error_job.get_result(self.timeout)
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\parallel.py", line 758, in get_result
    return self._return_or_raise()
           ^^^^^^^^^^^^^^^^^^^^^^^
  File "D:\python\virtualenv\venv_new\Lib\site-packages\joblib\parallel.py", line 773, in _return_or_raise
    raise self._result
joblib.externals.loky.process_executor.BrokenProcessPool: A task has failed to un-serialize. Please ensure that the arguments of the function are all picklable.
