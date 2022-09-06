# QUERY

# TROUBLE
## killed process still on my process list
![[Pasted image 20220906171808.png]]
### 상황
1. jupyter 에서 pymysql을 통한 executemany메서드 수행 중 insert 속도가 느려지면서 주피터 해당 커널 중지
2. 해당 프로세스 id로 kill 한 뒤 작업 대상 테이블을 수정하려하니 주피터 프로세스(스레드)가 프로세스리스트에 killed 상태로 계속 남아있었고, 대상 테이블에 대한 DML(컬럼 변경, CRUD)이 불가능(LOCK)
### 원인 및 해결방안
#### 원인
![[Pasted image 20220906172409.png]]
innodb 트랜잭션 롤백 중이라 killed 상태로 스레드가 남아있는 상태. [출처](https://dba.stackexchange.com/questions/5654/internal-reason-for-killing-process-taking-up-long-time-in-mysql)
#### 해결방안
1. 기다리기. 테이블 상태가 롤백될때까지 기다려야함.![[Pasted image 20220906172528.png]]
2. [force recovery](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html)
3. (권장하지 않음) 프로세스 강제 중지, 데몬 중지, 서비스 중지, 디스크 공간확보![[Pasted image 20220906172657.png]]

## How to Insert data into Maria DB
[출처](https://mariadb.com/kb/en/how-to-quickly-insert-data-into-mariadb/)
![[Pasted image 20220906191116.png]]
### 베스트 : LOAD DATA INFILE
![[Pasted image 20220906191239.png]]
