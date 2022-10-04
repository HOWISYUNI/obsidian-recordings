# QUERY

# TROUBLE
## Waiting for table metadata lock
*lock 걸린 프로세스 이전에 의심가는 프로세스를 지워라*인데....
![[Pasted image 20221004170008.png]]
이걸 어떻게 아냐고....
한다면!

### 내 상황
1. 프로세스가 도는와중에 A 테이블을 dbeaver를 통해 여러번 select 했었다
2. airflow(10.120.40.109) 서버에서 A table Truncate를 하려했는데 lock

### 역추론
1. `SHOW FULL PROCESS LIST` 시 Info의 A 테이블 관련 프로세스들을 다 kill했고
2. 내 로컬 dbeaver에서 select 했으니까 내 로컬 ip + lock 프로세스 이전에 실행된 프로세스들을 다 kill 했다
	- 요기가 찾기 힘들었는데
	   내 로컬이 `SHOW FULL PROCESSLIST` 시 **cs009802.lottechilsung.co.kr**로 찍혔고, cmd 에서 `ping cs009802.lottechilsung.co.kr` 하니까 내 로컬(10.120.40.175)로 dns가 잡혀있음을 알 수 있었다
	   그래서 cs009802.lottechilsung.co.kr 관련된 프로세스들도 다 죽였다
3. lock 걸린 프로세스가 뿅 하고 사라짐! 진행시켜!


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
