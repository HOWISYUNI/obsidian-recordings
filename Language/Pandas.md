# 문법
## pandas.DataFrame.to_csv()
![[Pasted image 20220906190731.png]]
1. parameters
	- header=False : dataframe에서 필드 이름 라인 제거하여 저장
	- index=False : dataframe에서 행 번호 삭제하여 저장

# DataFrame
[document](https://pandas.pydata.org/docs/reference/frame.html)
## 개략적 파악
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

### Row count
df['column'].size = df.count() + df.isnull().sum()
```python
df['column_name'].size # NaN 포함. df.size = df column 갯수 * df row 갯수
df.count() # NaN 미포함 row 갯수
df.isnull().sum() # NaN row 갯수
df['column_name'].value_counts() # 특정 colum의 특성 별 row 갯수
```
value_counts 결과를  df 에 병합
```python
df['value_counts'] = df.column_name.map(df.column_name.value_counts())
```
### unique
```python
df['column_name'].unique() # unique value 수
df['column_name'].nunique() # unique value 종류 수
df['column_name'].value_counts() # unique value 별 row 수
```
![[Pasted image 20221003183403.png]]

## 조작하기
### Slicing, Selecting, Filtering
#### Column
```python
DataFrame[['FieldName1', 'FieldName2', ...]]
```
#### Row
```python
# SLICING
DataFrame[start : end+1]

# 조건
DataFrame[DataFrame.컬럼 조건식]
DataFrame[(DataFrame.필드1 조건식) & (DataFrame.필드2 조건식) ...]
DataFrame[(DataFrame.필드1 조건식) | (DataFrame.필드2 조건식) ...]
```

#### Row Index & Column Index access : iloc
```python
# ROW
# ROW SELECTING
DataFrame.iloc[[10,20]]   # row selecting

# ROW SLICING
DataFrame.iloc[10:30]     # row slicing

# COLUMN
# COLUMN SELECTING
DataFrame.iloc[[10,30], 1] # Series 리턴
DataFrame.iloc[[10,30], [1]] # DataFrame 리턴
DataFrame.iloc[[10,30], [1, 3]]
DataFrame.iloc[10:30, [1, 3]]

# COLUMN SLICING
DataFrame.iloc[10:30, 1:3] 
DataFrame.iloc[[10,30], 1:3]
```

#### Column Name access & 조건  : loc[row 조건, (column 범위)]
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
### Combine
#### Join
**how : left, right, inner, outer**
```python
pd.merge(df_left, df_right, how=..., on=...)
pd.merge(df_left, df_right, how=..., on=['col1', 'col2', ...])
pd.merge(df_left, df_right, how=..., left_on=..., right_on=...)
```
#### Union
```python
pd.concat([df1, df2])
```
### iteration
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

### 함수 적용시키기
1. map : Series 객체에만 적용
	* lambda 함수로 조작
	* index로 두 series 맵핑 [[Pandas#Row count]]
	![[Pasted image 20221003185825.png]]
```python
x = pd.Series({'one':1,'two':2,'three':3})
y = pd.Series({1:'triangle',2:'square',3:'circle'})

x.map(y)
# one triangle
# two squre
# three circle
```
2. apply : Dataframe 의 Series 객체(행, 열) 단위로 적용 (default : column)
3. applymap : Dataframe의 요소 별 적용
```python
# map : Series에만 적용
sr.apply(lambda x: x*10)

# apply : DataFrame의 행, 열 단위로 적용
df.apply(lambda col: col.sum()) # column 단위 sum. df.sum()과 동일
df.apply(lambda row: row.sum(), axis=1) # row 단위

# applymap : DataFrame의 item 단위로 적용
df.applymap(lambda x : '%.2f'% x if (x > 5)  else x) # item별로 iteration 돌아 적용
```

### Type 변환
[document](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html)
```python
df.astype(dtype)
```
# DataFrame.GroupBy
[document](https://pandas.pydata.org/docs/reference/groupby.html)
![[Pasted image 20221003192422.png]]
1. by : list. 그룹핑 기준되는 column 들의 레벨 순서로 리스트 전달
2. level : MultiIndex DataFrame의 경우 특정 level로 group by. 0부터 시작
3. 

## 함수
```python
df = pd.DataFrame([[1.1, 1.1, 1.1, 2.6, 2.5, 3.4,2.6,2.6,3.4,3.4,2.6,1.1,1.1,3.3], list('AAABBBBABCBDDD'), [1.1, 1.7, 2.5, 2.6, 3.3, 3.8,4.0,4.2,4.3,4.5,4.6,4.7,4.7,4.8], ['x/y/z','x/y','x/y/z/n','x/u','x','x/u/v','x/y/z','x','x/u/v/b','-','x/y','x/y/z','x','x/u/v/w'],['1','3','3','2','4','2','5','3','6','3','5','1','1','1']]).T
df.columns = ['col1','col2','col3','col4','col5']
```
![[Pasted image 20221003192938.png]]
### Row count
![[Pasted image 20221003193022.png]]
### max 
![[Pasted image 20221003193045.png]]
### reset_index() : groupby ->  DataFrame
![[Pasted image 20221003193111.png]]
# Series
## Iteration
Series.items()
```python
for index, value in sr.items():
	# do something
```
