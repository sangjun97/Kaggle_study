jun@PF3SL25728402:/mnt/d/python/CUH/growing_apc$ /mnt/d/python/virtualenv/venv_new/Scripts/python.exe -c "import pstats; pstats.Stats('prof.out').sort_stats('cumtime').print_stats('src|scripts',30)"
Mon Jul 27 17:07:19 2026    prof.out

         22193481 function calls (22061502 primitive calls) in 36.711 seconds

   Ordered by: cumulative time
   List reduced from 8130 to 129 due to restriction <'src|scripts'>
   List reduced from 129 to 30 due to restriction <30>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001   36.715   36.715 scripts/release/validate_cuh.py:1(<module>)
        1    0.127    0.127   33.621   33.621 scripts/release/validate_cuh.py:348(main)
        2    0.019    0.010   33.406   16.703 scripts/release/validate_cuh.py:143(_predict_one_lot)
        2    0.174    0.087   22.728   11.364 D:\python\CUH\growing_apc\src\release\lot_context.py:106(collect_lot_context)
       14    0.001    0.000   20.413    1.458 D:\python\CUH\growing_apc\src\fetch\lot_query.py:318(fetch_lot_merged)
       28    0.038    0.001   20.235    0.723 D:\python\CUH\growing_apc\src\fetch\lot_query.py:76(fetch_df)
       14    0.055    0.004   13.730    0.981 D:\python\CUH\growing_apc\src\fetch\lot_query.py:163(extract_fdc_by_lot)
        2    0.203    0.102    8.260    4.130 D:\python\CUH\growing_apc\src\preprocess\pipeline.py:78(build_grower_from_runs)
       10    0.197    0.020    7.573    0.757 D:\python\CUH\growing_apc\src\preprocess\pipeline.py:41(process_ingot_for_grower)
       12    0.142    0.012    6.764    0.564 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:7(process_avg)
       12    0.000    0.000    6.124    0.510 D:\python\CUH\growing_apc\src\fetch\lot_query.py:179(extract_3340)
        1    0.000    0.000    1.763    1.763 D:\python\CUH\growing_apc\src\core\metrics.py:1(<module>)
        2    0.055    0.027    1.762    0.881 D:\python\CUH\growing_apc\src\release\pipeline.py:20(preprocess_ingot)
       12    0.000    0.000    1.403    0.117 D:\python\CUH\growing_apc\src\release\lot_context.py:57(_first_row_usage)
       12    0.002    0.000    1.400    0.117 D:\python\CUH\growing_apc\src\release\lot_context.py:59(<dictcomp>)
       32    0.107    0.003    1.122    0.035 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:74(process_avg_nan)
       10    0.000    0.000    1.109    0.111 D:\python\CUH\growing_apc\src\release\lot_context.py:81(usage_pair_ok)
       12    0.001    0.000    0.557    0.046 D:\python\CUH\growing_apc\src\fetch\lot_query.py:300(join_fdc_3340)
       26    0.002    0.000    0.440    0.017 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:130(process_set)
        2    0.000    0.000    0.438    0.219 D:\python\CUH\growing_apc\src\fetch\lot_query.py:113(get_lot_sequence)
        2    0.001    0.001    0.360    0.180 D:\python\CUH\growing_apc\src\release\pipeline.py:73(find_valid_prevs)
       12    0.006    0.000    0.354    0.029 D:\python\CUH\growing_apc\src\preprocess\matching.py:56(generate_reference_position)
        1    0.000    0.000    0.266    0.266 D:\python\CUH\growing_apc\src\release\lot_context.py:1(<module>)
        1    0.000    0.000    0.262    0.262 D:\python\CUH\growing_apc\src\fetch\lot_query.py:1(<module>)
        8    0.012    0.001    0.181    0.023 D:\python\CUH\growing_apc\src\release\pipeline.py:157(build_delta_table)
       20    0.001    0.000    0.169    0.008 D:\python\CUH\growing_apc\src\preprocess\pipeline.py:15(_apply_00_filter)
       10    0.034    0.003    0.160    0.016 D:\python\CUH\growing_apc\src\preprocess\validation.py:6(filter_valid_data)
       10    0.001    0.000    0.133    0.013 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:116(process_avg_const)
       10    0.002    0.000    0.117    0.012 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:138(process_oi)
        8    0.000    0.000    0.114    0.014 D:\python\CUH\growing_apc\src\core\config.py:40(load_yaml)
