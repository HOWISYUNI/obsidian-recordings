# 삭제
![[Pasted image 20220903103932.png]]
1. 열삭제 : DROP COLUMN
![[Pasted image 20220903104058.png]]
3. 행삭제 : DELETE
![[Pasted image 20220903104025.png]]
4. 데이터 삭제 : TRUNCATE
![[Pasted image 20220903104038.png]]
6. 테이블 삭제 : DROP
![[Pasted image 20220903104050.png]]


# 중복 PK 피하고 INSERT
## TURNCATE & BULK INSERT
서비스중단을 피할 수 없다
## INSERT IGNORE
PK 중복될 경우 기존 데이터 유지
	= 기존 row 유지
	= 신규 row 추가
```SQL
INSERT IGNORE INTO [TABLE] (COLUMN1, COLUMN2, ...)  
VALUES (VALUE1, VALUE2, ...)
```
## REPLACE INTO
PK가 중복될 경우 기존 데이터 DELETE -> 신규 데이터 INSERT
	= 기존 row update
	= 신규 row 추가
내부적으로는 PK검사 후 INSERT, UPDATE를 분기하는로직이므로 성능이 좋진 않음
사용자 입장에선 편한 쿼리
```sql
REPLACE INTO [TABLE명](변경하고자 하는 필드 명) VALUES(...)
```
## 추천! ON DUPLICATE KE UPDATE 
```SQL
INSERT INTO 테이블명 (컬럼a, 컬럼b, 컬럼c) 
VALUES (1,2,3) 
ON DUPLICATE KEY UPDATE 컬럼c = 컬럼c+1;
```

# SHOW PROCESSLIST
mysql, mariaDB
```SQL
SHOW [FULL] PROCESSLIST
```

# KILL process
![[Pasted image 20220906172822.png]]
[출처](https://mariadb.com/kb/en/kill/)
# LOAD DATA INFILE
BULK INSERT 중 가장 속도가 빠른 쿼리 (일반 INSERT 쿼리의 약 20배)
추출 -> csv 파일 떨굼 -> csv 파일을 읽어 TABLE INSERT(= LOAD DATA LOCAL INFILE)
1. 사전 조건 [출처](https://techtoart.tistory.com/52)
	- SERVER : local_infile 변수 'ON' ![[Pasted image 20220906191352.png]]
	- CLIENT
		- python API ![[Pasted image 20220906184735.png]]
		- SQL cli  
		-![[Pasted image 20220906184829.png]]
		- DBeaver (확인필요) ![[Pasted image 20220906185008.png]]
		
2. 문법 [출처(MySQL 8.0 Document)](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)[한글자료](https://getchan.github.io/data/mysql_load_data/)
![[Pasted image 20220906185047.png]]
- 예시
```sql
LOAD DATA LOCAL INFILE '{path}'
REPLACE                                           -- REPLACE ; 중복키값 INSERT 시 처리 방법
INTO TABLE msfa.f_weekly_2021_temp
FIELDS                                            -- FIELDS 구문 
    TERMINATED BY ','                             -- TERMINATED ; 필드 구분자
    OPTIONALLY ENCLOSED BY '"' ESCAPED BY  '\'     -- ENCLOSED : VALUE를 해당 문자로 감싼다. ESCAPED BY : 허용할 특수문자
LINES
    TERMINATED BY '\n'  -- TERMINATED : 개행 문자
    STARTING BY ''     -- STARTING BY : 문자를 모든 VALUE PREFIX로 사용. 해당 문자 없을 시 무시.
IGNORE 1 LINES  # IGNORE : 필드명 라인 첫 번째 줄 생략 가능
(@vyyyy, @vweekno, 
@vsec_cd, @vchannel_cd, @vdist_cd, @vstore_cd, @vsido, @vsigungu, @vvendor_cd, 
@vcategory1_cd, @vcategory2_cd, @vcategory3_cd, @vcategory4_cd, @vcategory5_cd, @vbarc, @vproduct_no, 
@vpkg_nm, @vpkg_size, @vvolume, 
@vcount, 
@vpromotion, 
@vmain_barc, 
@vpt_dist,
@vpt_week,
@vqty,
@vamount,
@vprice,
@vunit_qty,
@vunit_price,
@vprice_per_100)

SET                                               -- SET : 데이터 전처리 후 할당
    yyyy = NULLIF(@vyyyy, ''),                    -- NULLIF(str1, str2) : str1==str2 일 경우 NULL 할당(테이블 필드 디폴트 설정이 안먹어서 이렇게 해줌)
    weekno = NULLIF(@vweekno, ''),
    sec_cd = NULLIF(@vsec_cd, ''),
    channel_cd = NULLIF(@vchannel_cd, ''),
    dist_cd = NULLIF(@vdist_cd, ''),
    store_cd = NULLIF(@vstore_cd, ''),
    sido = NULLIF(@vsido, ''),
    sigungu = NULLIF(@vsigungu, ''),
    vendor_cd = NULLIF(@vvendor_cd, ''),
    category1_cd = NULLIF(@vcategory1_cd, ''),
    category2_cd = NULLIF(@vcategory2_cd, ''),
    category3_cd = NULLIF(@vcategory3_cd, ''),
    category4_cd = NULLIF(@vcategory4_cd, ''),
    category5_cd = NULLIF(@vcategory5_cd, ''),
    barc = NULLIF(@vbarc, ''),
    product_no = NULLIF(@vproduct_no, ''),
    pkg_nm = NULLIF(@vpkg_nm, ''),
    pkg_size = NULLIF(@vpkg_size, ''),
    volume = NULLIF(@vvolume, ''),
    count = NULLIF(@vcount, ''),
    promotion = NULLIF(@vpromotion, ''),
    main_barc = NULLIF(@vmain_barc, ''),
    pt_dist = NULLIF(@vpt_dist, ''),
    pt_week = NULLIF(@vpt_week, ''),
    qty = NULLIF(@vqty, ''),
    amount = NULLIF(@vamount, ''),
    price = NULLIF(@vprice, ''),
    unit_qty = NULLIF(@vunit_qty, ''),
    unit_price = NULLIF(@vunit_price, ''),
    price_per_100 = NULLIF(@vprice_per_100, '')
```
3. 이슈
	- Load NULL or None value from csv file
	추출 데이터프레임 -> to_csv 시, np.nan이나 None이 파일 내용에서 빈칸('')으로 변함
	_테이블 디폴트값이 설정돼있음에도 불구하고_ 이 데이터를 읽어 LOAD DATA LOCAL INFILE 수행 시 아래와 같은 오류 발생	![[Pasted image 20220906185808.png]]
		- 해결방안 [출처](https://stackoverflow.com/questions/2675323/mysql-load-null-values-from-csv-data)
			1. 적재 SQL 내 SET구문 을 통해 NULL로 초기화![[Pasted image 20220906190231.png]]
			2. ~~none 데이터를 `\N`으로 변경~~![[Pasted image 20220906190246.png]]
				아래 코드로 추출 데이터 프레임 내 NULL을 `\N`으로 변경하고 LOAD DATA LOCAL INFILE 구문 수행 시 아래와 같이 오류 발생 ![[Pasted image 20220906190424.png]]
