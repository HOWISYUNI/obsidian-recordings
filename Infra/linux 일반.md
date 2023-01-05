# 명령어
| 역할                     | 명령어                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------ |
| 파일 정보 확인           | `ll` (user - group 순서!)                                                                              |
| 리얼타임 자원 현황       | `top`  [[linux 일반#top]]                                                                              |
| 프로세스 리스트          | `ps -aux`                                                                                              |
| root 계정 빌리기         | sudo su                                                                                                |
| 프로세스 죽이기          | kill {옵션} {프로세스ID}  [블로그](https://www.lesstif.com/system-admin/unix-linux-kill-12943674.html) |
| 빈 디렉터리 삭제         | rmdir {디렉터리}                                                                                       |
| 채워져있는 디렉터리 삭제 | rm -r {디렉터리}                                                                                       |
| 특정위치로 압축풀기      | unzip {zip 파일} -d {디렉터리}                                                                         |
| 원격 파일 복사           | scp  {user}@{SRC_HOST}:{file위치} {user}@{DEST_HOST}:{디렉터리}                                        |
|                          |                                                                                                        |
## TOP : 서버 성능(CPU, 메모리) 모니터링
[출처 1](https://zzsza.github.io/development/2018/07/18/linux-top/) [출처 2](https://sabarada.tistory.com/146)
![[Pasted image 20230105095711.png]]
### 요약영역
전체 프로세스가 OS에 대해 리소스를 어느정도 차지하는지 요약
![[Pasted image 20230105095818.png]]
1. System
	1. 16:58:55 = System time : 시스템 현재 시간(GMT 기준. +9 시 KST)
	2. up 7 days 1 : 15 = OS 살아있던 시간
	3. 2users = 유저 세션 수 (`who` 명령어로 상세하게 확인 가능)
2. Load Average : CPU Load 이동평균
	1. 0.00 : 1분 이동 평균
	2. 0.00 : 5분 이동 평균
	3. 0.00 : 15분 이동 평균
	4. 해석
		1. 로드 : CPU가 실행 or 대기중인 프로세스의 양. 
		2. 1.0 = 100%
		3. 멀티코어 환경일 경우 `이동평균 x 코어 수` 를 하여 load 계산
		4. 1.0 초과 시 CPU에서 처리하지 못하고 대기중인 프로세스가 있다고 판단
3. Tasks : 현재 프로세스의 상태
	1. ![[Pasted image 20230105100631.png]]
	2. 프로세스 실행 = CPU 기반의 일(CPU - bound) ---> IO 이벤트(IO - bound)가 생길경우 waiting
4. CPU : CPU가 어떤 비율로 사용되고 있는지
	1.  us : 프로세스의 유저 영역에서의 CPU 사용률
	2. sy : 프로세스의 커널 영역에서의 CPU 사용률
	3. ni : 프로세스의 우선순위(priority) 설정에 사용하는 CPU 사용률
	4. id : 사용하고 있지 않는 비율
	5. wa : IO가 완료될때까지 기다리고 있는 CPU 비율
	6. hi : 하드웨어 인터럽트에 사용되는 CPU 사용률
	7. si : 소프트웨어 인터럽트에 사용되는 CPU 사용률
	8. st : CPU를 VM에서 사용하여 대기하는 CPU 비율
5. Memory
	1. 종류
		1. mem : RAM 메모리
		2. swap
			1. 디스크를 메모리처럼 사용하는 swap 영역
			2. 리눅스는 프로세스 빠른 처리를 위해 
				1. 실행중인 프로세스 : RAM 사용
				2. 나머지 : Disk cache (swap 메모리) 사용
	2. 상태
		1. total : 총 메모리 양
		2. free : 사용 가능한 메모리 양
		3. used : 사용중인 메모리 양
		4. buff/cache
			1. ![[Pasted image 20230105102837.png]]
			2. buff/cache : IO 성능 향상을 위해 커널이 사용하는 메모리
			3. 설명
				1. 데이터는 디스크에서 읽지만, 속도가 느려 읽은 내용은 메모리에 기억하고 재처리하는 것을 '캐싱' 이라 함
				2. 캐시에는 두가지가 있는데 `Page Cache` 와 `Buffer Cache` 가 있음
				3. `메모리 = 사용영역 + 캐시영역` 로 생각하면 프로세스에서 필요한 데이터가 메모리 사용영역보다 크면 캐시영역을 가져가서 사용
				4. 캐시까지 다 쓰면 swap 영역을 사용함

### 디테일영역
![[Pasted image 20230105102920.png]]
1. USER
    -   해당 프로세스를 실행한 USER 이름 또는 효과를 받는 USER의 이름입니다.
2. PR & NI
    -   PR : 커널에 의해서 스케줄링되는 우선순위입니다.
    -   NI : PR에 영향을 주는 nice라는 값입니다.
3. ***VIRT, RES, SHR, %MEM : 프로세스의 메모리***
    -   VIRT = RES + SHR
	    - 프로세스가 소비하고 있는 총 메모리입니다. 프로그램이 실행중인 코드, heap, stack과 같은 메모리, IO buffer 메모리를 포함합니다.
    -   RES : RAM에서 사용중인 메모리의 크기를 나타냅니다.
    -   SHR : 다른 프로세스와의 공유메모리(Shared Memory)를 나타냅니다.
    -   %MEM : RAM에서 RES가 차지하는 비율을 나타냅니다.
-   S : 프로세스의 현재 상태를 나타냅니다
	- D : Uninterruptible sleep. 디스크 네트워크 IO 대기
	- R : 실행중(CPU 소모)
	- S : Sleeping. 요청한 리소스 즉시 사용 가능
	- T : Traced or sTopped.
	- Z : Zombie. 부모 프로세스가 죽은 자식 프로세스
-   TIME+ : 프로세스가 사용한 토탈 CPU 시간
-   COMMAND : 해당 프로세스를 실행한 커맨드를 나타냅니다.
### 정렬
1. 메모리 사용량 순 정렬 : `shift + m`
2. CPU 사용량 순 정렬 : `shift + p`
3. 실행 시간 순 정렬 : `shift + t`
4. 메모리 사용률 시각화 : `p`
### 도커 컨테이너안에서 동작하는 프로세스
![[Pasted image 20230105103916.png]]
메모리 50기가 먹는 프로세스가 도커컨테이너 안에서 돌고있다고 예측된다
의심되는 [컨테이너에 대해 top 찍고 pid로 검색해봄]([[Docker#top : 컨테이너 내 프로세스 리소스 모니터링]]) `docker container {컨테이너명} | grep {키워드}` ex. docker container top tljh-dev | grep 2429137
![[Pasted image 20230105104111.png]]
이렇게 하면 대충 주피터에서 현종매니저님이 메모리를 50기가 먹고있구나 라고 생각하면됨

# 디렉터리
## /var 
[출처](https://jadehan.tistory.com/11)

## /usr
[출처](https://jadehan.tistory.com/3?category=836227)

# 파일
## 개요
[출처](https://nerd-mix.tistory.com/28)
파일 종류


## 링크
[출처](https://devkingdom.tistory.com/158)
### 하드링크
### 심볼릭 링크

# 사용자 그룹 관리
[출처](https://www.leafcats.com/132)
리눅스는 하나의 OS를 여러 사용자가 동시에 접속하는 경우가 많다(Multi-User System)
## 사용자 정보 관리
/etc/passwd
![[Pasted image 20221003195522.png]]

## 그룹 정보 관리
/etc/group
![[Pasted image 20221003195458.png]]



# crontab
## 설치
```bash
# db 컨테이너 안에서

# 설치
sudo apt update -y
sudo apt install -y cron

# cron 시작
sudo service cron start
# cron 서비스 상태 확인
sudo service cron status

# 위 과정으로 안되면
# cron systemctl 활성화 
sudo systemctl enable cron.service 
# cron systemctl 등록 확인 
sudo systemctl list-unit-files | grep cron

```

## 크론 잡 핸들링
```bash
# crontab 잡 추가
crontab -e
# job 추가 : 파일에 sql 정의하고 cron 잡 정의
# 매시 0분에 동작
0 * * * * psql -U postgres --host localhost --port 5432 --dbname ansung_relay < /home/delete_data.sql

# cron 잡 리스트 확인
crontab -l
```

# scp
![[Pasted image 20221129104104.png]]
```bash
scp ngdbc.jar lcsasmdl@10.121.117.175:/home/lcsasmdl/다운로드
```
