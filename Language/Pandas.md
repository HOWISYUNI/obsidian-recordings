# 문법
## pandas.DataFrame.to_csv()
![[Pasted image 20220906190731.png]]
1. parameters
	- header=False : dataframe에서 필드 이름 라인 제거하여 저장
	- index=False : dataframe에서 행 번호 삭제하여 저장

## Dataframe 정보
### property
- 모양
	1.  DataFrame.shape (tuple) : m x n 의 dataframe을 (m, n) 으로 표현
- 내용
	1. DataFrame.values (numpy.ndarray): numpy 배열의 값
	2. DataFrame.columns (pandas.core.indexes) : 필드(헤더) 명
	3. DataFrame.index(pandas.core.indexes) : dataframe 인덱스
	4. DataFrame.dtypes (pandas.Series): 필드 별 데이터 타입
	5. DataFrame.empty (Bool) : 진짜 아무것도 없는가 (None, Null만 있으면 False)![[Pasted image 20220907143319.png]]

### 함수
1. DataFrame.info() : column명, null count, dtype
2. DataFrame.describe() -> Series, DataFrame : DataFrame의 기술통계량

### missing value = None, np.nan, Null
1. DataFrame.isnull() = isna()와 동일![[Pasted image 20220907143237.png]]
2. 컬럼 별 missing value 찾기
```python
# 1. 컬럼 별 missing value 카운트
df.isnull().sum()

# 2. missing value 확인하는 컬럼 특정해서 관찰
df.loc[df.fieldName.isnull()]

```
![[Pasted image 20220908114944.png]]

## Slicing, Selecting, Filtering
### Column
```python
DataFrame[['FieldName1', 'FieldName2', ...]]
```
### Row
```python
# SLICING
DataFrame[start : end+1]

# 조건
DataFrame[DataFrame.컬럼 조건식]
DataFrame[(DataFrame.필드1 조건식) & (DataFrame.필드2 조건식) ...]
DataFrame[(DataFrame.필드1 조건식) | (DataFrame.필드2 조건식) ...]
```

### Row Index & Column Index access : iloc
```python
# ROW SLICING
DataFrame.iloc[10:30]     # row slicing
# ROW SELECTING
DataFrame.iloc[[10,20]]   # row selecting

# COLUMN SELECTING
DataFrame.iloc[[10,30], 1] # Series 리턴
DataFrame.iloc[[10,30], [1]] # DataFrame 리턴
DataFrame.iloc[[10,30], [1, 3]]
DataFrame.iloc[10:30, [1, 3]]

# COLUMN SLICING
DataFrame.iloc[10:30, 1:3] 
DataFrame.iloc[[10,30], 1:3]
```

### Column Name access & 조건  : loc[row 조건, (column 범위)]
```python
df.loc[(df.필드1 조건식) &(or |) (df.필드2 조건식) ...]

# ROW SLICING
df.loc[10:30, 'FieldName1': 'FieldName2']
df.loc[10:30, ['FieldName1', 'FieldName2']]
df.loc[[10,30], 'FieldName1': 'FieldName2']

# ROW FILTERING
df.loc[(df.필드1 조건식) &(or |) (df.필드2 조건식), ['FieldName1', 'FieldName2', ...]]
df.loc[(df.필드1 조건식) &(or |) (df.필드2 조건식), ['FieldName1' : 'FieldName4']]
```
## Combine
### Join
**how : left, right, inner, outer**
```python
pd.merge(df_left, df_right, how=..., on=...)
pd.merge(df_left, df_right, how=..., on=['col1', 'col2', ...])
pd.merge(df_left, df_right, how=..., left_on=..., right_on=...)
```
### Union
```python
pd.concat([df1, df2])
```
## iteration
| 속도순위 | method     | `iterrrows` 대비 속도 |
| -------- | ---------- | --------------------- |
| 1        | itertuples | x 8.1                 |
| 2        | at/iat     | x 4                   |
| 3        | loc/iloc   | x 2.5                 |
| 4        | iterrrows  | x 1                   |
![[Pasted image 20220901171933.png]]

![[Pasted image 20220901171958.png]]

![[Pasted image 20220901171910.png]]

![[Pasted image 20220901171925.png]]
