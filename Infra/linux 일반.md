# 명령어
| 역할               | 명령어    | 설명                                                |
| ------------------ | --------- | --------------------------------------------------- |
| 파일 정보 확인     | `ll`      | 권한-링크수-user-group-파일크기-최종수정일자-파일명 |
| 리얼타임 자원 현황 | `top`     | [[linux 일반#top]]                                                    |
| 프로세스 리스트    | `ps -aux` | a, u, x 옵션. `ps -aux | grep 머시기` 으로 검색     |
|                    |           |                                                     |
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
