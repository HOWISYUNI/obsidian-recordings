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
## top
[출처](https://zzsza.github.io/development/2018/07/18/linux-top/)
### 상단 설명
![[Pasted image 20221006175858.png]]
3:58 : 3시간 58분전 서버가 구동
load average : 1, 5, 15분 간 평균 실행/대기 중인 프로세스의 수. 코어수보다 적으면 문제없음
Tasks : 프로세스 갯수
KiB Mem, Swap : 각 메모리 사용량

### 테이블
VIRT : 프로세스의 virtual memory 총합 (VIRT = RES + SHR)
RES : 프로세스의 물리메모리 
SHR : 프로세스의 shared memory
PR : 실행 우선순위
S : 프로세스 상태
	D : Uninterruptible sleep. 디스크 네트워크 IO 대기
	R : 실행중(CPU 소모)
	S : Sleeping. 요청한 리소스 즉시 사용 가능
	T : Traced or sTopped.
	Z : Zombie. 부모 프로세스가 죽은 자식 프로세스

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
