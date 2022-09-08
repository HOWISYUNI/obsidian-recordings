# Parquet
## 사용하는 이유
1. Hive 성능 향상 : Hive Vectorized Operation 사용 가능
2. CSV 에 비해 작은 파일 크기
## 압축알고리즘 별 비교
[출처](https://stackoverflow.com/questions/35789412/spark-sql-difference-between-gzip-vs-snappy-vs-lzo-compression-formats)
| 압축 알고리즘 | 장점                           | 단점                           |
| ------------- | ------------------------------ | ------------------------------ |
| gzip          | 높은 압축률 = 파일 크기 작다   | 느리다(Throughput), Split 불가 |
| snappy(, lzo   | 빠르다(Throughput), Splittable | 낮은 압축률 = 파일 크기 크다   |

![[Pasted image 20220908122825.png]]
