# 명령어
https://blog.d0ngd0nge.xyz/docker-container/
https://www.daleseo.com/docker-run/
## CLI

| 의도                           | 명령어                                                         |
| ------------------------------ | -------------------------------------------------------------- |
| 컨테이너 생성 & 시작           | docker run -p {포트:포워딩} --name {컨테이너명} {이미지명:tag} |
| 컨테이너 시작                  | docker start {옵션} {컨테이너명}                               |
| 컨테이너 재시작                | docker restart {컨테이너명}                                    |
| 모든 컨테이너 중지             | docker stop $(docker ps -a -q)                                 |
| 특정 컨테이너 중지             | docker stop {컨테이너명}                                       |
| 특정 컨테이너 삭제             | docker rm   {컨테이너 ID}                                      |
| 중지된 컨테이너 삭제 (더 선호) | docker container prune                                         |
| 중지된 컨테이너 삭제           | docker rm $(docker ps --filter status=exited -q)               |
| 모든 이미지 삭제               | docker rmi $(docker images -q)                                 |
| 특정 이미지 삭제               | docker rmi {REPOSITORY}:{TAG}                                  |
| dangling 이미지 삭제           | docker image prune                                             |
| 컨테이너 내용 파악             | docker inspect {컨테이너 명}                                   |
| 볼륨 생성                      | docker volume create {볼륨 명}                                 |
| 볼륨 내용 확인                 | docker volume inspect {볼륨 명}                                |
| 볼륨 리스트 확인               | docker volume ls                                               |
| 볼륨 삭제                      | docker volume rm {볼륨 명}                                                               |


# 주요 개념 & 명령어
## RUN
```bash
docker run (<옵션>) <이미지 식별자> (<명령어>) (<인자>)
```
### 옵션
[출처](https://www.daleseo.com/docker-run/)
-it : 인터랙티브한 입력 창 활성화
-d : 백그라운드 실행
--name : 컨테이너 별명 지정
-p : 포트포워딩 `호스트포트:컨테이너내부포트` 로 정의
-v : 볼륨 마운트
-w : 초기 워킹 디렉터리 주소 설정. Dockerfile 의 WORKDIR 설정을 덮어씀
--rm : 컨테이너 종료 시 관련 리소스도 호스트에서 제거
--entrypoint : Dockerfile의 ENTRYPOINT 설정 덮어 씀

## 호스트 경로 마운트
[출처](https://www.daleseo.com/docker-volumes-bind-mounts/)
![[Pasted image 20221112170217.png]]
### 볼륨 
도커볼륨 개념 아래 생긴 파일들을 공유
```bash
# 볼륨 리스트 확인
docker volume ls

# 볼륨 생성
docker volume create kafka-vol

# 볼륨 상세 내용 확인
docker volume inspect kafka-vol

# 컨테이너에 볼륨 마운트하기
docker run -it -v kafka-vol:/app ...
```
### 바인드마운트
호스트의 특정 경로를 컨테이너와 공유
```bash
docker run -v 'pwd':/app ...
```
### 언제 뭘 사용해야하나
로컬 개발환경 구성 : 소스코드르 특정 경로에 클론받고, 그 경로를 바인드마운트하면 컨테이너 안에서 확인 가능. 컨테이너 안에서의 수정분도 반영 가능.
나머지 : 도커 볼륨(거의 대부분의 경우 볼륨을 사용하는걸 권장한다)

## Docker File


# 해결
## 이미있는 컨테이너에 새 볼륨을 마운트하고싶어요
[출처](https://stackoverflow.com/questions/28302178/how-can-i-add-a-volume-to-an-existing-docker-container)
1.지금 컨테이너를 이미지로 만들고, 그 이미지에 볼륨을 마운트해서 다시 컨테이너로 동작시켜라
![[Pasted image 20221112163733.png]]
2.호스트 경로를 컨테이너로 복사
![[Pasted image 20221112163923.png]]
3.컨테이너를 중지 -> 설정을 직접 변경 -> 컨테이너 재시작
![[Pasted image 20221112164026.png]]

