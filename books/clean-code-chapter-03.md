# [Chapter .3] 좋은 코드의 일반적인 특징

클린코드의 궁극적인 목표는 **코드를 가능한 견고하고 결함을 최소화하고 완전히 자명하도록 하는 것**이다.

코드는 디자인을 가장 상세히 보여주는 표현이다.

따라서 소프트웨어 디자인을 위한 몇 가지 원칙을 살펴보자.

_(단, 항상 모든 것을 적용해야하는 것은 아니다)_



## 계약에 의한 디자인

`계약에 의한 디자인(Design by Contract)`이란 관계자가 기대하는 바를 암묵적으로 코드에 삽입하는 대신   

양측이 동의하는 계약을 먼저 한 다음, 계약을 어겼을 경우 명시적으로 왜 계속할 수 없는지 예외를 발생시키라는 것이다.  

즉, 계약을 정의함으로써 런타임 시 오류가 발생했을 때 코드의 어떤 부분이 손상되었는지(계약이 파손되었는지) 명확히 알 수 있게 된다.  

여기서는 소프트웨어 컴포넌트 간 통신 중 지켜야할 몇 가지 규칙을 강제하는 것을 계약이라 한다.

계약에는 `사전조건(precondition)`, `사후조건(postcondition)`, `불변식(invariant)`, `부작용(side-effect)`을 기술하지만,   

코드레벨에서는  `사전조건(precondition)`, `사후조건(postcondition)`만 강제한다.



**계약에 의한 디자인의 단점(어려운점)**

- 애플리케이션의 핵심 논리뿐만 아니라 계약을 작성해야하므로 추가작업이 발생
- 계약에 대한 단위테스트 추가 가능성



**계약에 의한 디자인의 장점**

- 사전조건 혹은 사후조건 검증에서 실패하는 오류가 발생했을 때 오류를 쉽게 찾아낼 수 있음

- 잘못된 가정 하에 코드의 핵심 부분이 실행되는 것을 방지

- 책임의 한계를 명확히 할 수 있음
- 장기적인 품질 보장



### 사전조건(precondition)

함수나 메서드가 제대로 동작하기 위해 보장해야하는 모든 것을 의미하며  

파라미터에 제공된 데이터 유효성 검사하는 계약이다.  

호출자에게 부과된 임무로, 클라이언트는 코드를 실행하기 위해 사전에 약속한 조건을 준수해야한다.  

사전 조건 검증에서 실패했다는 것은 클라이언트에 결함이 있다는 것이다.



### 사후조건(postcondition)

메서드 또는 함수 반환값의 유효성을 검사하여 반환된 후의 상태를 강제하는 계약이다.  

컴포넌트는 클라이언트가 확인하고 강제할 수 있는 값을 보장해야하며  

사후검증을 통과한 클라이언트는 반환된 객체를 아무 문제없이 사용할 수 있어야한다.  

사후 조건 검증에서 실패했다는 것은 특정 모듈이나 제공 클래스 자체에 결함이 있다는 것이다.  



### 파이썬스러운 계약

파이썬스러운 방법은 메서드, 함수, 클래스에 RuntimeError 예외 또는 ValueError 예외를 발생시키는 제어 메커니즘 추가하는 것이며  

문제를 정확히 특정하기 어려울 때는 사용자 정의 예외를 만드는 것이 좋은 방법이다.  

또한 사전조건에 대한 검사, 사후조건에 대한 검사, 핵심 기능에 대한 구현을 구분하는 것이 좋다.  

~~(너무 일반적인 말들만..)~~



## 방어적(defensive) 프로그래밍

방어적 프로그래밍은 `예상할 수 있는 시나리오의 오류를 처리하는 방법` 과 `발생하지 않아야 하는 오류를 처리하는 방법`에 대한 것이다.  



### 에러 핸들링

오류가 발생하기 쉬운 상황에서 사용하며 일반적으로 데이터 입력 확인 시 자주 사용된다.  

에러 핸들링의 목적은 예상되는 에러에 대해 실행을 계속할 수 있을지 아니면 프로그램을 중단할지를 결정하는 것이다.  

에러 처리 방법의 일부로 `값 대체`, `에러 로깅`, `예외 처리`에 대해 알아보자.  



#### 값 대체

일부 시나리오에 오류가 있어 소프트웨어가 잘못된 값을 생성하거나 전체가 종료될 위험이 있을 때 결과값을 안전한 다른 값으로 대체하는 방법이다.  

이 때 잘못된 결과는 정합성을 깨지 않는 다른 값으로 대체해야한다.  

그러나 이 방법은 대체 값이 안전한 옵션인 경우에 한해서만 사용할 수 있어 항상 가능하지는 않다.  

견고성과 정확성 간의 트레이드오프가 발생한다.  

> 견고성: 프로그램이 실패하지 않는 것
>
> 정확성: 부정확한 결과를 내보내는 것(민감하고 중요한 정보를 다루는 경우 특히 중요)

제공되지 않은 값에 대해 기본값을 사용하는 방법도 있다.



#### 예외 처리

입력이 잘못되었다기보다는 외부 컴포넌트에 관련한 문제로 함수 호출에 실패할 수 있는데,   

이런 경우 함수는 오류에 대해 명확하게 알려주는 예외 메커니즘을 사용해야한다.  

예외 처리의 유의점과 권장사항에 대해 살펴보자.  



**예외 처리 유의점**

1. 예외를 사용하여 시나리오나 비지니스 로직을 처리하지 않도록 해야한다.

   : 프로그램의 흐름을 읽기 어려워진다.(캡슐화가 약해짐)  

2. 프로그램이 꼭 처리해야하는 예외적인 비지니스 로직을 except와 혼합하지 않도록 해야한다.

   : 유지보수가 필요한 핵심 논리와 오류를 구분하기 어려워진다.  

3. 너무 많은 예외를 발생시키지 않도록 해야한다.

   : 호출자가 문맥에서 자유롭지 않게 되며 문맥 유지가 어렵다.  



**예외 처리 권장사항**

1. 올바른 수준의 추상화 단계에서 예외 처리를 해야한다.

   : 함수가 처리하는 예외는 캡슐화된 로직과 일치해야한다.  

   ```python
   # deliver_event 메서드에 중점을 두고 살펴보자
   class DataTransport:
       """다른 레벨에서 예외를 처리하는 객체의 예"""
       
       retry_threshold: int = 5
       retry_n_times: int = 3
       
       def __init__(self, connector):
           self._connector = connector
           self.connection = None
           
       def deliver_event(self, event):
           try:
               self.connect()
               data = event.decode()
               self.send(data)
           except ConnectionError as e:
               logger.info(f"연결 실패: {e}")
               raise
           except ValueError as e:
               logger.error(f"{event} 잘못된 데이터 포함: {e}")
               raise
       def connect(self):
           for _ in range(self.retry_n_times):
               try:
                   self.connection = self._connector.connect()
               except ConnectionError as e:
                   logger.info(
                   	f"{e}: 새로운 연결 시도 {self.retry_threshold}"
                   )
                   time.sleep(self.retry_threshold)
               else:
                   return self.connection
           raise ConnectionError(
           	f"{self.retry_n_times} 번째 재시도 연결 실패"
           )
       def send(self, data):
           return self.connection.send(data)
   ```

    `ValueError`와 `ConnectionError`는 책임이 분산되어야하는 오류이나 현재 같은 레벨에서 추상화되고 있다.  

   따라서 `ConnectionError`를 connect 메서드 내에서 처리하여 명확히 분리할 수 있다.  

   뿐만 아니라 `ValueError`는 event를 decode하는 메서드에 속한 에러이므로 deliver_event 메서드와 연관이 없다.  

   따라서 deliver_event 메서드를 다음과 같이 분리하는 것이 더 좋다.  

   ```python
   def connect_with_retry(connector, retry_n_times, retry_threshold=5):
       """connector의 연결을 맺는다. <retry_n_times>회 재시도.
       
       연결에 성공하면 connection 객체 반환
       재시도까지 모두 실패하면 ConnectionError 발생
       
       :param connector: '.connect()' 메서드를 가진 객체
       :param retry_n_times int: ''connector.connect()''를 호출 시도하는 횟수
       :param retry_threshold int: 재시도 사이의 간격
       """
       
       for _ in range(retry_n_times):
           try:
               return connector.connect()
           except ConnectionError as e:
               logger.info(
               	f"{e}: 새로운 연결 시도 {retry_threshold}"
               )
               time.sleep(retry_threshold)
               
       exc = ConnectionError(f"{retry_n_times} 번째 재시도 연결 실패")
       logger.exception(exc)
       raise exc
   
   class DataTransport:
       """추상화 수준에 따른 예외 분리를 한 객체의 예"""
       
       retry_threshold: int = 5
       retry_n_times: int = 3
       
       def __init__(self, connector):
           self._connector = connector
           self.connection = None
           
       def deliver_event(self, event):
           self.connection = connect_with_retry(
           	self._connector, self.retry_n_times, self.retry_threshold
           )
           self.send(event)
           
       def send(self, data):
           try:
               return self.connection.send(event.decode())
           except ValueError as e:
               logger.error(f"{event} 잘못된 데이터 포함: {e}")
               raise
   ```

   

2. Traceback 노출을 하지 않아야 한다.

   : traceback에 포함된 다양한 디버깅 정보는 악의적 사용자에게도 유용한 정보이므로 지적 재산 유출이 발생할 위험이 있다.  

   따라서 예외가 있는 경우 효율적 문제 해결을 위해 traceback정보나 기타 수집가능한 정보를 로그로 남기되,  

   사용자에게 해당 정보를 보이지 말고 일반적인 메세지를 사용하는 것이 좋다.  

   (일반적인 메세지: 페이지를 찾을 수 없다 등)

     

3. 비어있는 except 블록을 지양해야한다.

   : 너무 방어적이어서 실패해야 할 경우에도 실패하지 않게 된다.   

   파이썬의 안티패턴 중 가장 악마같은 경우이며, 파이썬의 철학 중 `에러는 결코 조용히 전달되어서는 안된다.`는 것과도 위배되는 파이썬스럽지 못한 코드이다.

   ~~(아무래도 화가 잔뜩 난 것을 보니 글 쓴 사람이 여기에 많이 데인 듯 하다.)~~

   보다 구체적인 예외를 사용하고 except 블록에서 실제 오류를 처리하는 것이 좋다.  

   구체적 예외는 사용자는 무엇을 기대하는지 알게 되어 유지보수가 쉬우며 버그판단도 쉽다.   

   또한 except 블록에서 자체 예외 처리(예외로깅 등), 기본 값 반환  등으로 처리하는 것도 방법이다.  

   ```python
   try:
       process_data()
   except:
       pass
   ```

     

4. 원본 예외를 포함해야한다.

   : 파이썬에서는 다른 오류를 발생시키고 메세지를 변경할 수 있는데, 이 때 원래 예외를 포함하는 것이 좋다.

   ```python
   class InternalDataError(Exception):
       """업무 도메인 데이터의 예외"""
   
   def process(data_dictionary, record_id):
       try:
           return data_dictionary[record_id]
       except KeyError as e:
           raise InternalDataError("Record not present") from e
   ```

   

### 파이썬에서 어설션 사용하기

어설션은 절대로 일어나지 않아야하는 상황에 사용한다.  

이 상태가 된다는 것은 **소프트웨어에 결함이 있음**을 의미하는 것과 같다.  

에러 핸들링과의 차이점은 프로그램을 중지하는 것이 좋다는 것이며, 해당 결함을 수정하여 새 버전의 배포를 하는 것 외에 다른 방법이 없다.  



예제로 어설션을 사용할 때 주의해야할 점을 알아보자.  

```python
try:
    assert condition.holds(), "조건에 맞지 않음."
except AssertionError:
    alternative_procedure()
```

1. 어설션을 비지니스 로직과 섞거나 제어 흐름 메커니즘으로 사용해서는 안된다.

   : 잘못된 시나리오에 도달할 경우 프로그램이 피해를 입지 않도록 하거나 중단시키는 용도이기 때문

     

2. 어설션 문장은 함수로 구성해서는 안된다.

   : 함수호출은 부작용이 있을 수 있으며, 항상 반복가능하지 않을 수 있고 디버거 사용시에도 결과를 편리하게 볼 수 없음  

  

다음과 같이 변경하여 작성하는 것이 보다 유용하다.  

```python
result = condition.holds()
assert result > 0, f"에러 {result}"
```



## 관심사의 분리

저수준의 디자인(코드)뿐만 아니라 더 높은 수준의 추상화와도 관련있는 내용으로  

~~나중에 Erin님이 마지막 장 클린 아키텍처에서 멋지게 마저 설명할 예정이다~~  

~~(계획표 짠 사람 나와)~~   

  

소프트웨어에서 관심사를 분리하는 목표는 **파급효과를 최소화하여 유지보수성을 향상시키는 것**이다.  

책임이 다르면 컴포넌트, 계층, 모듈 등으로 분리되어야한다.  

프로그램의 각 부분은 기능의 일부분(관심사)에 대해서만 책임을 지며 나머지 부분에 대해서는 알 필요가 없다.  

각 관심사가 계약에 의해 시행될 수 있다는 점에서 DbC원칙과 비슷하나  

관심사의 분리는 일반적으로 함수, 메서드, 클래스 간의 계약이라기보단 모듈, 패키지, 컴포넌트에 대해 적용된다는 차이점이 있다.  



### 응집력(cohension)과 결합력(coupling)

소프트웨어 설계에 있어 중요한 개념이다.  

잘 정의된 소프트웨어는 `높은 응집력과 낮은 결합력`을 갖는다.  



**응집력**  

객체가 작고 잘 정의된 목적을 가져야하며 가능한 작아야한다는 것을 의미한다.  

객체의 응집력이 높을수록 더 유용하고 재사용성이 높아지므로 더 좋은 디자인이라고 할 수 있다.  



**결합력**   

두 개 이상의 객체가 서로 어떻게 의존하는지를 나타내며, 의존에 따른 종속성은 제한을 의미한다.    

객체나 메서드의 어떤 부분이 너무 의존적이라면 다음과 같은 문제가 생길 수 있다.  

- 낮은 재사용성: 함수가 객체에 결합되면 다른 상황에서 해당 함수를 사용하기 매우 어려움
- 파급(ripple) 효과: 두 부분 중 하나를 변경하면 다른 부분에도 영향을 미침
- 낮은 수준의 추상화: 서로 다른 추상화 레벨에서 문제 해결이 어려움  



## 개발 지침 약어

좋은 소프트웨어 관행을 약어로 소개한다.  

~~정식 학술 용어는 아니며 경험, 논문, 책에 있는 내용들임~~   



### DRY/OAOO

> DRY: Do not Repeat Yourself
>
> OAOO: Once and Only Once

코드에 있는 지식은 단 한 번, 단 한 곳에 정의되어야 함을 의미한다.  

따라서 코드를 변경하려고 할 때 수정이 필요한 곳은 단 한 곳이어야 한다.



코드 중복은 다음과 같은 부정적인 영향이 있다.

- 오류가 발생하기 쉬움: 여러 번 반복된 로직이 수정되어야할 때 인스턴스 하나라도 빠뜨리면 버그 발생
- 비용이 비쌈: 변경 시 개발, 테스트에 더 많은 시간이 소요되어 전체 개발 속도를 느리게 함
- 신뢰성이 떨어짐: 사람이 직접 모든 인스턴스 위치를 기억해야하며 단일 데이터 소스가 아니어서 완결성이 떨어짐



중복을 제거하는 방법은 다음과 같은 것들이 있다.

- 함수 생성: 가장 간단한 방법
- 완전히 새로운 객체 생성: 전체적으로 추상화를 하지 않은 경우에 사용가능한 방법
- 컨텍스트 관리자 사용
- 이터레이터나 제너레이터 사용
- 데코레이터 사용

_(파이썬에서는 어떤 방법이 가장 적합한지 알려주는 규칙이나 패턴은 없다)_



### YAGNI

> YAGNI: You Ain't Gonna Need It

미래 보장성이 높기를 바라며 과잉 엔지니어링을 하지 않아야함을 의미한다.  

유지보수가 가능한 소프트웨어를 만드는 것은 **현재의 요구사항을 잘 해결하기 위한 소프트웨어를 작성**하고 가능한**나중에 수정하기 쉽도록 작성**하는 것이다.  

미래의 모든 요구사항을 고려할 경우 복잡하고 추상화로 인해 코드 읽기 및 유지보수가 어려우며 이해도 어렵다.



### KIS

> KIS: Keep It Simple

소프트웨어 컴포넌트를 설계할 때 과잉 엔지니어링을 피하고 문제에 적합한 최소한의 솔루션인지 자문해야함을 의미한다.  

디자인이 단순할수록 유지 관리가 쉽다.  

높은 수준의 디자인을 할 때에도 특정 코드라인을 디자인할 때 염두에 두어야한다.  

단순한 것이 복잡한 것보다 낫다는 파이썬의 철학도 기억하자.



아래 예제를 보자.  

```python
# 파이썬스럽지 않은 방식
class ComplicatedNamespace:
    """프로퍼티를 가진 객체를 초기화하는 복잡한 예제"""
    
    ACCEPTED_VALUES = ("id_", "user", "location")
    
    @classmethod
    def init_with_data(cls, **data):
        instance = cls()
        for key, value in data.items():
            if key in cls.ACCEPTED_VALUES:
                setattr(instance, key, value)
        return instance
    
>>> cn = ComplicatedNAmespace.init_with_data(
		  id_=42, user="root", location="127.0.0.1", extra="excluded"
	)
>>> cn.id_, cn.user, cn.location
(42, 'root', '127.0.0.1')
>>> hasattr(cn, "extra")
False
```

> 초기화를 위해 `init_with_data`라는 일반적이지 않은 메서드의 이름을 알아야함

```python
# 파이썬스러운 방식
class Namespace:
    """Create an object form keyword arguments."""
    
    ACCEPTED_VALUES = ("id_", "user", "location")
	
    def __init__(self, **data):
        accepted_data = {
            k: v for k, v in data.items() if k in self.ACCEPTED_VALUES
        }
        self.__dict__.update(accepted_data)
```

> 파이썬에서 다른 객체를 초기화할 때는 `__init__` 메서드를 사용하는게 훨씬 간편함



### EAFP/LBYL

> EAFP: Easier to Ask Forgiveness than Permission
>
> LBYL: Look Before You Leap

EAFP와 LBYL은 상반되는 원칙인데  

EAFP는 일단  코드를 실행하고 실제 동작하지 않을 경우에 대응한다는 원칙이며  

LBYL은 코드를 실행하기 전에 무엇을 사용하려고 하는지 확인해야한다는 원칙이다.  



파이썬은 EAFP방식으로 만들어져있다.   

~~아니 그럼 LBYL은 안맞는거 아닌가요~~  

일반적으로 코드를 실행하면서 발생하는 예외를 catch하고 except블록에서 바로잡는 코드를 실행한다.

```python
# 파이썬스럽지 않은 방식
if os.path.exists(filename):
    with open(filename) as f:
        pass
```

```python
# 파이썬스러운 방식
try:
    with open(filename) as f:
        pass
except FileNotFoundError as e:
        logger.error(e)
```



## 포함(Composition)과 상속(Inheritence)

상속은 부모 클래스에게서 모든 멤버변수와 메서드를 가져오는 것이고, 포함은 클래스의 멤버변수에 다른 클래스의 객체를 선언하여 해당 멤버변수를 통해 다른 클래스의 멤버변수, 메서드에 접근하는 것을 의미한다.

### 상속의 특징
1. 부모와 강력하게 결합된 새로운 클래스가 생김
> 소프트웨어를 설계할 때는 결합력을 최소한으로 줄이는 것이 매우 중요함.
2. 코드 재사용
> 여러 상황에서 동작가능하고 쉽게 조합할 수 있는 응집력 높은 객체를 사용하는 것이 중요함.

### 상속이 좋은 케이스인 경우
#### 1. 클래스의 기능을 그대로 물려받으면서 추가 기능을 추가/특정 기능을 수정하려는 경우

```python
class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):

    """Simple HTTP request handler with GET and HEAD commands.

    This serves files from the current directory and any of its
    subdirectories.  The MIME type for files is determined by
    calling the .guess_type() method.

    The GET and HEAD requests are identical except that the HEAD
    request omits the actual contents of the file.

    """
    ...
```
> BaseHTTPRequestHandler의 기능에 GET, HEAD 요청에 대한 핸들러를 더했다.

#### 2. 인터페이스 정의
인터페이스는 Java와 같은 언어에서는 특정한 형태로 선언할 수 있으나, 파이썬에서는 이를 디폴트로 지원하지 않는다.<br>
인터페이스는 협업 상황에서 메서드, 멤버변수가 구현체가 없이 선언만 존재하는 클래스를 지칭하는 것으로, 파이썬에서는 추상클래스를 통해 다음과 같이 구현한다.
```python
class CarInterface(metaclass=ABCMeta):
    name = '차의 이름'
    
    def move(self):
        pass
	
class Ferarri(CarInterface):
    def __init__(self, name):
        self.name = name

    def move(self):
        print("부릉")

```

#### 3. 예외
앞서 설명했던 예외처리와 같이 모든 예외는 `Exception`에서 파생된다. 이것은 `except Exception:`과 같은 구문을 사용한다면 모든 예외를 catch할 수 있게된다는 것을 의미한다 

## 상속 안티패턴
> 상속은 `"A는 B이다"` 즉 is a 관계가 성립되어야 한다. 

### Case. 개발자가 코드 재사용만을 목적으로 상속을 사용.

```python
	class TransactionalPolicy(collections.UserDict):
		"""잘못된 상속의 예"""
		def change_in_policy(self, customer_id, **new_policy_data):
			self[customer_id].update(**new_policy_data)
```
#### 문제점
1. 사용자는 `TransactionalPolicy`라는 이름을 보았을 때 직관적으로 딕셔너리임을 파악할 수 없으며, `public` 인터페이스를 통해 노출된 `public` 메서드들을 확인하게 되면 특이하게 전문화된 이상한 계층구조라고 느끼게 됨.

2. 필요하지 않는 메서드가 포함되어 있음(부모 클래스인 딕셔너리 클래스의 메서드: `pop()`, `items()` 등)
```python
[ # 모든 매직 매서드는 간략화를 위해 생략..•
'change_in_policy', 'clear', 'copy', 'data', 'fromkeys', 'get', 'items', 'keys', 'pop', 'popitem', 'setdefault', 'update', 'values']
```
> `TransactionalPolicy`클래스의 인스턴스가 사용할 수 있는 메서드들.

3.`update()` 메서드가 유일한 추가 메서드로서 부모 클래스와는 상관이 없음.

#### 권장하는 해결법

`TransactionalPolicy` 자체가 사전이 되는 것이 아닌, 사전을 활용하는 관계로 변경, 필요한 사전은 `private` 속성에 저장하고 `__getitem__()`으로 사전의 프록시를 만들어 나머지 필요한 메서드를 구현

```python
class TransactionalPolicy:
	"""컴포지션 을 사용한 리팩토링 예제"""
	def __ init__ (self, policy_data , **extra_data):
		self. _data = {**policy_data, **extra_data}
		
	def change_in_policy(self, customer_id, **new_policy_data):
		self._data[customer_id].update(**new_policy_data)
		
	def __ getitem__ (self, customer_id):
		return self._data[customer_id]
		
	def __len__(self):
		return len(self._data)
```

해당 관계는 기존의 상속관계를 포함(Composition)관계로 변경한 것이다. 포함관계는 has a관계로 `A는 B를 가지고 있다.`라는 관계가 성립해야 사용한다.
해당 예시에서는 `TransactionalPolicy`가 딕셔너리이다. 라는 관계보다는 `TransactionalPolicy`가 딕셔너리를 가지고 있다. 라는 관계가 보다 맞는 말이 될 수 있다.

## 파이썬의 다중상속(⭐️⭐️⭐️⭐️⭐️)
다중상속 자체를 할 일이 현실에 많지 않음.
* 한 사물이 A,B,C 클래스가 있을 때 A클래스가 B와도 is a 관계가, C와도 is a 관계가 성립하는 것이 쉽지 않기 떄문.
* 따라서 올바르게 사용될 때만 유효한 해결책이 될 수 있으므로, 어댑터 패턴과 믹스인(mixin)을 사용하게 되었음.

~~(9장에서 Giraffe님이 멋지게 다룰 예정)~~



### 죽음의 다이아몬드
다중 상속은 아래와 같은 상황이 발생할 수 있으므로, 이를 지원하지 않는 객체지향 언어 역시 존재한다. _(ex. Java)_

<img width="500" alt="image" src=https://user-images.githubusercontent.com/59782504/170248592-75a4daf5-54a3-43c1-b697-c1fa73590679.png>


위의 상황에서는 `D.foo()`메서드를 호출할 때, `D`는 `B`클래스와 `C`클래스의 `foo()`메서드 중 어떤 것을 써야하는지 알 수 있는가?

### 메서드 결정 순서(MRO)
죽음의 다이아몬드 문제를 해결하는 알고리즘이 MRO이다. 다음과 같은 계층도를 가지는 클래스가 있다

<img width="492" alt="image" src="https://user-images.githubusercontent.com/59782504/170248925-daaa4695-6580-4ac6-a0e6-001aa74b9ee5.png">

그리고 아래와 같이 실제 구현하였다.
```python
class BaseModule:
	module_name = "top"
	
	def __init__(self, module_name): 
		self.name = module_name
		
	def __str__(self):
		return f"{self.module_name}:{self.name}"
		
		
class BaseModule1(BaseModule): 
	module _ name = "module-1"
	
	
class BaseModule2(BaseModule): 
	module_name = "module-2"
	
	
class BaseModule3(BaseModule): 
	module_name = "module-3"
	
	
class ConcreteModuleA12(BaseModule1, BaseModule2):
	"""1과 2 확장"""
	
	
class ConcreteModuleB23(BaseModule2, BaseModule3):
	"""2와 3 확장"""
```

이 경우 최하위 클래스 `str(ConcreteModuleA12("Test"))`을 호출한 결과는 아래와 같다.

```python
>>> str(ConcreteModuleA12("test")) 
'module-1:test'
```
예상으로는 `ConcreteModuleA12`가 두 클래스(`BaseModule1`, `BaseModule2`)를 상속받고 있으므로, 어떤 `module_name`이 들어갈 지 명확하지 않다. 그러나 MRO 알고리즘은 이를 해결한다.

#### MRO 알고리즘
상속받는 클래스를 먼저 정의한 순서대로 부모 클래스의 메서드를 호출하는 방식의 알고리즘을 뜻한다. `class ConcreteModuleA12(BaseModule1, BaseModule2):`에서는 `BaseModule1` 클래스를 먼저 상속받고 있으므로, `BaseModule1`의 `module_name`이 전달된다.

직접 MRO알고리즘이 어떤 순서대로 메서드를 호출하는지 `mro()`함수를 통해 확인할 수도 있다.
```python
>>>[cls.__name__ for cls in ConcreteModuleA12.mro()]
['ConcreteModuleA12', 'BaseModule1 ', 'BaseModule2', 'BaseModule', 'object']
```

### 믹스인(Mixin)
코드를 재사용하기 위해 일반적인 행동을 캡슐화 해놓은 기본클래스, <br>
일반적으로 믹스인 클래스는 그 자체로는 유용하지 않으며 대부분이 클래스에 정의된 메서드나 속성에 의존하기 때문에 이 클래스만 확장해서는 확실히 동작하지 않는다.

#### 하이픈과 영어가 포함되어 있는 문자열을 하이픈으로 구분된 값으로 파싱하는 파서 예시
```python
class BaseTokenizer:
	def __init__(self, str_token):
		self.str_token = str_token
	
	def __iter__(self):
		yield from self.str_token.split("-")
```

호출 결과
```python
>>> tk = BaseTokenizer(" 28a2320b-fd3f-4627-9792-a2b38e3c46b0") 
>>> list(tk)
['28a2320b' 'fd3f' '4627' , '9792' , 'a2b38e3c46b0']
```

#### 기본 클래스를 변경하지 않고 값을 대문자로 변환하기
```python
class UpperIterableMixin: 
	def __iter__(self):
		return map(str.upper, super().__iter__()) # super()가 정의되어 있지 않다.

class Tokenizer(UpperiterableMixin, BaseTokenizer): 
	pass
```

해당 구현방식의 놀라운 점은 상속 계층도는 아래와 같으나,

<img width="500" alt="image" src="https://user-images.githubusercontent.com/59782504/170260268-b7e9b9d8-838b-4496-ac96-fb5fdd648945.png">

실제 메서드의 흐름은 MRO알고리즘의 흐름을 따르기 때문에 아래의 사진과 같이 `Tokenizer` ► `UpperIterableMixin` ► `BaseTokenizer의` 서순을 따른다는 것이다.

<img width="500" alt="image" src="https://user-images.githubusercontent.com/59782504/170260408-9aece54b-4869-496d-9dc2-4f8c6e199258.png">


믹스인은 이처럼 새로운 코드 없이 상속만으로 새로운 기능을 구현하는 좋은 방법론이다.
> 만약 python이 컴파일 언어였다면, `UpperlterableMixin`클래스가 호출하는 `super()`이 명확하지 않기 때문에 컴파일 단계에서 오류를 호출할 것이다. ~~오늘도 어김없이 놀라운 파이썬이다.~~

## 함수와 인자
> 앞서 파라미터에 가변인자를 넣으면 안된다는 내용(2장에서 다루었음)과 가변인자를 매개변수로 전달하는 내용에 대한 부분이 있으나, 이미 다룬 내용이며, 상식적인 수준이라 제외하였음.

### 가변인자
인수의 개수가 변경될 수 있도록 한 인자 아스터리스크(*)로 패킹된 문자는 함수 내부에서 튜플처럼 활용된다.
```python
def print_my_classes(*classnames):
	for classname in classnames:
		print(classname)
```

받는 인자는 classnames 하나이지만, 호출에서는 여러 개의 인자를 받은 것처럼 활용할 수 있다.
```python
>>> print_my_classes("math", "korean", "social")
'math'
'korean'
'social'
```

패킹은 함수의 인자 뿐 아니라 다른 방향에서도 이를 활용할 수 있다.
```python
>>> def f(first, second, third): 
...		print(first)
... 	print(second) 
... 	print(third)
...
>>> l = [1, 2, 3] 
# 함수 호출 시 패킹 활용
>>> f(*l)
1
2
3
# 리스트 요소 패킹
>>> a, b, c = [1, 2, 3] 
>>> a
1
>>> b
2 
>>> c 
3

```
일련의 요소를 반복해야 하고 각 요소가 차례로 있다면 각 요소를 반복할 때 언패킹하는 것이 좋다

#### 패킹을 사용할 좋은 예
```python
def bad_users_from_rows(dbrows) -> list:
	"""DB 레코드에서 사용자를 생성하는 파이썬스럽지 않은 잘못된 사용 예""" 
	return [User(row[0], row[1], row[2]) for row in dbrows]

# more Pythonic
def users_from_rows(dbrows) -> list : 
	"""DB 레코드에서 사용자 생성""" 
	return [
		User(user_ id, first_ name, last_name)
		for (user_id, first_name, last_name) in dbrows
	]
```
언뜻 봐도 언패킹을 활용한 뒤에 정의한 함수가 알아보기 쉽다. row에 인덱스를 사용하여 조회하는 방법 역시 많이 사용되는 방법이긴 하나, python에서는 이처럼 패킹/언패킹이라는 좋은 기능을 제공한다.

## 함수 파라미터의 개수
> _"이 섹션에서는 너무 많은 인자를 사용하는 함수나 메서드가 왜 나쁜 디자인(코드의 나쁜 냄새)의 징후인지를 살펴본다."_

### 문제점
함수의 파라미터가 많은 것이 왜 문제냐고 반문할 수 있을 것이다. 많은 함수 파라미터는 읽는 사람으로 하여금 힘들게 만들고, 무슨 역할을 하는지 파악하는데 오랜 시간이 걸리게 한다.<br>
<br> 
함수의 파라미터가 많을 수록 호출하는 함수와 밀접하게 결합될 가능성이 커진다. <br>
아래의 코드를 보자
```python
def purchase(car_name, car_price, car_quentity, consumer_name, consumer_balance, employee_name, employee_point)
	car_sell(car_name, car_price, car_quentity) 
	...
	
```
이 경우 보다시피 `purchase()`함수가 무엇을 처리하는지 알기 어렵다.<br>
_(구매에만 관여하는지, 구매후 재고에도 관여하는지, 직원의 포인트를 관리하는지 등)_<br>
<br>

이 경우 두가지 문제점이 있다.<br>

<br>
1. 적절한 추상화가 이루어지지 않아 한눈에 함수를 파악하기 어렵다는 점
2. 특정 작업을 하도록 의도되었기 때문에 다른 환경에서 `purchase()`함수를 사용하기 힘들다는 점

### 해결책

#### 1. 구체화
구체화는 전달하는 파라미터를 모두 포함하는 하나의 객체를 만드는 것이다.<br>
<br>
함수의 인자로 객체를 넘기는 것으로 위 경우 가장 좋은 해결책이 될 수 있다.<br>
예시를 위해 필자는 아래와 같이 위에 쓰여있는 함수를 기반으로 리펙토링하였다. 

```python
class Product:
	def __init__(self, name, price, quentitiy)
		self.name = name
		self.price = price
		self.quentity = quentity
	...
	
class Car(Product):
	...

class Person:
	def __init__(self, name)
		self.name = name
	...

class Customer(Person):
	def __init__(self, balance, **kwargs)
		super().__init__(**kwargs)
		self.balance
	...

class Employee(Person):
	def __init__(self, point, **kwargs):
		super().__init__(**kwargs)
		self.point = point
	...

def purchase(product: Product):
	product.quentity -= 1
	...
	
def sell(employee):
	employee.point += 10
	...
```

클래스가 추가되어 코드 길이는 길어졌지만, 확장성과 추상화 측면에서 보다 쉽게 코드를 이해할 수 있다. <br>
이렇게 구체화하여 해결하는 방법은 일반적으로 파라미터가 반복되는 형태가 아닐때 주로 권장되는 방식이다. <br>
<br>
단, 이 경우도 주의할 점이 있다. 앞서 말했 듯 함수는 전달받은 객체를 변경해서는 안 된다는 것이다.

####  2. 가변인자
> 이전 파트에서 다루었기 때문에 다시 다루진 않는다. 다만, 가변인자는 만능의 해결책은 아니다. 매우 동적이어서 유지보수하기가 어렵기 때문이다. 해당 값이 어떤 파라미터인지 찾아서 이를 분기하고 있다면, 함수를 분리하라는 신호일 수 있다.

```python
def purchase(*arg)
	car_sell(arg[0:3]) 
	...
```

~~예를 들어 위의 경우 더 꼴보기 싫어질 수 있다.~~

이전 방법(구체화)가 통하지 않으면 함수의 파라미터를 \*args, \*\*kwargs 등으로 변경하여 다양한 파라미터를 받도록 허용할 수 있다. <br>
<br>
그러나 이 경우 가독성을 거의 상실하기 떄문에 인터페이스에 대한 문서화를 하고 정확하게 사용했는지 확실히 해야한다.

## 소프트웨어의 독립성
독립성은 일반적인 단어로 여러 의미나 해석을 가질 수 있으나 소프트웨어에서의 독립성은 모듈 클래스 또는 함수를 변경하면 수정한 컴포넌트가 외부 세계에 영향을 미치지 않아야한다는 것을 의미한다.
> 바람직하지만 항상 가능한 것은 아니다.

예시를 살펴보자 세금과 할인율을 고려하여 가격을 계산하는 함수를 가지고 있고, 최종 계산된 값을 포매팅하고 싶다고 해보자.
```python

def calculate_price(base_price: float, tax: float, discount: float) -> float:
	return (base_price * (1 + tax)) * (1 - discount)

def show_price(price: float) -> str: 
	return "$ {0:,.2f}".format(price)
	
def str_final_price(base_price: float, tax: float, discount: float , fmt_function=str) -> str: 
	return fmt_function(calculate_price(base_price, tax, discount))
```
위쪽 두 개의 함수는 독립성을 갖는다. 하나를 변경해도 다른 하나는 변경되지 않는다.
마지막 함수는 아무것도 전달하지 않으면 문자열 변환을 기본 표현 함수로 사용하고 사용자 정의 함수를 전달하면 해당 함수를 사용해 문자열을 포맷한다.

```python
>>> str_final_price(10, 0.2, 0.5) 
'6.0'
>>> str_final_price(1000, 0.2, 0) 
'1200.0'
>>> str_final_price(1000, 0.2, 0.1) 
'$ 1,080.00'
```

### 독립성과 관련된 품질 특성
코드의 두 부분이 독립적이라는 것은 다른 부분에 영향을 주지 않고 변경할 수 있다는 것을 뜻한다.<br>
이는 변경된 부분의 단위테스트가 나머지 단위 테스트와도 독립적이라는 뜻이다.<br>
<br>
이러한 가정하에 **개발자는 두 개의 테스트가 통과하면 전체 회귀 테스트를 하지 않고도 애플리케이션에 문제가 없다고 어느 정도 확신**할 수 있다.

## 코드 구조
코드를 구조화 하는 방법은 작업 효율성과 유지보수성에 영향을 미친다.<br>
특히 여러 정의(클래스, 함수, 상수)가 들어있는 큰 파일을 만드는 것은 좋지 않다.<br>
<br> 
극단적으로 하나의 파일에 하나의 정의만 유지하라는 것은 아니지만(실제로 자바는 클래스 단위로 극단적으로 정리한다), 좋은 코드라면 유사한 컴포넌트끼리 정리하여 구조화해야 한다.

~~모듈 분리와 파이썬 패키지 생성에 대한 자세한 내용은 10 장 "클린 아키텍처" Erin님이 멋지게 다루어주실 예정 ^^.~~

