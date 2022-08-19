# 명령어
## CLI
| 의도                 | 명령어                                           |
| -------------------- | ------------------------------------------------ |
| 모든 컨테이너 중지   | docker stop $(docker ps -a -q)                   |
| 중지된 컨테이너 삭제 | docker rm $(docker ps --filter status=exited -q) |
| 모든 이미지 삭제     | docker rmi $(docker images -q)                   |
| 특정 이미지 삭제     | docker rmi {IMAGE ID}                            |


## Docker File

