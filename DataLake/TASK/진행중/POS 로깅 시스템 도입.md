# to-do list
- 2022-08-18
	1. airflow 컴포넌트 별 airflow.cfg 의 auth_backend 수정
	```
	auth_backend = airflow.api.auth.backend.basic_auth
	```
	2. [airflow api](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html#tag/DAG)postman api 테스트
# 설계
- 목적 : 추출, 변환, 적재 데이터 성공 여부 및 카운트 모니터링
- 브레인스토밍
	1.  화면에서 어떻게 데이터를 볼건지
	2.  로깅 포맷
		1. 적재, 집계 데이터를 유추할만한 유의미한 데이터를 어떻게 남길지
		2. airflow 메타디비 활용방안 있는지
		3. 전체 태스크 결과를 노가다 뛸건지
	3.  어디에 데이터를 남길건지
		1. 후보군
			1. HFDS + HIVE
			2. RDB : 로그 양이 HDFS에 넣을만한 양인지
		2. ISSUE
			1. 수기 업로드 데이터는 어떻게 할건지

- 아이디어
1.  데이터 소스
	1. airflow 웹서버 로깅 데이터
		1. 소스
			- Meta Datasource 에 api 질의
				-   [Airflow API using python code](https://www.astronomer.io/guides/airflow-database/)
				-   [Access the Airflow DB](https://docs.astronomer.io/software/access-airflow-database)
			- 로그 파일
				- 위치 : [lotte@CSDLLZ01] /data/infrastructure/airflow_server/logs/{DAG}/{task}/{retry_num}.log
		1. 의사 결정 필요
			2. 수집주기
				- 리얼타임 : 태스크별로 로깅을 바로 어느 통에 때려버릴 수 밖에 없음
				- 세미 리얼타임 : 15분 ~ 30분
				- 배치 : 1 ~  3 시간 단위
			3.  파싱
			4.  필요 상태 값
			5.  파싱 가능한 헤더 포맷
	2. 태스크 내 로깅 코드라인추가
		1. 
2.   툴
	1. 또어플로우? : 로그 데이터 자주 수집하면 airflow 태스크 병렬 실행에 영향줄듯?
	2. 크론탭 : 쉘스크립트 공부

3.  머신 : 분석노드
	- airflow 쓸거면 : worker를 분석노드에서
	- 크론탭 쓸거면 : 분석노드에서
- 

4.  포맷
	1.  필요조건
		- 파싱 가능한 헤더를 남겨야한다
		- 성공, 실패 시 모두 남아야한다
		- kst 기준 타임스탬프를 남겨야한다
	2.  어디에 쌓을건가?
	3.  데이터 특성
		- 메트릭(지표) 성 데이터가 있나? -> 쌓고 그라파나 시각화까지
		- 메트릭성 없으면 table 형태까지만