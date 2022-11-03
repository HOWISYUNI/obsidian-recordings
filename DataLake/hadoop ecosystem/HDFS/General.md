# Hadoop 명령어
## hdfs 리스팅
```bash
hadoop fs -ls # 파일시스템 트리구조 출력
hadoop fs -ls /user/ # hdfs write 가능한 계정 리스트
```
![[Pasted image 20220929150541.png]]
# 빅데이터를 위한 binary 파일 포맷 = csv는 쓰지 않는게 좋다
| 포맷    | 장점                                 | 특징                                      |
| ------- | ------------------------------------ | ----------------------------------------- |
| Feather | 빠른 READ, WRITE, CPU, 메모리 성능   | 단기 보관용 데이터 포맷. 주피터 IO에 적합 |   
| Parquet | 준수한 READ, WRITE, CPU, 메모리 성능 | 대량, 장기 보관용. HADOOP, SPARK 에 적합  |   
![[Pasted image 20220908151335.png]]
## 퍼포먼스 비교
[출처](https://towardsdatascience.com/the-best-format-to-save-pandas-data-414dca023e0d)
### string 타입 데이터 : parquet 성능 우수
![[Pasted image 20220908151439.png]]
![[Pasted image 20220908151530.png]]
![[Pasted image 20220908151500.png]]
### pd.Categorical 타입 데이터 : feather 우수
![[Pasted image 20220908151849.png]]
![[Pasted image 20220908151923.png]]

## Apache Parquet
컬럼 지향 데이터 포맷 [공식 document](https://github.com/apache/parquet-format)
### 사용하는 이유
1. Hive 성능 향상 : Hive Vectorized Operation 사용 가능 [출처](https://blog.cloudera.com/faster-swarms-of-data-accelerating-hive-queries-with-parquet-vectorization/)
	- row-based exectuion : 매 row 마다 연산
	- vectorized execution : 필드 별로 연산 = 한 명령에 다중 연산(1024행 동시 연산) [HIVE 레퍼런스](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)
![[Pasted image 20220908142902.png]]
![[Pasted image 20220908142915.png]]

2. CSV 에 비해 우수한 퍼포먼스[출처](https://dzone.com/articles/how-to-be-a-hero-with-powerful-parquet-google-and)
![[Pasted image 20220908140713.png]]
[출처2](https://beomi.github.io/2020/01/29/Use-parquet-on-pandas/)
![[Pasted image 20220908143621.png]]

### 압축알고리즘 별 비교
[출처](https://stackoverflow.com/questions/35789412/spark-sql-difference-between-gzip-vs-snappy-vs-lzo-compression-formats)
| 압축 알고리즘 | 장점                           | 단점                           |
| ------------- | ------------------------------ | ------------------------------ |
| gzip          | 높은 압축률 = 파일 크기 작다   | 느리다(Throughput), Split 불가 |
| snappy, lzo   | 빠르다(Throughput), Splittable | 낮은 압축률 = 파일 크기 크다   |

![[Pasted image 20220908122825.png]]

### Python API
pyarrow 이나 fastparquet 패키지를 환경에 받고,

#### READ
![[Pasted image 20220908145252.png]]

#### WRITE
![[Pasted image 20220908145215.png]]

## Python AP를 활용해 Parquet을 DL에서 사용하기
### HDFS Client로 upload
1. 로컬에 parquet 파일로 떨구기
[Pandas to_parquet 문서](https://pandas.pydata.org/pandas-docs/version/1.1/reference/api/pandas.DataFrame.to_parquet.html)
```python
# 엔진은 디폴트 pyarrow 엔진
# 압축은 디폴트 snappy 압축
df.to_parquet(path=local_path_parquet, index=False, engine='pyarrow', compression='snappy')
```
2. hdfs로 upload
```python
hdfs_client.upload(hdfs_path=file_location, local_path=local_path_parquet, overwrite=True)
```
3. HIve에 parquet 테이블 생성하고
[[HiveQL#PARQUET]]
4. 테이블 필요하면 파티셔닝
[[HiveQL#ADD PARTITION]]

### Spark로 Hive 테이블에 Insert해 HDFS 에 parquet 자동 생성
[[Spark#parqute hive 테이블 데이터 insert]]
