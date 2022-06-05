# [Chapter .4] SOLID 원칙

## 단일 책임 원칙(Single Responsibility Principle - SRP)

SRP는 **소프트웨어 컴포넌트가 단 하나의 책임을 져야한다는 원칙**이다.<br>
클래스가 유일한 책임이 있다는 것은, 하나의 구체적인 일을 담당한다는 것을 의미한다.<br>
<br>
유지보수 측면에서 보았을 때, 수정이 생겼을 때 수정할 이유 역시 단 하나 밖에 없다는 것이다.<br>
<br>
### 신 객체(God Object)
여러 책임을 가진 객체는 존재햐서는 안된다. 필요 이상의 일을 하거나 **너무 많은 것을 알고 있는 객체를 신 객체**라고 한다.<br>
<br>
이러한 객체는 서로 다른 행동을 그룹화한 것이므로 유지보수를 어렵게한다. <br>

<img width="180" alt="image" src="https://user-images.githubusercontent.com/59782504/170297247-4793099b-6fc1-4b1e-8755-dd9a86601227.png">

```python
# srp_1.PY
class SystemMonitor :
    def load_activity(self):
        """소스에서 처리할 이벤트를 가져오기"""
    
    def identify_events(self):
        """ 가져온 데이터 를 파싱하여 도메인 객체 이벤트로 변환 ''""
    
    def stream_events(self):
        """파싱한 이벤트를 외부 에이전트로 전송 ''""
```
위 코드에 정의된 `SystemMonitor`의 문제점은 독립적인 동작을 하는 메서드를 하나의 인터페이스에 정의했다는 것이다. 각각의 동작은 나머지 부분과 독립적으로 수행할 수 있다.<br>
<br>
만약 위 경우 데이터 구조를 바꾸는 등의 이유로 이 중에 어떤 것이라도 수정해야 한다면 `SystemMonitor` 클래스를 변경해야 한다.

### Object 분리와 추상화 ~~(feat. 신 타락시키기)~~

<img width="400" alt="image" src="https://user-images.githubusercontent.com/59782504/170297355-89983e35-6088-4ddd-87c3-c5a0bc5e5364.png">

데이터 소스에서 이벤트를 로드하는 방법(`ActivityReader`)을 변경해도 `AlertSystem`은 이러한 변경 사항과는 관련이 없으므로 `SystemMonitor`는 아무 것도 수정하지 않아도 된다.<br>
`Output`도 마찬가지로 수정하지 않아도 된다<br>
<br>
그러나 각 클래스가 딱 하나의 메서드를 가져야 한다는 것을 뜻하는 것은 아님에 주의하자. 처리해야 할 **로직이 같은 경우** 하나의 클래스에 여러 메서드를 추가할 수 있다.



## 개방/폐쇄 원칙

개방/폐쇄 원칙(Open/Close Principle)은 모듈이 **한 측면에서는 개방되어 있으면서도 다른 측면에서는 폐쇄되어야한다**는 원칙이다.  

그렇다면 어떤 측면에서 개방되어있어야하고 어떤 측면에서 폐쇄되어있어야할까?

개방되어있어야 하는 부분은 로직을 캡슐화하여 새로운 요구사항이나 도메인 변화로의 확장이다.  

반면 폐쇄되어있어야 하는 부분은 기존 코드의 수정으로, 요구사항이 변경되면 새로운 기능을 구현하기 위해 추가만 하고 기존 코드를 수정하지 않아야함을 의미한다.

만약 새로운 기능을 추가하다가 기존 코드를 수정했다면 이는 기존 로직이 잘못 디자인되었다는 것이다.  



### OCP 위반 사례와 리팩토링

이벤트를 분류하는 기능을 가진 예제로 OCP 원칙을 따르지 않은 사례를 확인해보자.  

```python
class Event:
    def __init__(self, raw_data):
        self.raw_data = raw_data
        
class UnknownEvent(Event):
    """데이터만으로 식별할 수 없는 이벤트"""
    
class LoginEvent(Event):
    """로그인 사용자에 의한 이벤트"""

class LogoutEvent(Event):
    """로그아웃 사용자에 의한 이벤트"""
    
class SystemMonitor:
    """시스템에서 발생한 이벤트 분류"""
    
    def __init__(self, event_data):
        self.event_data = event_data
        
    def identify_event(self):
        if (
        	self.event_data["before"]["session"] == 0
            and self.event_data["after"]["session"]  == 1
        ):
            return LoginEvent(self.event_data)
        elif (
        	self.event_data["before"]["session"] == 1
            and self.event_data["after"]["session"] == 0
        ):
            return LogoutEvent(self.event_data)
        
        return UnknownEvent(self.event_data)
```

이벤트 유형의 계층 구조와 일부 비지니스 로직을 명확히 알 수 있으나 아래와 같은 문제점도 있다.

1. 이벤트 유형을 결정하는 논리가 중앙 집중화 되어있다.(SystemMonitor)
   - 이벤트가 늘어나면 매우 큰 메서드가 될 수 있음 
   - 한 가지 일만 하지 않음

2. 수정에 폐쇄적이지 않다.

   - class가 닫혀있지 않음

   - 새로운 이벤트 추가시 계속 수정해야함

이 코드를 어떻게 리팩토링해야할까?  

   

이 문제점들을 해결해보자면 아래와 같이 리팩토링할 수 있다.  

```python
class Event:
    def __init__(self, raw_data):
        self.raw_data = raw_data
       
    @staticmethod
    def meets_condition(event_data: dict):
        return False
    
class UnknownEvent(Event):
    """데이터만으로 식별할 수 없는 이벤트"""
    
class LoginEvent(Event):
    @staticmehtod
    def meets_condition(event_data: dict):
        return (
        	event_data["before"]["session"] == 0
            and event_data["after"]["session"] == 1
        )
    
class LogoutEvent(Evnet):
    @staticmethod
    def meets_condition(event_data: dict):
        return (
        	event_data["before"]["session"] == 1
            and event_data["after"]["session"] == 0
        )
    
class SystemMonitor:
    """시스템에서 발생한 이벤트 분류"""
    
    def __init__(self, evnet_data):
        self.event_data = event_data
        
    def identify_event(self):
        for event_cls in Event.__subclass__():
            try:
                if event_cls.meets_condition(self.event_data):
                    return event_cls(self.event_data)
            except KeyError:
                continue
                
        return UnknownEvent(self.event_data)
```

SystemMonitor를 추상적인 이벤트와 협력하도록 대응하였고 이벤트와 각 이벤트 클래스가 대응하게 하였다.  

즉, 분류 메서드(SystemMonitor)가 특정 이벤트 타입과 직접 상호작용하지 않고  일반적인 인터페이스(Event)와 동작하게 되었다.  (폐쇄)

또한 `__subclass__`를 사용하고 있기 때문에 새로운 이벤트가 추가된다고 하더라도 `Event`클래스를 상속받고 `meets_condition()` 메서드만 구현하면 되므로 메서드의 크기도 더이상 커지지 않는다. (확장)

실제 확장은 다음과 같이만 코드를 추가하면 된다.  

```python
class TransactionEvent(Event):
    """시스템에서 발생한 트랜잭션 이벤트"""
    
    @staticmethod
    def meets_condition(event_data: dict):
        return event_data["after"].get("trasaction") is not None
```

단, 특정 요구사항에 대한 추상화는 다른 요구사항에 적절하지 않을 수 있기 때문에 모든 프로그램에 적용가능한 원칙은 아니다.  



## 리스코프 치환 원칙(LSP)

리스코프 치환 원칙(Liskov Substitution Principle)은 **프로그램을 변경하지 않고 하위 타입의 객체로 치환이 가능해야 한다**는 원칙이다.  

즉, 클라이언트는 완전히 분리되어 있으며 클래스 변경 사항과 독립이 되어야한다.  

이 원칙은 계약을 통한 설계와도 관련이 있는데,  

주어진 타입과 클라이언트 사이에는 계약이 필요하며, 하위 클래서는 상위클래스에서 정의한 계약을 따르도록 디자인해야한다.  

LSP는 인터페이스의 메서드가 올바른 계층구조를 갖도록 하여 상속된 클래스가 부모 클래스와 다형성을 유지하도록 한다.  

LSP는 새로운 클래스가 계약과 호환되지 않는 확장을 하려고 한다면 클라이언트와의 계약이 깨지므로 확장가능하지 않게 한다.  

또한 확장을 가능하게 하려면 수정에 대해 폐쇄되어야함을 깨게 된다.  

즉, LSP는 OCP를 잘 지키도록 기여한다고도 할 수 있다.



### LSP 위반 사례

아래의 사례는 대표적인 LSP 위반 사례이다.

```python
class Event:
    ...
    def meets_condition(self, event_data: dict) -> bool:
        return False
    
class LoginEvet(Event):
    def meets_condition(self, event_data: list) -> bool:
        return bool(event_data)
```

```
# 에러 메세지
error: Argument 1 of "meets_condition" incompatible with supertype "Event"
```

파생 클래스가 부모 클래스에서 정의한 파라미터와 다른 타입을 사용했다.  

이 두 가지 타입의 객체를 치환하면 애플리케이션 실행에 실패하게 되는데, 이는 계층 구조의 다형성이 손상된 것이라고 할 수 있다.  

만약 반환 값을 부울 값이 아닌 다른 값으로 변경하더라도 같은 에러가 발생하게 되는데,  

그 이유는 클라이언트가 반환 값으로 부울 값을 사용할 것으로 기대하기 때문에 계약을 위반하게 되기 때문이다.  



또 다른 대표적 LSP 위반 사례를 살펴보자.  

```python
class LogoutEvent(Event):
    def meets_condition(self, event_data: dict, override: bool) -> bool:
        if override:
            return 
        ...
```

```
# 에러 메세지
Parameters differ form overridden "meets_condition" method (argumentsdiffer)
```

이는 메서드의 서명 자체가 아예 다른 경우이다.  

파이썬은 인터프리터 언어이므로 초기에 컴파일러를 통해 이런 오류들을 감지하는 것이 어려워 런타임까지 발견되지 않게 된다.  

따라서 정적 코드 분석기를 활용하여 이런 LSP 를 위반하는 오류들을 초기에 잡을 수 있으므로 활용하는 것이 좋다.  



## 인터페이스 분리 원칙

인터페이스 분리 원칙을 알아보기에 앞서서 파이썬에서의 인터페이스에 대해 먼저 알아보자.  



### 파이썬에서의 인터페이스

일반적으로 인터페이스는 객체가 노출하는 메서드의 집합이며 다른 클라이언트에서 호출할 수 있는 요청이다.  

파이썬에서의 인터페이스는 `duck typing` 원리를 따랐으나 파이썬3에서 추상 기본 클래스 개념을 도입하였다.  

_(duck typing원리란 어떤 새가 오리처럼 걷고 오리처럼 꽥꽥 소리를 낸다면 오리여야만 한다는 원리)_

추상 기본 클래스는 파생 클래스가 구현해야 할 일부분을 기본 동작 또는 인터페이스로 정의하는 것이다.  

또한 가상 하위 클래스(virtual subclass)라는 타입을 계층구조에 등록할 수 있다.  

이는 기존의 duck typing의 개념을 확장시키는 것이다.  



인터페이스 분리 원칙(Interface Segregation Principle)은 **다중 메서드를 가진 인터페이스가 있다면 더 적은 수의 메서드를 가진 여러 개의 메서드로 분할하는 것이 좋다**는 것이다.  

가능한 작은 단위로 인터페이스를 분리하면 각 클래스가 명확한 동작과 책임을 갖게 되기 때문에 응집력이 높아진다.  



### 인터페이스는 얼마나 작아야할까?

인터페이스는 응집력의 관점에서 가능한 단 한가지 일을 수행하는 작은 인터페이스여야한다.  

하지만 그렇다고해서 반드시 딱 한가지 메서드만 있어야 하는 것은 아니다.  

가령 컨텍스트 관리자를 추상화한 믹스인 클래스를 제공한다면,  

믹스인 클래스를 상속받은 클래스는 컨텍스트 관리자의 `__enter__`과 `__exit__` 라는 두 가지 메서드를 갖게 된다.   

따라서 작은 인터페이스의 의미를 오해하여 극단적으로 받아들여서는 안된다.



## 의존성 역전

의존성 역전 원칙(DIP)은 **코드가 세부 사항이나 구체적인 구현에 적응하도록 하지 않고 추상화된 객체에 적응하도록 하는 것**이다.  

특히 이 원칙은 외부 라이브러리 혹은 다른 팀의 모듈을 사용하는 경우 코드가 깨지거나 손상되는 취약점으로부터 보호해준다.  

만약 외부 코드에 의존하게되면 외부 코드가 변경될 때 원래의 코드가 깨지게 되는데,  

의존성 역전을 위해 추상화(인터페이스)를 개발하고 외부 코드가 인터페이스에 의존적이도록 해야한다.  

일반적으로 구체적인 구현이 추상화된 컴포넌트보다 더욱 자주 바뀌기 때문에  

시스템의 변경, 수정, 확장에서의 유연성을 확보하기 위해 추상화(인터페이스)를 사용하는 것은 중요하다.  

~~예시가 나와있지만 9장에서 Giraffe님이 10장에서 Erin님이 또 멋지게 다뤄준다고 한다.~~

~~대체 9장과 10장에는 무슨 내용이 담긴걸까~~

