# Docker buildx
[공식문서](https://docs.docker.com/build/buildx/install/)
이미지를 병렬로 빠르게 빌드해주는 플러그인이래
근데 안정화버전이 아니라서 docker desktop으로 세팅하는걸 추천하는데, 우린 돈이 없으니까 도커엔진에 세팅을 해줘보자 ([buildkit 공식문서](https://docs.docker.com/build/buildkit/))
buildx가 동작하는지는 `docker buildx` 명령어를 쳐보면 되는데, 사실 2번까지 세팅해줬는데도 별거 없었고, 3번에서 바이너리를 넣어주니까 바로됐다. 앞 단계에 의존성이 있는지는 정확히 모르겠는데 암튼 됐다;
## 1. Docker Buildkit 설정
![[Pasted image 20221114184105.png]]
Docker Buidkit은 Docker Buildx에 의존성이 있음
일단 docker buildkit을 활성화시켜주기위해
1. `/etc/docker/daemon.json` 의 buildkit 세팅을 true로 잡아주고
   ![[Pasted image 20221114184438.png]]
2. 도커 데몬을 재시작한다 : `systemctl restart docker.service`

## 2. Docker buildx 설정
[출처](https://80000coding.oopy.io/54dc871d-30c9-46cb-b609-2e8831541b5e)
1. docker 버전 확인 : 19.03 이후로만 가능
2. `~/.docker/config.json`  수정
```bash
export DOCKER_CLI_EXPERIMENTAL=enabled 
vi ~/.docker/config.json
```
![[Pasted image 20221114190547.png]]
위 같이 json 파일을 설정해준다.
이 설정 해주는게 docker desktop에서 
![[Pasted image 20221114190420.png]]
대충 요 설정 해주는거랑 비슷해보인다
여기까지 다 해줬는데 `docker buildx` 찍어도 설치안돼보였다
이후에 `systemctl status docker.service`를 실행해보니 configuration이 바뀌어서 그걸 확정하라는 어떤 명령어(뭔지는 기억안나네;;)를 치라는 가이드가 있었고, 그걸 해주긴했다.

## 3.buildx 바이너리 추가
윗단계까지 했는데 아무 효과가 없어서 [공식문서](https://docs.docker.com/build/buildx/install/#linux-packages)를 좀 읽어봤고, 문서에서는
1. 엥간하면 docker desktop으로 활성화해라
2. 수동설치 할 수 있긴한데 보장못한다. 테스트용으로만 써라
라고 권장했는데, bento 쓰려면 buildkit 필요하다는데 별수있나... 걍 수동설치 함 해봤다.

설치단계는 생각보다 간단한데
1. [docker buildx 배포 사이트](https://github.com/docker/buildx/releases)에서 적절한 바이너리를 다운받고
2. 서버 OS 에 맞춰 이미 정해진 파일 명으로 바이너리 파일을 넣어라
![[Pasted image 20221114191955.png]]
이다.
![[Pasted image 20221114191809.png]]
내가 시도한 시점 최신 버전은 0점대 버전이었으며, buildx가 stable한 프로젝트가 아님을 알 수 있다.
이제 어떤 바이너리를 받는지가 중요한데, 서버 OS랑 칩 아키텍쳐별로 다 다른가보다.
LCSASMDL 서버는 Intel Xeon Gold(cat /proc/cpuinfo 명령어로 확인)에 Ubuntu를 올려 사용중인데, 칩 아키텍쳐까지는 정확히 몰라서일단 ARM은 모바일 AP니까 그거 제외하고 linux 바이너리에 맞는거 다 넣어보자는 심산으로 했다.
근데 운이좋게도 제일 위 바이너리 `buildx-v0.9.1linux-amd64`가 동작을 했다. 운이 좋았다.

서버에 바로 받는법을 몰라서
1. 로컬에 받고
2. scp로 root권한 필요없는 디렉터리로 복사
```bash
# 로컬
scp buildx-v0.9.1.linux-amd64 lcsasmdl@10.121.117.175:/home/lcsasmdl/다운로드
```
3. buildx 바이너리가 들어가야할 목적지로 복사 : 이 때 파일 명은 `docker-buildx`여야 한다
```bash
# 서버

# cli-plugin 폴더 생성
mkdir cli-plugin

# 다운로드 디렉터리로 이동 후 cli-plugin 폴더로 buildx 바이너리 복사
# cp {복사 대상} {복사 목적지}
# 복사 목적지를 디렉터리로 하면 복사 대상 파일 명 그대로 복사되고
# 복사 목적지를 디렉터리/원하는파일명 으로 하면 원하는 파일 이름으로 복사된다.
cp buildx-v0.9.1.linux-amd64 /home/lcsasmdl/.docker/cli-plugins/docker-buildx
```

의 과정으로 진행했다. 

그리고 다시 `docker buildx`를 진행하니 정상 명령어로 인식했고
![[Pasted image 20221114192522.png]]
anacodna bentoml 환경에서 원하는 이미지를 bentoml 명령어로 빌드했다.
![[Pasted image 20221114192604.png]]