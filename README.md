  scripts\release\validate_cuh.py:294:                clip_cfg, customer_filter, x_cols, valid_status)
  scripts\release\validate_cuh.py:295:            if target_df is None or not cuh_pred_at:
> scripts\release\validate_cuh.py:296:                print(f" [{i}/{len(lots)}] {lot}: 예측 불가 (prev 부족/전처리 실패)", flush=True)
  scripts\release\validate_cuh.py:297:                continue
  scripts\release\validate_cuh.py:298:            rows = _target_rows_at_refs(target_df, rel_cfg, cuh_pred_at, cuh_prev_at)
  scripts\release\validate_cuh.py:299:            all_rows.extend(rows)
  scripts\release\validate_cuh.py:300:            print(f" [{i}/{len(lots)}] {lot}: {len(rows)}개 위치 저장", flush=True)
> scripts\release\validate_cuh.py:301:        except Exception as e:
  scripts\release\validate_cuh.py:302:            print(f" [{i}/{len(lots)}] {lot}: 오류 {e}", flush=True)
  scripts\release\validate_cuh.py:303:
  scripts\release\validate_cuh.py:304:    if not all_rows:
