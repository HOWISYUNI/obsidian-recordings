### 특징
1. 인터프리터식 언어 = line by line
2. 객체지향형 언어
3. 동적타입핑 대화형 언어  = 프로그램 실행 시 동적으로 타입이 정해짐(type evaluation)

## 네이밍 룰
| X                | O                        |
| ---------------- | ------------------------ |
| 숫자로 시작      | 소문자로 시작            |
| 키워드 사용 불가 | 단어 연결은 언더바 ( _ ) |
# native python functions
| 함수                    | 설명                                                                                                  | 사용법                                              |
| ----------------------- | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| eval(string Expression) | string type의 [표현식](https://docs.python.org/ko/3/reference/expressions.html)을 실행시켜버리는 함수 | eval('1') , eval('1+2'), _eval('function name ()')_ |
| dir(class name)         | 클래스 특성, 함수를 리스트로 출력                                                                     |                                                     |
| help(class name)        | 클래스 사용 방법. type(class name) 의 값을 help에 입력해 보면 편함                                    | class(type(객체))                                   |
|                         |                                                                                                       |                                                     |
|                         |                                                                                                       |                                                     |

## 클래스와 상속
### 클래스
### 왜 써야돼?
1. 계산기를 쉽게 만들어내면서
2. 각자의 속성은 보호하면서(캡슐화), 어떤거는 공유(클래스 속성)
3. 동작을 재활용하도록(상속, 오버라이드)

### 객체 vs 인스턴스
관점상 표현의 차이
```python
a = Cookie()
```
1. 객체 : `a는 객체`
2. 인스턴스 : `a는 Cookie 클래스의 인스턴스`
## self
생성된 객체 나를 가리킨다
`나 객체` 에 할당된 속성들의 의미를 강조하려고 self 라는 매개변수로 받는게 컨벤션이다.
![[Pasted image 20221018101510.png]]
### 속성 attribute
1. 인스턴스 속성 : 인스턴스만 사용하는 메모리
2. 클래스 속성 : 클래스, 인스턴스가 메모리 공유
```python
class Base():
	class_attr = 'CLASS ATTR' # 클래스, 인스턴스가 공유하는 메모리

	def __init___():
		instance_attr = "INSTANCE ATTR" # 인스턴스에만 할당되는 메모리
```

## Decorator
https://engineer-mole.tistory.com/181

# 빅데이터를 위한 파이썬
https://pnavaro.github.io/big-data/14-FileFormats.html



# 언패킹
## 딕셔너리 언패킹
![[Pasted image 20221004154052.png]]
## 리스트 언패킹
![[Pasted image 20221004153742.png]]







