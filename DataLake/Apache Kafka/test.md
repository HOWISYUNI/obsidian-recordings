linux 이미지 하나 pull 받아서 그 위에서 kafka 바이너리 테스트

# 설치
명령어 흐름
```bash
# DNS 설정 : 아래로 들어가서 search lottechilsung.co.kr 추가
vi /etc/resolv.conf

# docker 이미지 생성
docker pull centos:centos7 # centos8은 yum 이슈로 패키지 다운로드 불가

# docker pull 오류시, openssl로 docker.io:443 사이트의 ssl 인증서 가져옴
# 아래 커맨드 해석 : openssl의 s_client를 이용해 docker.io:443에 연결한 뒤 ssl 인증서를 cacert.pem 파일로 가져온다
openssl s_client -showcerts -servername "docker.io" -connect docker.io:443 > cacert.pem
# 인증서 업데이트 해주고
update-ca-certificates
# 도커 재시작
systemctl restart docker

# docker 컨테이너 생성
docker run -it -p 5005:5005 --name centos centos:centos7

#########################################################
# 도커 컨테이너 안 환경
# jdk 설치
yum install -y java-1.8.0-openjdk-devel.x86_64

# wget 설치
yum install wget

# kafka binary 설치 https://kafka.apache.org/downloads
wget https://archive.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz

# kafka 압축 해제
tar xvf kafka_2.12-2.5.0.tgz
```

만약 docker pull이 안된다면 보안에서 막거나 ssl 인증서가 없어서 그럴수도 있음
docker pull 실패
![[Pasted image 20221026174459.png]]
certificate 이 없으므로 openssl을 통해 docker hub의 ssl 가져옴
그럼 openssl로 docker.io의 ssl 인증서 가져오면 되는데
![[Pasted image 20221026175921.png]]
이렇게 소만사에서 막고있으면 웹키퍼쪽에서 해당 url의 인터넷 통신을 막고있는거니까 왕규선배랑 잘 이야기해서 웹키퍼 예외정책 먹이면 됨

dns 세팅은 resolv.conf에서 수행[resolv.conf 참고](https://it-serial.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-DNS-%EC%84%A4%EC%A0%95-%EB%B2%88%EC%99%B8%ED%8E%B8-resolvconf-%ED%8C%8C%EC%9D%BC%EC%B4%88%EA%B8%B0%ED%99%94-%ED%98%84%EC%83%81-%EC%84%A4%EB%AA%85)

완료됐다면
![[Pasted image 20221028152327.png]]

# kafka 설정
