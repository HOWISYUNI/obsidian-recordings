# 세팅
중계용 DB
```bash
# 이미지 다운로드
docker pull postgres:14.6

# 데이터 마운트 용 볼륨 설정
docker volume create pgdata

# 컨테이너 백그라운드 동작
docker run -p 5432:5432 --name postgresql -e POSTGRES_PASSWORD=postgresql -e TZ=Asia/Seoul -v pgdata:/var/lib/postgresql/data -d postgres:14.6

# (안해줘도됨)PostgreSQL URL 설정. 외부 DB에 연결하기 위해 URL 설정
export DB_URL = 'postgresql://postgres:postgresql@IP:5432/postgres'

# vim 설치
apt-get update
apt-get install vim

# 컨테이너 내부 진입 후 DB 쉘 동작
# super user : postgres / postgresql
# 추가 user : ansung / ansung
psql -U postgres

# DB 리스트 확인
postgres=# \l

# super user로 DB 생성
CREATE DATABASE ansung_relay

# dbeaver에서 USER 생성. default는 NOSUPERUSER, NOCREATEDB, NOCREATEUSER
# ansung 계정으로 ansung_relay에 테이블 생성 시 그 테이블에만 권한 들어감
create user ansung password 'ansung'

# 권한 조회
# 연결 계정별로 볼수있는 범위가 다름
select * from information_schema.role_table_grants
```

## system tables
### LOCK
```SQL
-- 수행중인 SQL 전체 조회
select * from pg_stat_activity;

-- LOCK 테이블 확인
SELECT t.relname, l.locktype, page, virtualtransaction, pid, mode, granted 
FROM pg_locks l, pg_stat_all_tables t 
WHERE l.relation = t.relid 
ORDER BY relation ASC;

-- 해당 프로세스만 KILL
SELECT pg_cancel_backend([pid]);

-- (권장) 연계된 프로세스 모두 종료
SELECT pg_terminate_backend([pid]) FROM pg_stat_activity;

```

# PL/pgSQL
PostgreSQL에서 제공하느 프로그램 언어
리턴값이 필요하면 function을 만들고 (얘가 좀더 권장하는 방법인듯)
리턴값이 필요없으면 procedure를 만든다

## function
[tutorial](https://www.postgresqltutorial.com/postgresql-plpgsql/)
[설명잘된 블로그](http://www.gisdeveloper.co.kr/?s=pl%2Fpgsql+%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC)

# trigger
[아주아주 정리잘된 블로그](https://jaeone94.github.io/posts/Postgresql-%ED%8A%B8%EB%A6%AC%EA%B1%B0%EB%A5%BC-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90/)
트리거 : 테이블에 변화(이벤트, INSERT, UPDATE, DELETE)가 생기기 전, 후 동작으로 뭔가를 하고싶을 때 지정
## 일반적인 흐름
1. 원하는 동작을 function으로 정의하고 ex. update_datetime()
2. 관심있는 테이블에 function을 엮은 trigger 생성 ex. trg_after_insert_tag01
3. 트리거 생성 구문(create trigger) 수행 후 커밋

## 문법
```SQL
CREATE OR REPLACE TRIGGER 트리거_이름  
{ BEFORE | AFTER | INSTEAD OF }     -- 트리거 실행 시점
{ DML이벤트 [ OR DML이벤트 ... ] }     -- 트리거 이벤트 (INSERT or UPDATE or DELETE ..)
ON 트리거를_적용시킬_테이블_또는_뷰_이름
FOR EACH { ROW | STATEMENT } 		-- 트리거 실행범위 ( 각행 또는 한번만 )
[WHEN (조건)] 						-- 트리거 실행 조건( EACH ROW 일 때 조건에 만족하는 행만 수행가능 )
EXCUTE { FUNCTION | PROCEDURE } 함수명("문자열_상수_파라미터");


-- 예시
create trigger trg_before_insert_tag22 -- 트리거 이름 선언
before insert on tag22_csc_ins_temp02 -- 
for each row execute function update_datetime()
```


## trigger 시점
### before
### after
## function 리턴 값

