linux 이미지 하나 pull 받아서 그 위에서 kafka 바이너리 테스트
도움받은 문서들
[Confluent Developer Documentaion](https://docs.confluent.io/home/overview.html)
[Confluent blog](https://www.confluent.io/blog/)

# Kafka Broker
## 설치 (broker, connect)
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
update-ca-certificates # ubuntu 용
# 도커 재시작
systemctl restart docker

# 도커 볼륨 생성 : 컨테이너 간 호스트 경로를 공유, 마운트 가능
# docker volume ls : 도커 볼륨 리스트 확인
# docker volume inspect kafka-vol : 도커 볼륨 상세 내용 확인
docker volume create kafka-vol

# docker 컨테이너 생성
docker run -it -p 5005:5005 -p 9092:9092 -d -v kafka-vol:/app --name kafka-broker centos:centos7

# kafka connect 컨테이너 생성 할 땐 아래 명령어
docker run -it -p 8083:8083 -v kafka-vol:/app -d --name kafka-connect centos:centos7

#########################################################
# 도커 컨테이너 안 환경
# jdk 설치
yum install -y java-1.8.0-openjdk-devel.x86_64
# java 설치 확인
java -version

# tcpdump 설치
yum install tcpdump

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

## 설정
```bash
# advertised listener 설정
config/server.properties = 
```
## zookeeper, kafka 실행
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
## zookeeper, kafka 중지
```bash
bin/kafka-server-stop.sh

bin/zookeeper-server-stop.sh
```

## kafka 커맨드라인 툴
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

# 컨슈머
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello.kafka --from-beginning
# key-value 로 데이터 소모
# --group 으로 새로운 컨슈머 그룹 생성
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello.kafka --property print.key=true --property key.separator="-" --group hello-group --from-beginning

# 컨슈머 그룹 리스트
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
# 특정 컨슈머 그룹 상세 현황 확인
# CURRENT-OFFSET : 입력된 데이터 양
# LOG-END-OFFSET : 소모한 데이터 위치
# LAG : 지연. CURRENT-OFFSET - LOG-END-OFFSET
# CONSUMER-ID : 컨슈머의 토픽 할당.
# HOST : 컨슈머가 동작하는 host 명
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group hello-group --describe

# 네트워크 통신 테스트 용 producer, consumer
# producer
bin/kafka-verifiable-producer.sh --bootstrap-server localhost:9092 --max-message 10 --topic verify-test
# consumer
bin/kafka-verifiable-consumer.sh --bootstrap-server localhost:9092 --topic verify-test --group-id test-group

# 토픽 삭제
bin/kafka-topics.sh --delete --bootstrap-server localhost:9092 --topic example_mssql_topic_XSCADA_ALARM_LOG

# 이미 적재된 토픽 데이터 삭제
vi delete-topic.json
bin/kafka-delete-records.sh --bootstrap-server localhost:9092 --offset-json-file delete-topic.json
```
- 토픽 옵션 변경
파티션 변경
![[Pasted image 20221028185655.png]]
retention 변경
![[Pasted image 20221028185952.png]]
![[Pasted image 20221028190149.png]]

## 개념
- 토픽 데이터는 어디에 저장되나요?
`config/server.properties` 의 `log.dir` 에 정의된 위치에 로그파일로 쌓임 (default = tmp/kafka-log)
![[Pasted image 20221117211654.png]]
- 토픽 데이터는 얼마나 오래 보관되나요?
`config/server.properties` 의 `log.retention.hours` 에 정의된 시간만큼 보유 (default = 7일)
![[Pasted image 20221117211803.png]]


# Kafka Connect
[Apache Kafka Connect 공식문서](https://kafka.apache.org/documentation/#connect)
- Confluent Kafka Connect 와는 조금 다르니 위 문서 기반으로 이해할것
- 기본적인 Connector들은 Apache Kafka에 내장
- 추가적인 Connector는 jar 구현체를 plugin 형식으로 Connect에 등록해줘야 하며, 각각 설정이 상이하므로 개별 Connector 의 설정을 따라야한다.

## 분산모드 kafka connect
```bash
# 2대 이상의 connect 서버 
# 각 서버당 1개의 분산모드 커넥트 실행 권장
# connect 컨테이너를 만들고 그 서버 위에 커넥트 프로세스 실행함

# 1. config/connect-distributed.properties 수정
bootstrap.servers=10.121.117.175:9092
key.converter.schemas.enable=true
value.converter.schemas.enable=true
plugin.path=/app/plugins/confluentinc-kafka-connect-jdbc-10.6.0-custom/lib

# 2. connector jar 추가 [[#JDBC Connector]] 를 수행

# 3. distributed kafka connect worker 실행
bin/connect-distributed.sh config/connect-distributed.properties
```

## config/connect-distributed.properties 세팅
초기 세팅 상태는 아래와 같다
```properties
##
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
##

# This file contains some of the configurations for the Kafka Connect distributed worker. This file is intended
# to be used with the examples, and some settings may differ from those used in a production system, especially
# the `bootstrap.servers` and those specifying replication factors.

# A list of host/port pairs to use for establishing the initial connection to the Kafka cluster.
bootstrap.servers=10.121.117.175:9092

# unique name for the cluster, used in forming the Connect cluster group. Note that this must not conflict with consumer group IDs
group.id=connect-cluster

# The converters specify the format of data in Kafka and how to translate it into Connect data. Every Connect user will
# need to configure these based on the format they want their data in when loaded from or stored into Kafka
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
# Converter-specific settings can be passed in by prefixing the Converter's setting with the converter we want to apply
# it to
key.converter.schemas.enable=false
value.converter.schemas.enable=false

# Topic to use for storing offsets. This topic should have many partitions and be replicated and compacted.
# Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
offset.storage.topic=connect-offsets
offset.storage.replication.factor=1
#offset.storage.partitions=25

# Topic to use for storing connector and task configurations; note that this should be a single partition, highly replicated,
# and compacted topic. Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
config.storage.topic=connect-configs
config.storage.replication.factor=1

# Topic to use for storing statuses. This topic can have multiple partitions and should be replicated and compacted.
# Kafka Connect will attempt to create the topic automatically when needed, but you can always manually create
# the topic before starting Kafka Connect if a specific topic configuration is needed.
# Most users will want to use the built-in default replication factor of 3 or in some cases even specify a larger value.
# Since this means there must be at least as many brokers as the maximum replication factor used, we'd like to be able
# to run this example on a single-broker cluster and so here we instead set the replication factor to 1.
status.storage.topic=connect-status
status.storage.replication.factor=1
#status.storage.partitions=5

# Flush much faster than normal, which is useful for testing/debugging
offset.flush.interval.ms=10000

# List of comma-separated URIs the REST API will listen on. The supported protocols are HTTP and HTTPS.
# Specify hostname as 0.0.0.0 to bind to all interfaces.
# Leave hostname empty to bind to default interface.
# Examples of legal listener lists: HTTP://myhost:8083,HTTPS://myhost:8084"
#listeners=HTTP://:8083

# The Hostname & Port that will be given out to other workers to connect to i.e. URLs that are routable from other servers.
# If not set, it uses the value for "listeners" if configured.
#rest.advertised.host.name=
#rest.advertised.port=
#rest.advertised.listener=

# Set to a list of filesystem paths separated by commas (,) to enable class loading isolation for plugins
# (connectors, converters, transformations). The list should consist of top level directories that include
# any combination of:
# a) directories immediately containing jars with plugins and their dependencies
# b) uber-jars with plugins and their dependencies
# c) directories immediately containing the package directory structure of classes of plugins and their dependencies
# Examples:
# plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins,/opt/connectors,
```


### bootstrap.servers

### key.conveter, value.converter
kafak connect가 메시지(key: value) 를 가져올 때
1. 원본 데이터
2. 원본 데이터를 kafka connect 메시지로 직렬화
3. kafka connect 메시지를 kafka topic 메시지로 전달
위 과정을 거친다.
- converter
2번 과정에서 kafka로 들어올때 메시지가 직렬화되는 방식들을 지정해줘야 하며,
기본 세팅의 경우 `org.apache.kafka.connect.json.JsonConverter`를 사용한다.
apache kafka의 경우 `JsonConverter`, `StringConverter`, `ByteArrayConveter` 를 기본제공
- converter.schemas.enable
`true` 로 세팅 시 메시지 직렬화 시 값에 스키마 정보도 가져오는데 보기 좀 까다롭다
`false` 로 세팅 시 row 데이터만 토픽 메시지로 들어와서 보기 편하다
![[Pasted image 20221201120826.png]]
```json
# false로 세팅 시 payload 에 내가 원하는 값들이 들어있다.
{
    "schema":{
        "type":"struct",
        "fields":[{"type":"int64","optional":false,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"otime"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"name":"org.apache.kafka.connect.data.Timestamp","version":1,"field":"last_modified"},{"type":"string","optional":false,"field":"value"},{"type":"string","optional":true,"field":"rawvalue"},{"type":"string","optional":false,"field":"datetime"}],
        "optional":false,
        "name":"tag02_csc_bdp_db1_dbd342"
    }
    ,"payload":{
        "otime":1669862421124
        ,"name":"F5.P.CSC.1.BDP.01.00.P_BDP1.DB1_DBD342"
        ,"last_modified":1669804438725
        ,"value":"186"
        ,"rawvalue":"186"
        ,"datetime":"2022-12-01 11:40:21"
        }
}

# true 로 세팅 시 내가 원하는 값만 토픽 메시지로 들어온다
{"otime":1669862877123
,"name":"F5.P.CSC.1.BDP.01.00.P_BDP1.DB1_DBD342"
,"last_modified":1669804438725
,"value":"186"
,"rawvalue":"186"
,"datetime":"2022-12-01 11:47:57"
}
```

### plugin
apache kafka가 제공하지 않는 connector, convertor, transformation가 있을 경우 다운로드 받고 `plugin.path` 에 해당 플러그인의 경로를 명시해야함

## Connector
개념
- connector
	- kafka-connect 서버에서 source나 sink로의 연결부 jar 구현체
	- jar로 개발된 오픈소스 connector도 있고[(confluent hub)](https://docs.confluent.io/kafka-connectors/self-managed/kafka_connectors.html), 스스로 개발할수도 있다
	- 오픈소스 connector를 사용할땐 플러그인 개념으로 동작하며 `config/connect-distributed.properties` 의 `plugins.path` 에 오픈소스 connector의 jar가 있는 디렉터리 위치를 추가해줘야한다.
	- 설정이 동일하면 하나의 connector 생성
		- ex. LCSASMDL 서버의 postgresql 의 public 스키마 테이블들을 하나의 connector로 생성
	- task
		- connector 하위 개념
		- connector -> kafka-topic 로의 데이터 전달을 해주는 실제 프로세스인듯....
		- 하나의 태스크가 여러 토픽으로 데이터를 전달할 수 도 있고, 여러 태스크가 하나의 토픽으로 데이터를 전달할 수도 있음
		- 즉, connector-task : kafka-topic = 1 : N 일수도 있고, connector-task : kafka-topic = N : 1 일수도 있음


### JDBC Connector
[Confluent JDBC Connector#Overview](https://docs.confluent.io/kafka-connectors/jdbc/current/source-connector/index.html)
JDBC를 이용해 RDB 데이터를 kafka로 이동시킬 때 필요한 커넥터

#### 세팅과정
1. confluent JDBC Connector [다운로드](https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc) [소스코드](https://github.com/confluentinc/kafka-connect-jdbc)
2. ZIP으로 받았을 경우 원하는 경로로 unzip
3. [jdbc connector를 kafka connect의 plugin으로 등록]`{kafka 경로}/config/connect-distributed.properties` 의 `plugin.path` 를 `{confluent kafka connector}/lib` 으로 세팅
![[Pasted image 20221117205158.png]]
위 상황은 confluent JDBC Connector가 `/app/plugins/confluentinc-kafka-connect-jdbc-10.6.0-custom/` 에 압축 풀린상태로 있고, `/app/plugins/confluentinc-kafka-connect-jdbc-10.6.0-custom/lib` 에 JDBC connector가 구현된 jar가 있어 그곳을 kafka connect 프로세스가 바라보도록 함
![[Pasted image 20221117205527.png]]
아래 jdbc는 jdbc connector 구동할때 안쓰이는것 같고, 실제로 사용되는 jdbc들은 다음 단계의 `libs`아래의 jar들을 사용함

4. [jdbc connector 구동에 필요한 jdbc jar 추가] 소스 RDB에 연결가능한 JDBC jar를 `{kafka 경로}/libs/` 하위로 이동
연결에 필요한 JDBC jar는 kafka 바이너리가 있는 경로에다가 옮겨야한다.(주의!!!)
처음에는 jdbc를 confluent kafka connector의 lib에 넣어서 삽질을 좀 했는데 아니고 꼭 apache kafka 바이너리가 있는 경로의 libs에 넣어야한다
나의 경우 kafka-connect 컨테이너의 `/kafka_2.13-3.0.0/libs` 하위에 JDBC jar를 추가했다
![[Pasted image 20221117205855.png]]
5. 분산모드 kafka connect worker 실행
`bin/connect-distributed.sh config/connect-distributed.properties`
6. [어느곳에서든] kafka connect 서버 떴는지 `GET` 메서드로 확인 : `curl -X GET 10.121.117.175:8083`
curl 명령어로 요청하면 아래 사진같을거고
![[Pasted image 20221117205927.png]]
postman에서 확인해도 된다.
7. [어느곳에서든] 적절한 source connector property로 `POST` 수행해 소스 커넥트 등록
![[Pasted image 20221117205009.png]]
샘플
```json
{ "name": "my-source-connect", 
     "config": { 
         "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
         "connection.url": "jdbc:postgresql://10.121.117.175:5432/postgres", 
         "connection.user":"ansung", 
         "connection.password":"ansung",
         "mode":"incrementing",
         "incrementing.column.name" : "datetime", 
         "table.whitelist" : "tag01_csc_bdp_db1_dbd338, tag02_csc_bdp_db1_dbd342, tag03_csc_bdp_db1_dbd346", 
         "topic.prefix" : "lcsasmdl_topic_", 
         "tasks.max" : "1"
    } 
}
```

- `table.blacklist`  `table.whitelist`
`table.blacklist` 나 `table.whitelist` 에 여러 테이블을 나열할땐 컴마로 구분하지만, **띄어쓰기가 들어가면 안된다!!!!!!!!!!!!!!!!!!!!!!!!!!1**
ex. (O) "table.blacklist" : "errors,test_table"                 (X) "table.whitelist" : "errors,test_table" 
- `key.converter`, `value.converter` 
API 호출 시 json body로 `key.converter`, `value.converter`  선언 시 기존 `connect-distributed.proterties` 세팅은 override 된다 ([Apache Kafka Doc#Configuring Connectors](https://kafka.apache.org/documentation/#connect_configuring)])
- `transforms`
	- jdbc connector를 통해 들어오는 메시지를 일차적으로 변형시키는 함수
	- Single Message Transformas(SMTs) 기반 [Confluent - How to User Single Message Transfors in Kafka Connect](https://www.confluent.io/blog/kafka-connect-single-message-transformation-tutorial-with-examples/)
	- 실제로 미리 apache kafka connect로 풀린 클래스를 사용
	- 리스트 선언 순서대로 변환먹으니까 스크립트처럼 생각할것
	1. `transforms` : 변형 항목 리스트 선언
	2. `transforms.변형항목.type` : apache kafka connect 클래스
	3. `transforms.변형항목.___` : type에서 지원하는 항목에 맞춰 작성(공식문서 [Apache Kafka Doc#Configuring Connectors](https://kafka.apache.org/documentation/#connect_configuring)참조)
```json
{ "name": "postgresql-lcsasmdl-ansung_relay-connect", 
	 "config": { 
		 "connector.class":"io.confluent.connect.jdbc.JdbcSourceConnector",
		 "connection.url":"jdbc:postgresql://10.121.117.175:5432/ansung_relay", 
		 "connection.user":"ansung", 
		 "connection.password":"ansung",
		 "db.timezone":"Asia/Seoul",
		 "mode":"timestamp",
		 "timestamp.column.name":"otime", %%NULL값가능 컬럼 입력하면 오류%%
		 "schema.pattern":"public",
		 "table.blacklist":"errors", 
		 "topic.prefix":"ansung_relay_topic_",
		 "transforms":"InsertKey,IncludeFields",
		 "transforms.InsertKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
		 "transforms.InsertKey.fields":"datetime",
		 "transforms.IncludeFields.type":"org.apache.kafka.connect.transforms.ReplaceField$Value",
		 "transforms.IncludeFields.include":"name,rawvalue,datetime"
	} 
}
```

```json
{ "name": "saphana-lcssapdb-material_master-connect", 
	 "config": { 
		 "connector.class":"io.confluent.connect.jdbc.JdbcSourceConnector",
		 "connection.url":"jdbc:sap://10.120.2.230:30015/?autocommit=false", 
		 "connection.user":"SAPABAP1", 
		 "connection.password":"Chilsung123",
		 "db.timezone":"Asia/Seoul",
		 "mode":"bulk",
		 "query":"SELECT * FROM (SELECT ERSDA AS CREATIONDATE, LTRIM(MAKT.MATNR, '0') AS MATNUM, MAKT.MAKTX AS MATNAME, MARA.MATKL AS MATCLSNUM, T.WGBEZ AS MATCLSNAME FROM (SELECT MATNR, MATKL, ERSDA FROM MARA) AS MARA LEFT JOIN (SELECT MATNR, MAKTX FROM MAKT WHERE SPRAS = '3') AS MAKT ON MARA.MATNR = MAKT.MATNR LEFT JOIN (SELECT MATKL, WGBEZ FROM T023T WHERE SPRAS ='3' AND MANDT = '200') AS T ON MARA.MATKL = T.MATKL)",
		 "transforms":"InsertKey",
		 "transforms.InsertKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
		 "transforms.InsertKey.fields":"MATNUM",
		 "poll.interval.ms":10000,
		 "topic.prefix":"sap_material_master_topic"
	} 
}
```

```json
{ "name" : "test-view",
	"config":{
		"connector.class":"io.confluent.connect.jdbc.JdbcSourceConnector",
		"connection.url":"jdbc:sqlserver://10.121.117.158:1433;databaseName=UAPI",
		"connection.user":"UAPIViewer",
		"connection.password":"lcsmes1!",
		"db.timezone":"Asia/Seoul",
		"schema.pattern" : "dbo,",
		"table.types" : "view",
		"mode" : "bulk", # bulk모드에서는whilte,blacklist필요x,
		"query" : "SELECT * FROM VW_CSC_PO_BOM",
		"transforms":"InsertKey",
		"transforms.InsertKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
		"transforms.InsertKey.fields":"DATE",
		"poll.interval.ms":3000,
		"topic.prefix":"test-view"
	}
}
```

#### 커넥터 설정(Configuration Properties)
[Confluent JDBC Connector#Properties](https://docs.confluent.io/kafka-connectors/jdbc/current/source-connector/source_config_options.html#jdbc-source-connector-configuration-properties)
- jdbc.url
![[Pasted image 20221129153354.png]]


#### 참고문서
- 1 ~ 3 과정 :  [이 블로그](https://cjw-awdsd.tistory.com/53) 에서 도움 많이 받았고
- 4 이후 나머지 과정 [이 블로그](https://sup2is.github.io/2020/06/08/kafka-connect-example.html) 에서 도움 받았다
- jdbc 커넥터 세팅 과정 공식 문서 : [Confluent JDBC Connector 공식문서](https://docs.confluent.io/kafka-connectors/jdbc/current/index.html#jdbc-connector-source-and-sink-for-cp) 
- 자세한 내용 블로그 설명 : [Confluent 블로그 : Kafka Connect Deep Dive - JDBC Source Connector](https://www.confluent.io/blog/kafka-connect-deep-dive-jdbc-source-connector/#no-suitable-driver-found)

## REST API
| HTTP 메서드 | 호출경로                            | 설명                   |
| ----------- | ----------------------------------- | ---------------------- |
| POST        | /connectors/{connector이름}/restart | 실행중인 커넥터 재시작 | 

# Kafka Streams
https://kafka.apache.org/documentation/streams/developer-guide/
https://github.com/bbejeck/kafka-streams-in-action
https://github.com/confluentinc/examples
https://www.confluent.io/blog/building-real-time-streaming-etl-pipeline-20-minutes/
https://www.confluent.io/resources/kafka-streams-serialization-deserialization-code-examples/

# Kafka Connect + Kafka Streams
[Confluent - Pipelining with Kafka Connect and Kafka Streams](https://docs.confluent.io/platform/current/tutorials/examples/connect-streams-pipeline/docs/index.html#pipelining-with-kconnect-long-and-kstreams)
## Apache Kafka 지원
1. CLI Producer with String
2. JDBC source connector with JSON

## Apache Kafka 미지원
avro를 Apache Kafka가 지원안하는것같아 관련된 것 미지원
1. JDBC source connector with SpecificAvro
2. JDBC source connector with GenericAvro
3. ...

# Trouble Shooting
## Error connecting to node
1. config/server.properties 의 advertised.listeners 수정
로컬 머신의 ip로 배포해야 클라이언트에서 연결 가능하다
![[Pasted image 20221102171005.png]]
2. kafka 서비스 재기동[[#zookeeper, kafka 중지]] [[#zookeeper, kafka 실행]]

- 네트워크 이슈 없는지 확인 = tcpdump 찍기[출처](https://mkil.tistory.com/482)
1. 머신 ip로 패킷이 들어오는지 확인
2. 도커 컨테이너로 패킷이 들어오는지 확인
```bash
tcpdump host {ip} # 이 아이피로 들어오거나 나가는 패킷 모두 리스닝
tcpdump port {port} # 이 포트로 들어오거나 나간느 패킷 모두 리스닝
tcpdump dst port {port} # 이 포트로 들어오는 패킷 모두 리스닝
```

## 메시지가 안들어감
상황
로컬에서 `bin/kafka-broker-api-versions.sh --bootstrap-server 10.121.117.175:9092`도 오류나고
![[Pasted image 20221102171420.png]]
클라이언트 자바 프로그램에서도 메시지가 안들어감(얘는 인텔리제이 콘솔상 성공이랑 실패 로그는 같은데, 성공일때는 프로세스가 호다닥 끝나고, 실패일때는 타임아웃때까지 프로세스가 실행됨)
근데 토픽 이름 바꾸고 자바프로그램 실행시키면 서버에 토픽은 만들어짐.. 뭐지
[갓플루언트 참고문서](https://www.confluent.io/blog/kafka-client-cannot-connect-to-broker-on-aws-on-docker-etc/)
![[Pasted image 20221102171136.png]]
![[Pasted image 20221102171352.png]]
클라이언트와의 연결은 아래와 같다
![[Pasted image 20221102171609.png]]
![[Pasted image 20221102171855.png]]
![[Pasted image 20221102171946.png]]
계속 연결이 힘들었던건 아래와 같은 이유때문이다
config/server.properties에 선언된 ip가 클라이언트에서는 연결할 수 없는 주소였기때문
![[Pasted image 20221102172025.png]]

삽질하는 동안 `advertised.listener`는 아래와 같았다
![[Pasted image 20221101212444.png]]
`docker inspect {컨테이너}` 일 때 ip는 172.17.0.2였고, 내 생각으로는 도커의 ip를 리턴해줘야 클라이언트에서 연결될 수 있다고 생각한거였다
![[Pasted image 20221102172629.png]]
모식도는 위와 같으며, 클라이언트에서 `10.121.117.175:9092`로 패킷을 보내면, 포트포워딩 된 9092 포트를 통해 카프카 컨테이너로 패킷이 전송되는 구조다. 다만, 머신 내에서 컨테이너를 식별할 수 있는 주소는 `172.17.0.2` 이다.
트러블 슈팅을 위해서 생각한건 아래 순서였다
1. LCSASMDL로 패킷이 가는지
2. 컨테이너 안으로 패킷이 가는지
3. 패킷이 간다면 왜 메시지가 등록 안되는건지 -> 무조건 세팅 잘못이다

1. LCSASMDL로 패킷 가는지 확인
: 머신 위에서 `tcpdump dst port 9092` 로 모니터링하니 `bin/kafka-broker-api-versions.sh --bootstrap-server 10.121.117.175:9092` 실행 시 반응 있다 = 서버로 패킷 간다
 2. 컨테이너 안으로 패킷이 가는지
: 컨테이너 안에서 `tcpdump dst port 9092` 로 모니터링 + `bin/kafka-broker-api-versions.sh --bootstrap-server 10.121.117.175:9092` 실행시 반응 있다 = 컨테이너로 패킷 간다
 3. 세팅 뭐가 잘못이지?
	 1. 모든 문서봐도 클라이언트로의 연결은 `advertised.listener` 가 정한다고 말함
	 2. 와중에 갓플루언트 설명 찾음
`advertised.listener` 가 잘못 설정되면 아래와 같은 상황이 펼쳐진다
대표적으로, `advertised.listener` 를 세팅하지 않고 클라이언트를 붙일경우 브로커는 `localhost:9092` 를 리턴하고, 클라이언트는 루프백으로 자신 안에서 9092 포트 서비스를 찾게된다.
![[Pasted image 20221102173208.png]]
![[Pasted image 20221102173216.png]]
![[Pasted image 20221102173226.png]]
이처럼, 나는 `advertised.listener`를 컨테이너의 ip인 `172.17.0.2:9092`로 설정해놨고, 클라이언트 연결 시 브로커 주소를 연결가능한 주소인 LCSASMDL의 주소 `10.121.117.175:9092`이 아니라 `172.17.0.2:9092` 로 받게되므로 연결, 메시지를 등록하지 못했던것이다.

그래서 결국 내가 해줘야 할 거는 `advertised.listener`를 머신 ip인 `10.121.117.175:9092`로 세팅해주고 카프카를 띄우면 되는거였다.
그러니까
1. 카프카 메타데이터 요청도 잘 됐고
![[Pasted image 20221102173654.png]]
2. 클라이언트 프로그램에서 메시지도 잘 들어갔다 (성공이나 실패시 로그는 같지만, 성공하면 프로세스가 호다닥 끝난다)
![[Pasted image 20221102173745.png]]



# etc
## PostgreSQL(중계 DB) 세팅
[[PostgreSQL#세팅]]

## Crontab 세팅
[[linux 일반#crontab]]

