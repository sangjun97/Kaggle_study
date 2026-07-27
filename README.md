jun@PF3SL25728402:/mnt/d/python/CUH/growing_apc$ /mnt/d/python/virtualenv/venv_new/Scripts/python.exe -m cProfile -s cumtime scripts/release/validate_cuh.py --meta config/validation_meta_tmp.csv --set-name new_data_t200 --out outputs/val_ne
w_data_t200.csv 2>&1 | head -40
D:\python\virtualenv\venv_new\Lib\site-packages\requests\__init__.py:113: RequestsDependencyWarning: urllib3 (2.6.3) or chardet (7.4.3)/charset_normalizer (2.1.1) doesn't match a supported version!
  warnings.warn(
         1869576 function calls (1828387 primitive calls) in 4.009 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
      179    0.004    0.000   12.380    0.069 __init__.py:1(<module>)
   1612/1    0.026    0.000    4.011    4.011 {built-in method builtins.exec}
        1    0.000    0.000    4.011    4.011 validate_cuh.py:1(<module>)
  1570/10    0.008    0.000    3.969    0.397 <frozen importlib._bootstrap>:1165(_find_and_load)
  1540/10    0.006    0.000    3.969    0.397 <frozen importlib._bootstrap>:1120(_find_and_load_unlocked)
  1498/13    0.005    0.000    3.946    0.304 <frozen importlib._bootstrap>:666(_load_unlocked)
  1264/13    0.003    0.000    3.946    0.304 <frozen importlib._bootstrap_external>:934(exec_module)
  3603/22    0.002    0.000    3.941    0.179 <frozen importlib._bootstrap>:233(_call_with_frames_removed)
   706/64    0.001    0.000    3.218    0.050 {built-in method builtins.__import__}
        1    0.000    0.000    2.262    2.262 metrics.py:1(<module>)
       11    0.000    0.000    2.164    0.197 base.py:1(<module>)
 1602/820    0.002    0.000    2.131    0.003 <frozen importlib._bootstrap>:1207(_handle_fromlist)
        1    0.000    0.000    2.077    2.077 _chunking.py:1(<module>)
        1    0.000    0.000    2.075    2.075 _param_validation.py:1(<module>)
     5585    1.776    0.000    1.776    0.000 {built-in method nt.stat}
     5565    0.003    0.000    1.772    0.000 <frozen importlib._bootstrap_external>:140(_path_stat)
        2    0.000    0.000    1.694    0.847 _array_api.py:1(<module>)
     1524    0.010    0.000    1.540    0.001 <frozen importlib._bootstrap>:1054(_find_spec)
     1501    0.001    0.000    1.519    0.001 <frozen importlib._bootstrap_external>:1499(find_spec)
     1501    0.004    0.000    1.518    0.001 <frozen importlib._bootstrap_external>:1467(_get_spec)
        2    0.000    0.000    1.472    0.736 validation.py:1(<module>)
     2127    0.018    0.000    1.403    0.001 <frozen importlib._bootstrap_external>:1607(find_spec)
     1264    0.010    0.000    1.088    0.001 <frozen importlib._bootstrap_external>:1007(get_code)
        1    0.000    0.000    0.921    0.921 fixes.py:1(<module>)
       10    0.000    0.000    0.769    0.077 api.py:1(<module>)
17790/16644    0.004    0.000    0.659    0.000 {built-in method builtins.hasattr}
        1    0.000    0.000    0.642    0.642 _stats_py.py:1(<module>)
     2008    0.002    0.000    0.593    0.000 <frozen importlib._bootstrap_external>:150(_path_is_mode_type)
     1840    0.001    0.000    0.544    0.000 <frozen importlib._bootstrap_external>:159(_path_isfile)
     1264    0.005    0.000    0.535    0.000 <frozen importlib._bootstrap_external>:1127(get_data)
1498/1463    0.002    0.000    0.522    0.000 <frozen importlib._bootstrap>:566(module_from_spec)
        6    0.000    0.000    0.499    0.083 _base.py:1(<module>)
       11    0.000    0.000    0.487    0.044 __init__.py:720(__getattr__)
