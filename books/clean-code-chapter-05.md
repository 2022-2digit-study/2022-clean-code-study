# [Chatper .5] 데코레이터를 사용한 코드 개선

## 데코레이터

데코레이터는 오래 전 PEP-318에서 **함수와 메서드의 기능을 쉽게 수정하기 위한 수단으로 소개**되었다.<br>
<br>
`classmethod`나 `staticmethod`같은 함수가 원래 메서드의 정의를 변형하는데 사용되고 있었기 때문에 고안된 수단인데 이런 방법은 추가 코드가 필요하고 함수의 원래 정의를 수정해야만 했다.

### 데코레이터는 고차함수

크게 보면 데코레이터는 **함수를 파라미터로 받아서 함수를 반환하는 함수**이다. 이런 함수를 **고차함수**라고 부른다. <br>
실제로 데코레이터 본문에 정의된 함수가 호출된다. 아래의 예를 보자<br>

### 데코레이터 적용 전

```python
def original(...):
    ...


original = modifier(original)
```

### 데코레이터 적용 후

```python
@modifier
def original(...):
    ...
```

정리하면 즉, 데코레이터는 데코레이터 이후에 나오는 것을 데코레이터의 첫 번째 파라미터로 하고 데코 레이터의 결과 값을 반환하게 하는 문법적 설탕*(syntax sugar)일 뿐이다.<br>
<br>
위의 예제에서 `modifier`는 파이썬 용어로 데코레이터라 하고, `original`을 데코레이팅된 decorated) 함수 또는 래핑된(wrapped) 객체라 한다.<br>
<br>
사실 함수 뿐 아니라 메서드, 클래스, 제너레이터 등에도 적용이 가능하다.

## 함수 데코레이터(Function Decorator)

이론적으로 함수 데코레이터는 어떤 종류의 로직에도 적용 가능하다

### 사용 예

1. 파라미터 유효성 검사
2. 사전조건 검사
3. 기능 전체를 새롭게 정의
4. 서명을 변경
5. 원래 함수의 결과를 캐싱

### 도메인의 특정 예외에 대해서 특정 횟수만큼 재시도 하는 데코레이터 예

▼ 데코레이터 선언

```python
class ControlledException(Exception):
    """도메인에서 발생하는 일반적인 예외"""


def retry(operation):
    @wraps(operation)
    def wrapped(*args, **kwargs):
        last_raised = None
        RETRIES_LIMIT = 3

        for _ in range(RETRIES_LIMIT):
            try:
                return operation(*args, **kwargs)
            except ControlledException as e:
                logger.info("retrying %s", operation.__qualname__)
                last_raised = e

        raise last_raised

    return wrapped
```

▼ 데코레이터 사용

```python

@retry
def run_operation(task):
    """실행 중 예외가 발생할 것으로 예상되는 특정 작업율 실행 """
    return task.run()

```

## 클래스 데코레이터(Class Decorator)

데코레이터 함수의 파리미터로 함수가 아닌 **클래스를 받는 데코레이터**

### 주의점

클래스 데코레이터가 복잡하고 가독성을 떨어뜨릴 수 있음.
> 클래스에서 정의한 속성과 메서드를 **데코레이터에서 완전히 다른 용도로 변경 가능하기 때문**

### 장점

- 코드 재사용과 DRY(Don't Repeat Yourself)원칙의 모든 이점을 공유함.
- 작고 간단한 클래스를 먼저 생성하고 추후 데코레이터로 기능을 보강할 수 있음.
- 특정 클래스에 대해서는 유지보수 시 데코레이터를 사용해 기존 로직을 쉽게 변경 가능 _(메타 클래스를 사용하는 방법도 있으나 권장되진 않음)_

```python
class LoginEventSerializer:
    def __init__(self, event):
        self.event = event

    def serialize(self) -> dict:
        return {
            "username": self.event.username,
            "password": "**민감한 정보 삭제**",
            "ip": self.event.ip,
            "timestamp": self.event.timestamp.strftime("%Y-%m-%d %H:%M"),
        }


class LoginEvent:
    SERIALIZER = LoginEventSerializer

    def __init__(self, username, password, ip, timestamp):
        self.username = username
        self.password = password
        self.ip = ip
        self.timestamp = timestamp

    def serialize(self) -> dict:
        return self.SERIALIZER(self).serialize()
```

로그인 이벤트에 직접 매핑할 클래스를 선언하고 민감한 개인정보인 `password` 필드를 숨기고, `timestamp` 필드를 포매팅하였다.<br>
처음에는 잘 작동할 지 모르나, 시간이 지나면서 시스템을 확장하게 된다면 개발자는 다음과 같은 문제를 마주해야 할 수 있다.

1. 클래스가 점점 많아진다.

> 이벤트 클래스와 직렬화 클래스가 1:1로 매핑되어있다. 이렇게 만들 경우 직렬화 클래스는 본질적인 역할은 한 가지임에도 점점 많아지게 된다.

2. 유연하지 않다.

> `password`필드를 가진 다른 클래스에서도 이 필드를 숨기려면 함수로 분리한 다음 여러 클래스에서 호출해야 한다. 코드를 충분히 재사용 했다고 볼 수 없다.

3. 표준화 문제

> `serialize()` 메서드는 모든 이벤트 클래스에 있어야 한다. 믹스인을 사용해 분리할 수는 있으나, 상속을 제대로 사용했다고 볼 수 없다.

```python
from datetime import datetime


def hide_field(field) -> str:
    return "**민감한 정보 삭제**"


def format_time(field_timestamp: datetime) -> str:
    return field_timestamp.strftime("%Y-%m-%d %H:%M")


def show_original(event_field):
    return event_field


class EventSerializer:
    def __init__(self, serialization_fields: dict) -> None:
        self.serialization_fields = serialization_fields

    def serialize(self, event) -> dict:
        return {
            field: transformation(getattr(event, field)) for field, transformation in self.serialization_fields.item()
        }


class Serialization:
    def __init__(self, **transformations):
        self.serializer = EventSerializer(transformations)

    def __call__(self, event_class):
        def serialize_method(event_instance):
            return self.serializer.serialize(event_instance)

        event_class.serialize = serialize_method
        return event_class


@Serialization(username=show_original,
               password=hide_field,
               ip=show_original,
               timestamp=format_time)
class LoginEvent:
    def __init__(self, username, password, ip, timestamp):
        self.username = username
        self.password = password
        self.ip = ip
        self.timestamp = timestamp
```

이 때 단순히 데이터 필드를 정의하는 클래스 `LoginEvent`는 파이썬 3.7 이상부터 정의된 `dataclass` 데코레이터를 통해 다음과 같이 단순화할 수 있다.

```python
from dataclasses import dataclass
from datetime import datetime


@dataclass
class LoginEvent:
    username: str
    password: str
    ip: str
    timestamp: datetime
```

## 파라미터를 갖는 데코레이터를 구현하는 방법

### 재시도 데코레이터(`@retry`)에 인스턴스마다 재시도 횟수를 지정하고 싶은 경우

#### 1. 간접 참조

> 새로운 레벨의 중첩 함수를 만들어 데코레이터의 모든 것을 한단계 더 깊게 만드는 방식

`@retry` 데코레이터의 인자로 파라미터를 넣은 형태가 된다. `@retry(arg1 , arg2, ...)`<br>
<br>
이는 의미상 다음과 같다.<br>
`<original_function> = retry(arg1 , arg2, .... ) (<original_function>)`

#### 데코레이터 구현

```python
RETRIES_LIMIT = 3

def with_retry(retries_limit = RETRIES_LIMIT, allowed_exceptions = None):
    allowed_exceptions = allowed_exceptions or (ControlledException,)
    def retry(operation):
       @wraps(operation)
       def wrapped(*args, **kwargs):
           last_raised = None
           for _ in range(retries_limit):
               try:
                   return operation(*args, **kwargs)
               except allowed_exceptions as e:
                   logger.info("retrying %s due to %s ", operation, e)
                   last_raised = e
           raise last_raised

        return wrapped

    return retry

```

#### 데코레이터 호출

```python
# decorator_parameterized_1.py
@with_retry()
def run_operation(task):
    return task.run()


@with_retry(retries_limit=5)
def run_with_custom_retries_limit(task):
    return task.run()


@with_retry(allowed_exceptions=(AttributeError,))
def run_with_custom_exceptions(task):
    return task.run()


@with_retry(retries_limit=4, allowed_exceptions=(ZeroDivisionError, AttributeError))
def run_with_custom_parameters(task):
    return task.run()
```

#### 2. 데코레이터를 위한 클래스를 만들기

> 일반적으로 가독성이 더 좋음 _(세 단계이상 중첩된 클로저 함수보다 객체가 더 이해하기 쉽기 때문)_

```python
class WithRetry:
    def __init__(self, retries_limit=RETRIES_LIMIT, allowed_exceptions=None):
        self.retries_limit = retries_limit
        self.allowed_exceptions = allowed_exceptions or (ControlledException,)

    def __call__(self, operation):
        @wraps(operation)
        def wrapped(*args, **kwargs):
            last_raised = None

            for _ in range(self.retries_limit):
                try:
                    return operation(*args, **kwargs)
                except self.allowed_exceptions as e:
                    logger.info("retrying %s due to %s", operation, e)
                    last_raised = e

            raise last_raised

        return wrapped
```

#### Flow

1. `@WithRetry` 데코레이터 부착

```python
@WithRetry(retries_limit=5)
def run_with_custom_retries_limit(task):
    return task.run()
```

2. `__init__()` 메서드에 파라미터 전달 ▶ `WithRetry(5)(run_with_custom_retries_limit)`
3. `__call__()` 매직 메서드에서 데코레이터 로직 실행 ▶ `wrapped(task) => operation(task)`
4. `operation(task)` 호출 ▶ `run_with_custom_retries_limit(task) => task.run()`

## 데코레이터의 좋은 구현 예

1. 파라미터 변환

> 3번 파라미터 유효성 검사와 함께 DbC(Design by Contract)원칙에 따라 사전조건 또는 사후조건을 강제할 수 있다.

2. 코드 추적
   &nbsp;&nbsp;&nbsp;&nbsp;3.1 함수 경로 추적(stack trace)<br>
   &nbsp;&nbsp;&nbsp;&nbsp;3.2 함수 지표 모니터링(CPU 사용률, 메모리 사용량)<br>
   &nbsp;&nbsp;&nbsp;&nbsp;3.3 함수 실행시간 측정<br>
   &nbsp;&nbsp;&nbsp;&nbsp;3.4 함수 호출 시점 및 파라미터 로깅<br>
3. 파라미터 유효성 검사
4. 재시도 로직 구현
5. 일부 반복 작업을 데코레이터로 이동하여 클래스 단순화

## 흔한 실수

1. 래핑된 객체를 유지하기

```python
def trace_decorator(function):
    def wrapped(*args, **kwargs):
        logger.info("%s 실행:", function. __qualname__)
        return function(*args, **kwargs)

    return wrapped


@trace_decorator
def process_account(account_id):
    """id 별 계정 처리"""
    logger.info("%s 계정 처리", account_id)
```

다음과 같은 데코레이터를 적용한 함수 `process_account()` 있다. 함수의 정보를 위해 `help()` 메서드를 써보자.

```python
>>> help(process_account)
Help on function wrapped in module decorator_wraps_1:

wrapped(*args, **kwargs)
```

난 `process_account()`메서드의 정보를 보고 싶었는데, 데코레이터에 선언한 `wrapped()`메서드의 정보가 나왔다.
이는 보안적으로도 좋지 않으며, 적절한 시기에 함수의 문서를 볼 수 없게 만든다.

말이 데코레이터이지 `wrapped()`로 재정의 한것과 마찬가지이기 때문에 발생한 현상이며, 이는 `@wraps()`메서드를 통해 해결할 수 있다.

```python
from functools import wraps


def trace_decorator(function):
    @wraps(function)
    def wrapped(*args, **kwargs):
        logger.info("running %s ", function.__qualname__)
        return function(*args, **kwargs)

    return wrapped
```

```python

>>> help(process_account)

Help on function process_account in module decorator_wraps_2:

process_account(account_id)
'id 별 계정 처리'

>>> process_account.__qualname__
'process_account'
```

만약 수정되지 않은 원본에 접근하고 싶으면 `__wrapped__` 속성을 사용하여 접근할 수 있다.

## 데코레이터 부작용 처리

데코레이터 내부의 코드는 가장 안쪽에 정의된 함수에 해야 한다.
> 그렇지 않으면 `import` 과정에서 문제가 발생할 수 있다.

```python
import functools


def traced_function_wrong(function):
    logger.info("%s 함수 실행", function)
    start_time = time.time()

    @functools.wraps(function)
    def wrapped(*args, **kwargs):
        result = function(*args, **kwargs)
        logger.info(
            "함수 %s의 실행시간: %.2fs", function, time.time() - start_time
        )
        return result

    return wrapped
```

```python
@traced_function_wrong
def process_with_delay(callback, delay=0):
    time.sleep(delay) 
    return callback( )
```
실행을 직접 해보아도 의도한 결과대로 동작하는 듯 하다.<br>
<br>
위 함수는 얼핏보면 문제 없이 보이나 중대한 문제가 있다.
해당 함수를 대화형 인터프리터에서 `import` 해보면,
```python
>>> from decorator_side_effects_1 import process_with_delay 
INFO:<function process_with_delay at 0x...> 함수 실행
```
이상하다. 나는 함수를 호출한 적이 없다.

```python
>>> main()
INFO:함수 <function process_with_delay at 0x〉의 실행시간: 8.67s
>>> main()
INFO:함수 <function process_with_delay at 0x〉의 실행시간: 13.39s
>>> main()
INFO:함수 <function process_with_delay at 0x〉의 실행시간: 17.01s
```
위처럼 반복해보면 문제는 더욱 심각해지는데 실행할 때마다 실제 함수 실행시간이 오래걸리는 것을 알 수 있다.

이는 데코레이터의 정의를 보면 알 수 있다.<br>
`@traced_function_wrong`는 실제로는 다음구문을 의미한다.<br>

```python
process_with_delay = traced_function_wrong(process_with_delay)
```

문제는 이 문구가 데코레이터를 `import` 하는 시점에 실행된다는 것이다.<br>
따라서 `start_time`은 모듈을 처음 `import` 할 때의 시간이 되며, 연속적으로 호출하면 함수의 최초 시작 시점과 시간차를 계산해서 실행되는 것이다.<br>
<br>
다행히 이런 문제를 바로 잡는 것은 매우 쉽다. 위에 설명한 원칙에 따라 아래와 같이 래핑된 함수 내부로 코드를 이동시키면 된다.
```python
def traced_function_wrong(function):
    @functools.wraps(function)
    def wrapped(*args, **kwargs):
        logger.info("%s 함수 실행", function)
        start_time = time.time()
        result = function(*args, **kwargs)
        logger.info(
            "함수 %s의 실행시간: %.2fs", function, time.time() - start_time
        )
        return result

    return wrapped
```

### 부작용 활용하기

부작용을 의도적으로 사용하여 실제 실행이 가능한 시점까지 기다리지 않는 경우도 있다.<br>
#### 모듈의 공용 레지스트리에 객체 등록하기
```python
EVENTS_REGISTRY = {}
def register_event(event_cls) :
    """모듈에서 접근 가능하도록 이벤트 클래스를 레지스트리에 등록""" 
    EVENTS_REGISTRY[event_cls.__name__] = event_cls
    return event_cls
class Event:
    """기본 이벤트 객체"""
    
class UserEvent: 
   TYPE = "user"
   
@register_event
class UserLoginEvent(UserEvent):
    """사용자가 시스템에 겁근했을 때 발생하는 이벤트"""
    
@register_event
class UserlogoutEvent(UserEvent):
    """사용자가 시스템에서 나갈 때 발생하는 이벤트"""
```

만약 위에서 일부 이벤트만 사용하고 싶다면,<br> 
기존과 같은 방식은 계층 구조에 가상의 클래스르 만들고 일부 파생 클래스에 대해서만 처리해야 할 것이다.<br>
<br>
그러나 부작용을 활용하면 위에서 선언한 일부 이벤트를 `import` 하는 것으로 `EVENTS_REGISTRY`에 데코레이터로 지정한 클래스로 채워지게 된다.

```python
>>> from decorator_side_effects_2 import EVENTS_REGISTRY
>>> EVENTS_REGISTRY

{'UserloginEvent': decorator_side_effects_2.UserLoginEvent,
'UserlogoutEvent': decorator_side_effects_2.UserLogoutEvent}
```

그러나 이 코드는 이해하기 어렵다.<br>
`EVENTS_REGISTRY`는 런타임 중에 모듈을 임포트한 직후에야 최종 값을 가지므로 코드만 봐서는 어떤 값이 될지 쉽게 예측 하기 어렵기 때문이다.<br>
<br>
그러나 많은 웹 프레임워크나 널리 알려진 라이브러리들은 이 원리를 통해 객체를 노출하거나 활용하고 있다.

## 데코레이터의 표준화
데코레이터는 여러 시나리오에 적용될 수 있다.
> 예를 들어 같은 데코레이터를 함수나 클래스, 메서드 또는 정적 메서드 등 여러 곳에 재사용하려는 경우이다.

그러나 데코레이터는 다른 유형에 적용을 하려면 많은 상황에서 오류가 발생한다는 것을 알 수 있다.<br>
일반적으로 `*args`와 `**kwargs` 서명을 시용하여 데코레이터를 정의하면 모든 경우예 사용할 수 있다.<br>
<br>
그러나 아래의 이유로 원래 함수의 서명과 비슷하게 데코레이터를 정의하는 것이 좋을 때가 있다.
1. 원래의 함수와 모양아 비슷하기 때문에 읽기가 쉽다
2. 파라미터를 받아서 뭔가를 하려면 `*args`와 `**kwargs`를 사용하는 것이 불편하다.

파라미터를 받아서 특정 객체를 생성하는 경우가 많을 경우 파라미터를 변환해주는 데코레이 터를 만들어 중복을 제거할 수 있다.

### 파라미터를 변환해주는 데코레이터

#### 선언
```python
import logging
from functools import wraps
logger = logging.getLogger(__name__)

class DBDriver:
    def __init__(self, dbstring):
        self.dbstring = dbstring
        
    def execute (self , query):
        return f"{self.dbstring} 에서 쿼리 {query} 실행"

def inject_db_driver(function) :
    """데이터베이스 dns 문자열을 받아서 DBDriver 인스턴스를 생성하는 데코레이터"""
    @wraps(function)
    def wrapped(dbstring):
        return function(DBDriver(dbstring)) 
    return wrapped

@inject_db_driver
def run_query(driver):
    return driver.execute("test_function")
```

#### 함수를 통한 호출
```python
>>> run_query("test_OK")
'test_OK 에서 쿼리 test_function 실행'
```
정상적으로 동작한다.

#### 클래스를 통한 호출
```python
class DataHandler:
    @inject_db_driver
    def run_query(self, driver) :
        return driver.execute(self.__class__.__name__)
```
```python
>>> DataHandler().run_query("test_fails")
Traceback (most recent call last):
    ...
TypeError: wrapped() takes 1 positional argument but 2 were given
```
클래스 내부의 메서드의 경우 모든 메서드에 `self`라는 추가 변수가 있다.<br>
따라서 위 오류에서도 1개의 파라미터를 받는 것을 의도하였으나, 2개를 받았다고 경고하는 것이다.<br>

#### 해결
이 문제를 해결하려면 메서드와 함수에 대해서 동일하게 동작하는 데코레이터를 만들어야 한다.<br>
디스크립터 프로토콜을 구현한 데코레이터 객체를 만든다.<br>

~~디스크립터는 7장 “제너레이터 사용하기”에서 Giraffe님이 자세히 알아볼 예정~~

```python
from functools import wraps 
from types import MethodType

class inject_db_driver:
    """문자열을 DBDricer 인스턴스로 변환하여 래핑된 함수에 전달"""
    def __init__(self, function): 
        self.function = function
        wraps(self.function)(self)
        
    def __call__(self, dbstring):
        return self.function(DBDriver(dbstring))
    
    def __get__(self, instance, owner): 
        if instance is None:
            return self
        return self.__class__(MethodType(self.function, instance))
```
~~잘 모르겠어서 논리를 같이 읽어보았으면 좋겠습니다.~~<br>
<br>
현재 상황에서는 호출할 수 있는 객체를 메서드에 다시 바인딩한다는 정도만 알면 된다. <br>
즉 함수를 객체에 바인딩하고 데코레이터를 새로운 호출 가능 객체로 다시 생성한다.<br>
<br>



## 데코레이터와 관심사의 분리

```python
import functools
import logging
import time

logger = logging.getLogger()
def traced_function(function): 
    @functools.wraps(function) 
    def wrapped(*args, **kwargs):
        logger.info( "%s 함수 실행" , function .__qualname__) 
        start_time = time.time()
        result = function(*args, **kwargs)
        logger.info("함수 %s 처리 소요시간 %.2fs", function .__qualname__ , time.time( ) - start_time)
        return result 
    return wrapped
```
다음은 실행한 함수를 호출하고 기록하는 함수이다.<br>
이 함수는 문제점이 하나 있다. 바로 이전 줄에 말했던 부분이다.<br>
<br>
위 데코레이터는 아래와 같이 좀 더 구체적이고 제한적인 책임을 지닌 더 작은 데코레이터로 분류되어야 한다.

```python
import functools
import logging
import time
logger = logging.getLogger()

def log_execution(function): 
    @functools.wraps(function)
    def wrapped(*args, **kwargs):
        logger.info( "started execution of %s", function.__qualname__)
        return function(*kwargs, **kwargs) 
    return wrapped

def measure_time(function): 
    @functools.wraps(function)
    def wrapped(*args, **kwargs):
        start_time = time.time( )
        result = function(*args, **kwargs)
        logger.info("function %s took %.2f", function.__qualname__, time.time() - start_time)
        return result
```

로깅도 시간이 들어가며, 반복적으로 호출되는 함수의 경우 로그 확인을 어렵게 할 수도 있다.<br>
또한 기본적으로 출력량이 많아지면, 미미하지만 함수의 수행속도에 영향을 미친다. <br>
따라서 개발자는 최소한의 성능시간만 체크하고 싶을 수 있다.<br>
<br>

반대로 성능보다 로그를 중요시 봐야 할 수 있다.<br>
보안적인 측면에서 취약점을 쉽게 발견하기 위해 중요한 함수에서는 성능보다는 로깅을 중요시할 수 있다.<br>
<br>
따라서 개발자는 데코레이터의 관심사를 분리하여 적절히 설계할 필요가 있다. 

## 데코레이터를 권하는 경우(⭐️)
1. 처음부터 데코레이터를 만들지 않는다. 패턴이 생기고 데코레이터에 대한 추상화가 명확해지면 그 때 리팩토링을 한다.
2. 데코레이터가 적어도 3회 이상 필요한 경우에만 구현한다.
3. 데코레이터 코드를 최소한으로 유지한다.

## 좋은 데코레이터가 갖추어야 할 특성
1. **캡슐화와 관심사의 분리**
> 좋은 데코레이터는 실제로 하는 일과 데코레이팅하는 일의 책임을 명확히 구분해야 한다.
> 어설프게 추상화 하면 안 된다. 
> 즉, 데코레이터의 클라이언트는 내부에서 어떻게 구현했는지 전혀 알 수 없는 블랙박스 모드로 동작해야 한다.

2. **독립성**
> 데코레이터가 하는 일은 독립적이어야 하며 데코레이팅되는 객체와 최대한 분리되어야 한다.

3. **재사용성**
> 데코레이터는 하나의 함수 인스턴스에만 적용되는 것이 아니라 여러 유형에 적용 가능한 형태가 바람직하다. 
> 왜냐하면 하나의 함수에만 적용된다면 데코레이터가 아니라 함수로 대신할 수도 있기 때문이다.
> 충분히 범용적이어야 한다.
