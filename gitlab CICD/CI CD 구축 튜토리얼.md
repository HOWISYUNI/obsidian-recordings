1. gitlab web 프로젝트 생성
2. 서버에 gilab-runner 설치
3. gitlab web에 runner 등록
	1. Settings > CI/CD > expand
	2. URL, token 복사
	3. 서버의 gitlab-runner에 git 레포 등록 
	```bash
	sudo gitlab-runner register
	```
	4. gitlab-runner 리스트 확인
	```bash
	gitlab-runner list
	```
4. 프로젝트 최상위경로에 `.gitlab-ci.yaml` 파일 생성 후 내용 작성 ([공식문서](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html))
	```YAML
	stage: # 파이프라인 동작 순서 정의
		- build
		- test
	
	# job 정의
	build:
		stage: build # stages에서 인식하는 이름
		script:      # 이 job 실행 시 동작해야하는 로직
			- ~~~
			- ~~~
	test:
		stage: test
		script:
			- ~~~
			- ~~~
	```
	
5. git 레포에 push 하면 CI/CD 파이프라인 자동 동작