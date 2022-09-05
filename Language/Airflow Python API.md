# PythonOperator
## 템플릿 변수와 Python Callable의 kwargs 호출

### 1. Operator안에서 (진자 aka. jinja) 템플릿 변수 사용하기
진자 템플릿화된 컨텍스트 변수를 사용할 수 있다 (airflow engine이 변환 수행해준대)
- 기본적으로 {{}} 로 감싸서(jinja 템플릿) 사용하면되고
```python
fetch_events=BashOperator(
	task_id="fetch_events",
	bash_command=(
		"mkdir -p /data &&"
		"curl -o /data/events.json"
		"http://localhost:5000/events?"
		"start_date={{execution_date.srftime('%Y-%m-%d')}}"
		"&end_date={{next_execution_date.strftime('%Y-%m_%d')}}"
	)
)
```
- custom context 변수에도 사용할 수 있다
```python
calculate_stats=PythonOperator(
	task_id='calculate_stats',
	python_callable=_calculate_stats,
	templates_dict={
		"input_path": "/data/events/{{ds}}.json",
		"output_path": "/data/stats/{{ds}}.csv"
	}
)
```
진자 템플릿 변수는 렌더링돼 실제 구동시 사용된다

템플릿화 가능한 기본 컨텍스트 변수는 아래와 같다

![[Pasted image 20220901175339.png]]
![[Pasted image 20220901175357.png]]
![[Pasted image 20220901175410.png]]


### 2. Python Callable 원하는 변수 주입

DAG에서 PythonOperator 태스크가 아래같이 설정돼있다면
```python
# 제품 재고이동

task_whsinfo010 = PythonOperator(

    task_id = 'ETL_whsinfo010',

    python_callable = whsinfo010,

    params={'sourceDB' : CONFIG[OPERENV]['source'],

            'destinationDB' : CONFIG[OPERENV]['destination'],

            'loadType':EXTMODE},

    provide_context=True,

    dag = dag

)
```

Python Callable에서는 아래와 같이 사용한다
`**kwargs`라던지 `**context` 라던지 변수명은 중요하지 않고 keyword argument들을 언패킹할수있는 변수만 callable의 파라미터로 넣어주고 callable안에서 사용하면 된다.
```python

def whsinfo010(**context):

    loadType = context['params']['loadType']
    tiberoDB = context['params']['sourceDB']
    hanaDB = context['params']['destinationDB']
   ...
```
사실 얘네들은 태스크 속성(task attribute)라 airflow GUI에서 확인 가능하다

GUI로 들어가서
![[Pasted image 20220901180142.png]]
Task Attributes를 확인하면
![[Pasted image 20220901180204.png]]
![[Pasted image 20220901180249.png]]
이런 식으로 Task Attribute 하위에 내가 선언한 변수가 딕셔너리로 물려있음을 알 수 있다.

### 3. 왜그럴까 (소스코드 까보기)
Python Operator 정의는 아래같다 ([출처](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/operators/python/index.html?highlight=template_fields#airflow.operators.python.PythonOperator.template_fields))
![[Pasted image 20220901180509.png]]
Python Operator 소스를 보면 아래처럼 흘러서, 합해진 딕셔너리 변수가 pytho  callable 함수로 전달된다
![[Pasted image 20220901181026.png]]
	모든 operator들은(python operator까지 포함해서) __baseoperator__를 상속하니 더 깊은 내용을 알고싶으면 baseoperator를 까보시길 추천한다 ([소스코드](https://airflow.apache.org/docs/apache-airflow/stable/_modules/airflow/models/baseoperator.html#ScheduleInterval))
	