# Parquet
컬럼 지향 데이터 포맷
## 사용하는 이유
1. Hive 성능 향상 : Hive Vectorized Operation 사용 가능 [출처](https://blog.cloudera.com/faster-swarms-of-data-accelerating-hive-queries-with-parquet-vectorization/)
	- row-based exectuion : 매 row 마다 연산
	- vectorized execution : 필드 별로 연산 = 한 명령에 다중 연산(1024행 동시 연산) [HIVE 레퍼런스](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)
![[Pasted image 20220908142902.png]]
![[Pasted image 20220908142915.png]]

2. CSV 에 비해 우수한 퍼포먼스[출처](https://dzone.com/articles/how-to-be-a-hero-with-powerful-parquet-google-and)
![[Pasted image 20220908140713.png]]
[출처2](https://beomi.github.io/2020/01/29/Use-parquet-on-pandas/)
![[Pasted image 20220908143621.png]]

## 압축알고리즘 별 비교
[출처](https://stackoverflow.com/questions/35789412/spark-sql-difference-between-gzip-vs-snappy-vs-lzo-compression-formats)
| 압축 알고리즘 | 장점                           | 단점                           |
| ------------- | ------------------------------ | ------------------------------ |
| gzip          | 높은 압축률 = 파일 크기 작다   | 느리다(Throughput), Split 불가 |
| snappy(, lzo   | 빠르다(Throughput), Splittable | 낮은 압축률 = 파일 크기 크다   |

![[Pasted image 20220908122825.png]]
