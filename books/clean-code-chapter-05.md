# [Chatper .5] 데코레이터를 사용한 코드 개선

## 데코레이터

데코레이터는 오래 전 PEP-318에서 함수와 메서드의 기능을 쉽게 수정하기 위한 수단으로 소개되었다.<br>
<br>
`classmethod`나 `staticmethod`같은 함수가 원래 메서드의 정의를 변형하는데 사용되고 있었기 때문에 고안된 수단인데 이런 방법은 추가 코드가 필요하고 함수의 원래 정의를 수정해야만 했다.

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

데코레이터 함수의 파리머터로 함수가 아닌 클래스를 받는 데코레이터

### 주의점

클래스 데코레이터가 복잡하고 가독성을 떨어뜨릴 수 있음.
> 클래스에서 정의한 속성과 메서드를 데코레이터에서 완전히 다른 용도로 변경 가능하기 때문

### 장점

- 코드 재사용과 DRY원칙의 모든 이점을 공유함.
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

로그인 이벤트에 직접 매핑할 클래스를 선언하고 민감한 개인정보인 password 필드를 숨기고, timestamp 필드를 포매팅하였다.<br>
처음에는 잘 작동할 지 모르나, 시간이 지나면서 시스템을 확장하게 된다면 개발자는 다음과 같은 문제를 마주해야 할 수 있다.

1. 클래스가 점점 많아진다.

> 이벤트 클래스와 직렬화 클래스가 1:1로 매핑되어있다. 이렇게 만들 경우 직렬화 클래스는 본질적인 역할은 한 가지임에도 점점 많아지게 된다.

2. 유연하지 않다.

> `password`필드를 가진 다른 클래스에서도 이 필드를 숨기려면 함수로 분리한 다음 여러 클래스에서 호출해야 한다. 코드를 충분히 재사용 했다고 볼 수 없다.

3. 표준화 문제

> Serialize 메서드는 모든 이벤트 클래스에 있어야 한다. 믹스인을 사용해 분리할 수는 있으나, 상속을 제대로 사용했다고 볼 수 없다.

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
이 때 단순히 데이터 필드를 정의하는 클래스 LoginEvent는 파이썬 3.7 이상부터 정의된 dataclass 데코레이터를 통해 다음과 같이 단수화할 수 있다.

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

