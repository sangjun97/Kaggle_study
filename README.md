jun@PF3SL25728402:/mnt/d/python/CUH/growing_apc$ /mnt/d/python/virtualenv/venv_new/Scripts/python.exe -c "import pstats; pstats.Stats('prof.out').sort_stats('cumtime').print_stats('src|scripts',30)"
Mon Jul 27 16:52:11 2026    prof.out

         1083194299 function calls (1071725308 primitive calls) in 772.934 seconds

   Ordered by: cumulative time
   List reduced from 8116 to 131 due to restriction <'src|scripts'>
   List reduced from 131 to 30 due to restriction <30>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001  772.938  772.938 scripts/release/validate_cuh.py:1(<module>)
        1    0.127    0.127  768.623  768.623 scripts/release/validate_cuh.py:348(main)
        2    0.018    0.009  768.403  384.202 scripts/release/validate_cuh.py:143(_predict_one_lot)
       12    0.224    0.019  718.099   59.842 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:11(process_avg)
  2195310   11.453    0.000  624.272    0.000 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:61(<lambda>)
        2    0.206    0.103  616.144  308.072 D:\python\CUH\growing_apc\src\preprocess\pipeline.py:78(build_grower_from_runs)
       10    0.203    0.020  615.465   61.546 D:\python\CUH\growing_apc\src\preprocess\pipeline.py:41(process_ingot_for_grower)
        2    0.057    0.028  128.143   64.071 D:\python\CUH\growing_apc\src\release\pipeline.py:20(preprocess_ingot)
       32    0.107    0.003   24.053    0.752 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:67(process_avg_nan)
        2    0.179    0.090   23.462   11.731 D:\python\CUH\growing_apc\src\release\lot_context.py:106(collect_lot_context)
       14    0.000    0.000   20.814    1.487 D:\python\CUH\growing_apc\src\fetch\lot_query.py:318(fetch_lot_merged)
       28    0.037    0.001   20.751    0.741 D:\python\CUH\growing_apc\src\fetch\lot_query.py:76(fetch_df)
    70778    0.363    0.000   20.125    0.000 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:105(<lambda>)
       14    0.048    0.003   13.942    0.996 D:\python\CUH\growing_apc\src\fetch\lot_query.py:163(extract_fdc_by_lot)
       12    0.000    0.000    6.336    0.528 D:\python\CUH\growing_apc\src\fetch\lot_query.py:179(extract_3340)
        1    0.000    0.000    2.492    2.492 D:\python\CUH\growing_apc\src\core\metrics.py:1(<module>)
       12    0.000    0.000    1.627    0.136 D:\python\CUH\growing_apc\src\release\lot_context.py:57(_first_row_usage)
       12    0.002    0.000    1.624    0.135 D:\python\CUH\growing_apc\src\release\lot_context.py:59(<dictcomp>)
       10    0.000    0.000    1.229    0.123 D:\python\CUH\growing_apc\src\release\lot_context.py:81(usage_pair_ok)
       12    0.001    0.000    0.535    0.045 D:\python\CUH\growing_apc\src\fetch\lot_query.py:300(join_fdc_3340)
        2    0.000    0.000    0.524    0.262 D:\python\CUH\growing_apc\src\fetch\lot_query.py:113(get_lot_sequence)
       26    0.002    0.000    0.451    0.017 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:125(process_set)
        1    0.000    0.000    0.399    0.399 D:\python\CUH\growing_apc\src\release\lot_context.py:1(<module>)
        1    0.000    0.000    0.391    0.391 D:\python\CUH\growing_apc\src\fetch\lot_query.py:1(<module>)
        2    0.001    0.001    0.360    0.180 D:\python\CUH\growing_apc\src\release\pipeline.py:73(find_valid_prevs)
       12    0.006    0.000    0.345    0.029 D:\python\CUH\growing_apc\src\preprocess\matching.py:56(generate_reference_position)
        8    0.012    0.001    0.181    0.023 D:\python\CUH\growing_apc\src\release\pipeline.py:157(build_delta_table)
       20    0.001    0.000    0.172    0.009 D:\python\CUH\growing_apc\src\preprocess\pipeline.py:15(_apply_00_filter)
       10    0.036    0.004    0.157    0.016 D:\python\CUH\growing_apc\src\preprocess\validation.py:6(filter_valid_data)
       10    0.001    0.000    0.130    0.013 D:\python\CUH\growing_apc\src\preprocess\smoothing.py:111(process_avg_const)
