(venv_new) PS D:\python\CUH\growing_apc_LOW> foreach ($s in $sweep) {
>>     $name=$s.name; $log="$OUT\log_$name.txt"
>>     Get-ChildItem config\*$name* -ErrorAction SilentlyContinue | Remove-Item -Force
>>     WriteUtf8 $log "=== $name lr=$($s.lr) nest=$($s.nest) bags=$($s.bags) ===" $false
>>
>>     RunPy @("scripts\model\01_train.py",
>>         "--data-train","data\dataset\delta_train.csv","--data-test","data\dataset\delta_test.csv",
>>         "--set-name",$name,"--file-version","v1","--model-config","config\model.yaml",
>>         "--lr",$s.lr,"--n-est",$s.nest,"--outer-bags",$s.bags,"--pw-high","7.0") $log
>>     RunPy @("scripts\model\02_refine_primary.py","--set-name",$name,"--gen","GEN5") $log
>>     RunPy @("scripts\model\03_refine_base.py","--set-name",$name,"--gen","GEN5") $log
>>     RunPy @("scripts\model\04_build_models_json.py","--set-name",$name) $log
>>     RunPy @("scripts\release\validate_cuh.py",
>>         "--meta","config\validation_meta_low.csv","--paths-config","config\paths.yaml",
>>         "--preprocess-config","config\preprocess.yaml","--model-config","config\model.yaml",
>>         "--set-name",$name,"--out","$OUT\val_$name.csv") $log
>>
>>     if (Test-Path "$OUT\val_$name.csv") {
>>         $line = & $PY scripts\release\eval_k.py --in "$OUT\val_$name.csv" --name $name 2>&1 |
>>             ForEach-Object { $_.ToString() }
>>         $line | ForEach-Object { Write-Host $_; WriteUtf8 $summary $_ $true }
>>     } else {
>>         $msg = "[$name] FAILED - no val csv"
>>         Write-Host $msg -ForegroundColor Red
>>         WriteUtf8 $summary $msg $true
>>     }
>> }
[K1_lr02] k=0.288 MAE=38.07 Bias=-16.27 nonsat_MAE=30.31 (baseline: k=0.254 MAE=37.17)
[K2_lr05] FAILED - no val csv
[K3_bag1] FAILED - no val csv
(venv_new) PS D:\python\CUH\growing_apc_LOW>
(venv_new) PS D:\python\CUH\growing_apc_LOW> Write-Host "`n=== 완료. 결과: $summary ==="

=== 완료. 결과: outputs\retrain_compare\k_summary.txt ===
(venv_new) PS D:\python\CUH\growing_apc_LOW> Get-Content $summary -Encoding UTF8
=== k 개선 재학습 결과 ===
[K1_lr02] k=0.288 MAE=38.07 Bias=-16.27 nonsat_MAE=30.31 (baseline: k=0.254 MAE=37.17)
[K2_lr05] FAILED - no val csv
[K3_bag1] FAILED - no val csv
