1. gitlab web 프로젝트 생성
2. 서버에 gilab-runner 설치
3. gitlab web에 runner 등록
	1. Settings > CI/CD > expand
	2. URL, token 복사
	3. 서버의 gilab-runner에 git 레포 등록 
	```bash
	sudo gitlab-runner register
```
4. 프로젝트 최상위경로에 