![[Pasted image 20220818161629.png]]
![[Pasted image 20220818161535.png]]

# Hive Parquet Table 도입
1. 추출 데이터를 HDFS에 parquet 파일로 저장
2. Hive on Tez 의 property 설정 : hive.vectorized.execution.enabled=true;
3. Hive Parquet 테이블 생성
4. parquet파일로 파티셔닝

## Hive on Tez Vectorized
cli 환경에서는 아래와 같이 처리하고
![[Pasted image 20220908122131.png]]

우리같이 Ambari 사용할 경우 아래와 같이 처리한다
Ambari > Hive > configs > advanced > General > Enable Vectorization and Map Vectorization
![[Pasted image 20220908121927.png]]
### Hive Parquet External 테이블 생성
[출처](https://moons08.github.io/programming/Hive_Table/)
![[Pasted image 20220908122025.png]]

