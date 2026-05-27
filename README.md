
import pandas as pd
df = pd.read_csv('data/raw_daily/202605/fdc_with_3340_20260526.csv', encoding='utf-8-sig', low_memory=False)
# 컬럼명 소문자 통일
df.columns = df.columns.str.lower()
print('전체:', len(df), '행')

# 문제의 grower 만
target = 'TG100' # ← 여기 grower 이름 넣기
sub = df[df['grower'] == target]
print(f'{target} 행수:', len(sub))

if len(sub) > 0:
    print(f'\ncurrent_event 분포: {dict(sub["current_event"].value_counts())}')
    print(f'ingot_status 분포: {dict(sub["ingot_status"].value_counts())}')
    print(f'attempt 분포: {dict(sub["attempt"].value_counts())}')
    print(f'max_attempt 분포: {dict(sub["max_attempt"].value_counts())}')
    print(f'customer_grp_2: {dict(sub["customer_grp_2"].value_counts())}')
    print(f'length 범위: [{sub["length"].min():.0f}, {sub["length"].max():.0f}]')

    # 4가지 필터 적용 결과
    f = sub[(sub['current_event'] == 'BODY') &
            (sub['ingot_status'] == 'F/S') &
            (sub['attempt'] == 1) &
            (sub['attempt'] == sub['max_attempt'])]
    print(f'\n4조건 통과 후 행수: {len(f)}')


import pandas as pd
df = pd.read_csv('data/raw_daily/202605/fdc_with_3340_20260526.csv', encoding='utf-8-sig', low_memory=False)
# 컬럼명 소문자 통일
df.columns = df.columns.str.lower()
print('전체:', len(df), '행')

# 문제의 grower 만
target = 'TG113' # ← 여기 grower 이름 넣기
sub = df[df['grower'] == target]
print(f'{target} 행수:', len(sub))

if len(sub) > 0:
    print(f'\ncurrent_event 분포: {dict(sub["current_event"].value_counts())}')
    print(f'ingot_status 분포: {dict(sub["ingot_status"].value_counts())}')
    print(f'attempt 분포: {dict(sub["attempt"].value_counts())}')
    print(f'max_attempt 분포: {dict(sub["max_attempt"].value_counts())}')
    print(f'customer_grp_2: {dict(sub["customer_grp_2"].value_counts())}')
    print(f'length 범위: [{sub["length"].min():.0f}, {sub["length"].max():.0f}]')

    # 4가지 필터 적용 결과
    f = sub[(sub['current_event'] == 'BODY') &
            (sub['ingot_status'] == 'F/S') &
            (sub['attempt'] == 1) &
            (sub['attempt'] == sub['max_attempt'])]
    print(f'\n4조건 통과 후 행수: {len(f)}')
