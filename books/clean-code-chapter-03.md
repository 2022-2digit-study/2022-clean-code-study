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

   : 프로그램의 흐름을 읽기 어려워진다.(캡술화가 약해짐)  

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

