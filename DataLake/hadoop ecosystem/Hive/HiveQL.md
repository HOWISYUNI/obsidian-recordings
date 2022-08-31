# CREATE TABLE
# ALTER TABLE
## ADD PARTITION
## DROP PARTITION
## CHANGE COLUMN
# SHOW PARTITIONS
# DESCRIBE
# GROUP
## GROUP BY
일반적인 그룹바이
```sql
SELECT pv_users.gender, count(DISTINCT pv_users.userid), count(*), sum(DISTINCT pv_users.userid)
FROM pv_users
GROUP BY pv_users.gender
```
## GROUPING SETS
하나 이상의 Group By 키를 설정할 때 사용
다수의 Group by 쿼리에 대한 UNION 결과와 동일함
![[Pasted image 20220831171527.png]]

## Cubes
group by key 값에 대해 모든 가능한 조합으로 group by를 수행
![[Pasted image 20220831172259.png]]

## Rollups
hierarchy level의 aggregation을 수행
Group by a,b,c는 a를 drill down 하는 hirearchy를 가정
![[Pasted image 20220831172357.png]]

----------------------------------------------------------------------
# 데이터타입
![[Pasted image 20220818162246.png]]
![[Pasted image 20220818162254.png]]
