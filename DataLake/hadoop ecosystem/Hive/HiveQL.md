# CREATE EXTERNAL TABLE
## CSV
```SQL
CREATE EXTERNAL TABLE ods.factory_as_mes_bom_info(
	po_cd varchar(50) COMMENT 'PO코드',  -- NOT NULL 제한 불가능
	po_mat varchar(100) COMMENT 'PO제품코드',
	po_mat_name string COMMENT 'PO제품명',
	bom_version varchar(100) COMMENT 'BOM버전',
	bom_number int COMMENT 'BOM아이템', 
	bom_mat varchar(50) COMMENT 'BOM자재코드',
	bom_mat_name string COMMENT 'BOM자재명',
	bom_mat_cls varchar(50) COMMENT 'BOM자재분류코드',
	bom_mat_cls_name string COMMENT 'BOM자재분류명')
PARTITIONED BY (pt_ymd STRING, vkgrp STRING) -- 파티셔닝 할 경우 기준 
ROW FORMAT DELIMITED
      FIELDS TERMINATED BY '' -- 오류가 나지 않을법한 구분자
      ESCAPED BY '\\'
      LINES TERMINATED BY '\n' 
STORED AS TEXTFILE              -- 포맷 : 텍스트파일
LOCATION '/path/to/your/data' -- INSERT문 수행 시 데이터가 쌓이는 위치, 생략가능
```
## PARQUET
```SQL
CREATE EXTERNAL TABLE ext_pos_ltmart_liq_sale(
	sale_date varchar(8) COMMENT '판매일자',
	store_cd varchar(5) COMMENT '점포코드',
	sale_time varchar(5) COMMENT '판매시각',
	barcd string COMMENT '제품바코드',
	vendor_cd string COMMENT '제조사코드',
	qty int COMMENT '판매수량',
	amount int COMMENT '판매금액')
PARTITIONED BY (pt_ymd STRING) -- 파티셔닝 할 경우 기준 
ROW FORMAT DELIMITED
	FIELDS TERMINATED BY ','
STORED AS PARQUET             -- 포맷 : Parquet
LOCATION '/path/to/your/data' -- INSERT 수행 시 데이터가 쌓이는 위치, 생략시 hdfs default 위치에 쌓임(https://stackoverflow.com/questions/47199342/where-data-will-be-stored-when-we-create-hive-external-table-without-location-pr)
```

# ALTER TABLE
## ADD PARTITION
## DROP PARTITION
## CHANGE COLUMN
![[Pasted image 20220928143142.png]]
![[Pasted image 20220928143151.png]]

## CHANGE EXTERNAL LOCATION
![[Pasted image 20221110122709.png]]
```SQL
ALTER TABLE ods.ext_pos_ltmart_liq_sale SET LOCATION "hdfs://csdlnn01:8020/ODS/PROD/EXTERNAL/POS/BW/MART/LIQ/SALES"
```
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
![[Pasted image 20230112160657.png]]