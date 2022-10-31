# 명령어
## CLI

| 의도                           | 명령어                                                         |
| ------------------------------ | -------------------------------------------------------------- |
| 컨테이너 시작                  | docker run -p {포트:포워딩} --name {컨테이너명} {이미지명:tag} |
| 모든 컨테이너 중지             | docker stop $(docker ps -a -q)                                 |
| 특정 컨테이너 중지             | docker stop {컨테이너명}                                                               |
| 특정 컨테이너 삭제             | docker rm   {컨테이너 ID}                                      |
| 중지된 컨테이너 삭제 (더 선호) | docker container prune                                         |
| 중지된 컨테이너 삭제           | docker rm $(docker ps --filter status=exited -q)               |
| 모든 이미지 삭제               | docker rmi $(docker images -q)                                 |
| 특정 이미지 삭제               | docker rmi {REPOSITORY}:{TAG}                                  |
| dangling 이미지 삭제           | docker image prune                                             |

# 주요 명령어
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

## Docker File

