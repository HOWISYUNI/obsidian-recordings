![[Pasted image 20220901181329.png]]
## 1. Client : ssh key 생성
1. `{home}/.ssh/`로 이동
```bash
$ ssh-keygen -t rsa
```
2. ssh key 생성
	- __*id_rsa.pub*__ : 공개키. authorized_keys에 추가돼야함
	- __*id_rsa*__ : 개인키

## 2. Server : client 공개키를 서버 *authorized_keys*에 등록
편한 에디터를 이용해 추가

## 3. _known_hosts_
client - server sftp 최초 연결 시 `{home}/.ssh/known_hosts`에 자동 등록