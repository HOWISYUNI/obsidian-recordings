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
# java 설치 확인
java -version

# wget 설치
yum install wget

# kafka binary 설치 https://kafka.apache.org/downloads
wget https://archive.apache.org/dist/kafka/3.0.0/kafka_2.13-3.0.0.tgz
# 책엔 이 주소(wget https://archive.apache.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz)로 하니까 zookeeper 가 안떠서 2.13 으로 설치

# kafka 압축 해제
tar xzf kafka_2.13-3.0.0.tgz
# 책엔 2.12 버전 tar xvf kafka_2.12-2.5.0.tgz
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
# zookeeper, kafka 실행
```bash
# 실행 안될 경우 -daemon 빼고 실행시켜서 에러메시지 확인
# zookeeper 실행
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
# zookeeper 실행 확인
jps -vm

# kafka 브로커 실행
bin/kafka-server-start.sh -daemon config/server.properties
# kafka 실행 확인
jps -m
```

# kafka 커맨드라인 툴
```bash
# 토픽 생성 (필수 옵션만 입력)
# --bootstrap-server : 기존 zookeeper를 이용한 통신(kafka 2.2까지)에서 탈피해 카프카와 직접 통신
# partition : 최소 1개
# replication : 최소 1개 ~ 최대 클러스터 브로커 개수
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic hello.kafka --partitions 1 --replication-factor 1

# 토픽 리스트 조회 : --list 옵션
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

# 토픽 상세 조회 : --describe 옵션
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic hello.kafka
# 설정 조회
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name hello.kafka --describe

# 토픽 옵션 수정
# partition 개수 변경 : kafka-topics.sh
bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic hello.kafka --alter --partitions 4

# 나머지 변경 : kafka-config.sh 86400000ms는 하루
/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name hello.kafka --alter --add-config retention.ms=86400000

# 레코드(데이터. key:value로 생성) 넣기
# 키가 null인 데이터로 생성
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic hello.kafka

# key:value로 데이터를 생성
# separator로 : 를 지정. default는 \t(탭)
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic hello.kafka --property "parse.key=true" --property "key.separator=:"
```
- 토픽 옵션 변경
파티션 변경
![[Pasted image 20221028185655.png]]
retention 변경
![[Pasted image 20221028185952.png]]
![[Pasted image 20221028190149.png]]
