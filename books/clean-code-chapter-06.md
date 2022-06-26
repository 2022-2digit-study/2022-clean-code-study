# [Chapter .6] 디스크립터로 더 멋진 객체 만들기

디스크립터(Descriptor)는 파이썬의 고급 기능으로 다른 언어에서는 생소한 개념이다.    

[파이썬 3.10.5 의 공식 문서](https://docs.python.org/ko/3/howto/descriptor.html)에 따르면 디스크립터는 **`__get__()`, `__set__()` 또는 `__delete__()`를 정의하는 모든 객체**를 부르는 이름이다. (선택적으로 `__set_name__()` 메서드를 가지기도 함)

즉, 이 세 가지 매직 메서드가 어트리뷰트에 정의되면 그것을 **디스크립터**라고 한다.  

디스크립터를 구현하는 이유는 클래스 변수에 저장된 객체가 어트리뷰트 조회 중 발생하는 일을 제어할 수 있도록 하는 것이다.  



## 디스크립터 개요

디스크립터를 구현하려면 최소 두 개의 클래스(`클라이언트 클래스`와 `디스크립터 클래스`)가 필요하다.  

`클라이언트 클래스`는 일반적인 추상화 객체이며 `디스크립터 클래스`는 디스크립터 로직 구현체이다.  

`디스크립터`는 디스크립터 프로토콜을 구현한 클래스의 인스턴스로, 이 클래스는 앞서 살펴본 매직 메서드 중 최소 한 개 이상을 포함해야한다.

이와 관련하여 다음과 같은 네이밍 컨벤션을 사용한다.  

| 이름            | 의미                                                         |
| --------------- | ------------------------------------------------------------ |
| ClientClass     | 디스크립터 구현체의 기능을 활용할 도메인 추상화 객체<br />디스크립터의 클라이언트<br />클래스 속성(class attribute)으로 디스크립터를 가짐<br />(디스크립터는 DescriptorClass의 인스턴스) |
| DescriptorClass | 디스크립터 클래스<br />디스크립터 프로토콜을 따르는 매직 메서드 중 한 개 이상 구현 |
| client          | ClientClass의 인스턴스<br />client = ClientClass()           |
| descriptor      | DescriptorClass의 인스턴스<br />descriptor = DescriptorClass<br />클래스 속성으로서 ClientClass에 위치 |

> 클래스 속성(class attribute)는 클래스 본문에 정의한 속성으로, 여러 객체가 값을 공유한다.
>
> `self.속성` 형태로 정의한 인스턴스 속성(instance attribute)과 다름

`ClientClass`와 `DescriptorClass`의 관계는 다음과 같이 표현할 수 있는데,  

![descriptor](https://user-images.githubusercontent.com/55227276/173017430-d9d0f757-c470-4494-9058-c67485d64e24.PNG)

이 프로토콜이 정상 동작하기 위해서는 디스크립터 객체가 클래스 속성으로 정의되어야 한다.  



### 디스크립터 메커니즘

디스트립터의 상호작용을 살펴보자.  

먼저 일반적인 클래스의 속성 또는 프로퍼티에 접근하면 다음과 같이 결과를 얻을 수 있다.  

```python
class Attribute:
    value = 42
    
class Client:
    attribute = Attribute()
    
>>> Client().attribute
<__main__.Attribute at 0x7fe5846e6730>
>>> Client().attribute.value
42
```

디스크립터는 이와 조금 다르게 동작한다.  

클래스 속성으로 객체를 선언하면(=디스크립터로 인식되면) 클라이언트에서 속성을 호출할 때 객체 자체를 반환하는 것이 아니라 `__get__` 매직 메서드의 결과를 반환한다.

```python
class DescriptorClass:
    def __get__(self, instance, owner):
        if instance is None:
            return self
        logger.info("Call: %s.__get__(%r, %r)", self.__class__.__name__, instance, owner)
        return instance

class ClientClass:
    descriptor = DescriptorClass()
    
>>> client = ClientClass()
>>> client.descriptor
INFO:Call: DescriptorClass.__get__(<ClientClass object at 0x7fe5846d6f10>, <class 'ClientClass'>)
>>> client.descriptor is client
INFO:Call: DescriptorClass.__get__(<ClientClass object at 0x7fe5846d6f10>, <class 'ClientClass'>)
True
```

이와 같이 디스크립터를 이용하면 완전히 새롭게 프로그램의 제어 흐름을 변경할 수 있다.  

또한 `__get__` 메서드 뒤쪽으로 모든 종류의 논리를 추상화할 수 있으므로 새로운 레벨의 캡슐화라고도 할 수 있다.  



### 디스크립터 프로토콜의 메서드

디스크립터는 객체이기 때문에 프로토콜 메서드들은 self를 첫번째 파라미터로 사용하며,  

여기서 self는 디스크립터 객체 자신을 의미한다.  

그렇다면 디스크립터 프로토콜 메서드 `__get__`, `__set__`, `__delete__`, `__set_name__`에 대해 알아보자.  



#### `__get__(self, instance, owner)`

`__get__` 메서드의 파라미터는 self를 제외하고 2가지가 더 있는데, 

instance는 디스크립터를 호출한 객체로 예제의 client와 같다.  

owner는 해당 객체의 클래스로 client의 클래스인 ClientClass와 같다.  

`__get__` 메서드를 통해 디스크립터가 **1. 클래스에서 호출될 때**와 **2. 인스턴스에서 호출될 때**의 차이점을 알아보자.

```python
class DescriptorClass:
    def __get__(self, instance, owner):
        if instance is None:
            return f"{self.__class__.__name__}.{owner.__name__}"
        return f"value for {instance}"
    
class ClientClass:
    descriptor = Descriptor
    
# 1. 클래스에서 호출
>>> ClientClass.descriptor
'DescriptorClass.ClientClass'
# 2. 객체에서 호출
>>> ClientClass().descriptor
'value for <__main__.ClientClass object at 0x7fe5846e6e80>'
```

> 클래스에서 호출할 경우 네임스페이스와 함께 클래스 이름을 출력하지만  
>
> 객체에서 호출할 경우 디스크립터 자체를 반환한다.  



#### `__set__(self, instance, vlaue)`

`__set__` 메서드는 디스크립터에 값을 할당하려고 할 때 호출된다.  

파라미터 중 instance는 디스크립터를 호출한 객체인 client이고 value는 할당할 값이다.  

`__set__` 메서드를 통해 디스크립터를 활용해 유효성 검증하는 예시를 살펴보자.  

```python
class Validation:
    def __init__(self, validation_function, error_msg: str):
        self.validation_function = validation_function
        self.error_msg = error_msg
        
    def __call__(self, value):
        if not self.validation_function(value):
            raise ValueError(f"{value!r} {self.error_msg}")
            
class Field:
    def __init__(self, *validations):
        self._name = None
        self.validations = validations
        
    def __set_name__(self, owner, name):
        self._name = name
        
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name]
    
    def validate(self, value):
        for validation in self.validations:
            validation(value)
            
    def __set__(self, instance, value):
        self.validate(value)
        instance.__dict__[self._name] = value
        
class ClientClass:
    descriptor = Field(
    	Validation(lambda x: isinstance(x, (int, float)), "는 숫자가 아님"),
        Validation(lmabda x: x >= 0, "는 0보다 작음"),
    )
    
>>> client = ClientClass()
>>> client.descriptor = 42
>>> client.descriptor
42
>>> client.descriptor = -42
ValueError                                Traceback (most recent call last)
...
ValueError: -42 는 0보다 작음
>>> client.descriptor = "invalid value"
ValueError                                Traceback (most recent call last)
...
ValueError: 'invalid value' 는 숫자가 아님
```

> 이 예에서 `__set__()` 메서드가 @property.setter가 하던 역할을 대신하고 있다.  



#### `__delete__(self, instance)`

`__delete__`의 파라미터 중 self는 descriptor 속성을 나타내며 instance는 client를 나타낸다.   

`__delete__` 메서드를 통해 관리자가 권한이 없는 객체에 대해 속성을 제거하지 못하도록 하는 예시를 살펴보자.  

```python
class ProtectedAttribute:
    def __init__(self, requires_role=None) -> None:
        self.permission_required = requires_role
        self._name = None
    
    def __set_name__(self, owner, name):
        self._name = name
        
    def __set__(self, user, value):
        if value is None:
            raise ValueError(f"{self._name}를 None으로 설정할 수 없음")
        user.__dict__[self._name] = value
        
    def __delete__(self, user):
        if self.permission_required in user.permissions:
            user.__dict__[self._name] = None
        else:
            raise ValueError(
            	f"{user!s} 사용자는 {slef.permission_required} 권한이 없음"
            )
            
class User:
    """admin 권한을 가진 사용자만 이메일 주소를 삭제할 수 있음"""
    
    email = ProtectedAttribute(requires_role="admin")
    
    def __init__(self, username: str, email: str, permission_list: list = None) -> None:
        self.username = username
        self.email = email
        self.permissions = permission_list or []
        
    def __str__(self):
        return self.username
    
>>> admin = User("root", "root@d.com", ["admin"])
>>> user = User("user", "user1@d.com", ["email", "helpdesk"])
>>> admin.email
'root@d.com'
>>> del admin.email
>>> admin.email is None
True
>>> user.email
'user1@d.com'
>>> user.email = None
ValueError: email를 None으로 설정할 수 없음
>>> del user.email
ValueError: user 사용자는 admin 권한이 없음
```

> User 클래스는 username과 email파라미터를 필수로 받으며  
>
> email 속성을 지우면 User 클래스에서 정의한 인터페이스와 맞지 않는 상태가 된다.  
>
> User를 사용하고자하는 객체는 email 속성이 있을 것으로 기대하고 있으므로 삭제를 하더라도 None으로 변경해주어야하고, email을 None으로 변경하는 것은 삭제와 같아지므로 금지해야한다.   
>
> 이렇게 하면 특정 권한이 있는 사용자만 특정 행동을 할 수 있다.  



#### `__set_name__(self, owner, name)`

디스크립터 객체를 만들 때 디스크립터가 처리하려는 속성의 이름을 알아야하는데  

파이썬 3.6 이전에는 객체 초기화 시 명시적으로 속성의 이름을 전달하였으나 이를 간편하게 하는 것이 바로 `__set_name__` 메소드이다.  

`__set_name__` 메소드는 파라미터로 디스크립터를 소유한 클래스와 디스크립터의 이름을 받으며,  

디스크립터에 이 메소드를 추가해두면 필요한 이름을 지정할 수 있다.  

아래는 파이썬 3.6 이전의 속성 이름 설정법이다.

```python
class DescriptorWithName:
    def __init__(self, name):
        self.name = name
        
    def __get__(self, instance, value):
        if instance is None:
            return self
        logger.info("%r에서 %r속성 가져오기", instance, self.name)
        return instance.__dict__[self.name]
    
    def __set__(self, instance, value):
        instance.__dict__[self.name] = value
        
class ClientClass:
    descriptor = DescriptorWithName("descriptor")  # 이와 같이 명시적 전달
```

이런 명시적 전달을 파이썬 3.6부터는 아래와 같이 해결할 수 있다.  

```python
class DescriptorWithName:
    def __init__(self, name=None):
        self.name = name
        
	def __set_name__(self, owner, name):
        self.name = name
        
class ClientClass:
    descriptor = DescriptorWithName()
```

  

## 디스크립터의 유형

디스크립터는 그 작동방식에 따라 두 가지로 구분할 수 있다.  

디스크립터가 `__set__`이나 `__delete__`메서드를 구현했다면 **데이터 디스크립터(data descriptor)**라고 부르고   

디스크립터가 `__get__` 만을 구현했다면 **비데이터 디스크립터(non-data descriptor)** 라고 부른다.  

이에 대해 자세히 알아보자.  



### 비데이터 디스크립터

예제를 통해 특징을 살펴보자.  

```python
class NonDataDescriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self
       	return 42
    
class ClientClass:
    descriptor = NonDataDescriptor()
```

위와 같이 비데이터 디스크립터를 만들어놓고 이를 호출하면

```python
>>> client = ClientClass()
>>> client.descriptor
42
```

`__get__` 메서드의 결과를 얻을 수 있게 된다.  

이제 descriptor 속성을 다른 값으로 바꾸면 새로 설정한 값을 얻을 수 있게 된다.

```python
>>> client.descriptor = 43
>>> client.descriptor
43
```

이제 descriptor를 지우고 다시 물으면 어떻게 될까?  

정답은 42를 얻게 된다.  

```python
>>> del client.descriptor
>>> client.decriptor
42
```

이렇게 되는 이유는 비데이터 디스크립터가 객체의 사전 값을 우선적용하고 사전 값이 없을 때 디스크립터를 호출하기 때문이다.  

처음 client 객체를 만들면 descriptor 속성은 클래스 필드안에 있다.(인스턴스 필드X)

따라서 처음 객체를 만들고 client 객체의 사전을 조회하면 다음과 같이 사전이 비어있다.

```python
>>> vars(client)
{}
```

이렇게 객체의 사전에서 호출하고자 하는 속성을 찾을 수 없을 때, 클래스에서 디스크립터를 찾게 되고 이에 따라 처음에 42를 얻을 수 있었다.  

그 다음 .discriptor 속성에 43이라는 다른 값을 설정하면서 객체의 사전이 변경되며 비어있지 않게 된다.   

```python
>>> client.descriptor = 43
>>> vars(client)
{'descriptor': 43}
```

다른 값을 설정하고 속성을 호출하면 객체의 사전에서 descriptor 키를 찾을 수 있어 클래스로 넘어가 디스크립터를 찾기 전에 객체 사전의 값을 반환한다.  

따라서 객체의 사전에서 값을 가져와 43을 얻게 되는 것이다.  

이후 del로 객체의 사전에서  descriptor 키를 삭제하게 되므로, 같은 호출을 할 경우 다시 클래스로 넘어가 디스크립터를 찾아 42를 반환하게 되는 것이다.  

즉, 비데이터 디스크립터는 객체의 사전에 디스크립터와 동일한 이름의 키가 있으면 객체의 사전 값이 적용되고 디스크립터는 호출되지 않는다.



### 데이터 디스크립터

비데이터 디스크립터와의 차이를 살펴보기 위해 예제를 보자.  

```python
class DataDescriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return 42
    
    def __set__(self, instance, value):
        logger.debug("%s.descriptor를 %s 값으로 설정", instance, value)
        instance.__dict__["descriptor"] = value
       
class ClientClass:
    descriptor = DataDescriptor()
```

데이터 디스크립터를 만들어 호출하면 다음과 같은 값을 얻을 수 있다.  

````python
>>> client = ClientClass()
>>> client.descriptor
42
````

그럼 아까와 똑같이 객체 사전을 업데이트 해주면 어떻게 될까?

```python
>>> client.descriptor = 99
>>> client.descriptor
42
```

비데이터 디스크립터와 달리 데이터 디스크립터는 descriptor 의 반환값이 변경되지 않았다.  

대신 객체의 사전은 업데이트 되었다.  

```python
>>> var(cliebt)
{'discriptor': 99}
>>> client.__dict__["descriptor"]
99
```

이렇게 되는 이유는 `__set__` 메서드가 호출되면 객체의 사전에 값을 설정하기 때문이다.  

즉, 데이터 디스크립터에서는 디스크립터와 동일한 이름을 갖는 키가 객체의 사전에 존재하더라도 디스크립터 자체가 항상 먼저 호출된다. 

따라서 다음이 동작하지 않는다.  

```python
>>> del client.descriptor
AttributeError: __delete__
```

왜냐하면 디스크립터를 항상 먼저 호출하기 때문에 del작업을 수행하기 위해서는 데이터 디스크립터 내에 `__delete__` 메서드가 구현되어있어야하는데, 보다시피 구현되어있지 않기 때문이다.

이전의 비데이터 디스크립터 예제에서도 `__delete__` 메서드를 구현하지 않았지만 del이 정상시행된 이유는  

del이 `__delete__`메서드가 실행된 것이 아닌 객체 사전에서 속성을 지웠기 때문에 (dict의 del) 가능했던 것이다.

   

이것이 데이터 디스크립터와 비데이터 디스크립터의 차이이다.  

기억해야할 것은 **`__set__`메서드가 디스크립터에 구현되면, 디스크립터가 객체의 사전보다 높은 우선순위를 갖는다**는 것이다.  

   

## 디스크립터 실전

이제 우리는 디스크립터가 무엇인지, 어떻게 작동하는지 등을 알아보았다.  

그렇다면 이제 디스크립터를 통해 어떤 멋진 일들을 할 수 있는지 알아보자.  

  

### 디스크립터를 사용한 애플리케이션

디스크립터를 사용하면 중복 코드를 추상화하여 클라이언트의 코드가 혁신적으로 줄어드는 것을 확인할 수 있다.  

그렇다면 먼저 디스크립터를 사용하지 않은 예제를 보자.  

```python
class Traveller:
    def __init__(self, name, current_city):
        self.name = name
        self._current_city = current_city
        self._cities_visited = [current_city]
        
    @property
    def current_city(self):
        return self._current_city
    
    @current_city.setter
    def current_city(self, new_city):
        if new_city != self._current_city:
            self._cities_visited.append(new_city)
            self._current_city = new_city
            
    @property
    def cities_visited(self):
        return self._cities_visited
```

> 클래스는 여행자이며 현재 어느 도시에 있는지를 속성으로 가진다. 또한 방문한 모든 도시를 추적한다.

```python
>>> alice = Traveller("Alice", "Barcelona")
>>> alice.current_city = "Paris"
>>> alice.current_city = "Brussels"
>>> alice.current_city = "Amsterdam"
>>> alice.cities_visited
['Barcelona', 'Paris', 'Brussels', 'Amsterdam']
```

프로퍼티로 원하는 내용들을 충분히 구현해내었다.  

그런데 어플리케이션의 여러 곳에서 구입한 티켓 추적 등 다양한 것을 추적한다면 같은 코드가 반복되어야 할 것이다.  

또한 다른 클래스에서 같은 로직을 사용하려면 데코레이터나 프로퍼티 빌더 등을 사용하지 않고는 마찬가지로 같은 코드를 반복해야할 것이다.  

그렇다면 디스크립터를 통한 이상적인 구현을 확인해보자.  

사실 예제에 명시되지 않은 요구사항까지 구현에 포함하여 필요이상의 기능을 제공하지만

실전에서의 디스크립터 사용방법을 묘사하기 위한 구현임을 알아두자.  

```python
class HistoryTracedAttribute:
    def __init__(self, trace_attribute_name) -> None:
        self.trace_attribute_name = trace_attribute_name
        self._name = None
        
    def __set_name__(self, owner, name):
        self._name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__[self._name]
    
    def __set__(self, instance, value):
        self._track_change_in_value_for_instance(instance, value)
        instance.__dict__[self._name] = value
        
    def _track_change_in_value_for_instance(self, instance, value):
        self._set_default(instance)
        if self._needs_to_track_change(instance, value):
            instance.__dict__[self.trace_attribute_name].append(value)
            
    def _needs_to_track_change(self, instance, value) -> bool:
        try:
            current_value = instance.__dict__[self._name]
        except KeyError:
            return True
        return value != current_value
    
    def _set_default(self, instance):
        instance.__dict__.setdefault(self.trace_attribute_name, [])
        
class Traveller:
    
    current_city = HistoryTracedAttribute("cities_visited")
    
    def __init__(self, name, current_city):
        self.name = name
        self.current_city = current_city
```

> 디스크립터에 `current_city`라는 변수를 할당하고, `cities_visited`라는 속성의 이름을 지어주었다.   
>
> 디스크립터 최초 호출 시 추적값이 존재하지 않으므로  `_set_default`를 통해 빈 배열로 초기화해주며  
>
> 마찬가지로 `_needs_to_track_change`를 통해 방문지도 초기화 및 다를 때만 추가하도록 한다.  
>
> 이제 Traveller의 `__init__`에서 디스크립터가 생성된다.  

  

방금의 예제에서 디스크립터를 사용한 것은 아래와 같은 장점이 있다. 

- 클라이언트의 코드가 상당히 간단해진다.

- 어떤 비지니스 로직도 포함되어있지 않기 때문에 완전 다른 클래스에서도 사용이 가능하다.

(디스크립터는 비지니스 로직의 구현보다는 라이브러리, 프레임워크 또는 내부 API를 정의하는데 적합하다.)  

  

### 생각해볼 것

**1. 아래의 코드에 대해 잠깐 생각해보자.**

```python
instance.__dict__["descriptor"] = value
```

우리는 지금까지 인스턴트의 `__dict__`에 직접 접근하여 key와 value를 설정해주었는데  

파이썬의 `setattr()`을 활용하여 다음과 같이 간단히 할 수는 없었을까?  

```python
setattr(instance, "descriptor", value)
```

이 방법이 불가능한  이유는 디스크립터의 속성에 무언가를 할당하려고 하면 `__set__` 메서드가 호출되기 때문이다.   

`setattr()`이 `__set__` 메서드를 호출하고 `__set__`메서드가 다시 ` setattr()`을 호출하면서 무한루프에 빠지게 된다.  

따라서 `setattr()`을 활용하여 간단히 하는 것은 불가능하다.  

**2. 다음에 대해서도 생각해보자.**

디스크립터는 왜 모든 인스턴스의 프로퍼티 값을 보관할 수 없을까?  

그 이유는 바로 `순환 종속성(circular dependencies)`이 생기기 때문이다.  

클라이언트 클래스가 이미 디스크립터의 참조를 가지고 있기 때문에 디스크립터가 다시 클라이언트 객체를 참조할 경우 순환 종속성이 생기는 것은 불가피하다.  

이는 약한 참조를 사용하여 어느정도 해소할 수 있다.  

  

방금 같이 생각해본 두 가지를 염두에 두고 디스크립터의 특성에 관련된 문제를 이해해보자.



### 전역 상태 공유 이슈

디스크립터는 반드시 클래스 속성으로 설정해야한다는 것은 이제 모두가 아는 사실이다.  

그런데 클래스 속성의 문제점은 이들이 해당 클래스의 모든 인스턴스에서 공유된다는 것이다.  

따라서 디스크립터 객체에 데이터를 보관하면 모든 객체가 동일한 값에 접근할 수 있게 된다.  

그럼 다음과 같은 이슈가 생길 수 있다.  

```python
class SharedDataDescriptor:
    def __init__(self, initial_value):
        self.value = initial_value
        
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.value
    
    def __set__(self, instance, value):
        self.value = value
       
class ClientClass:
    descriptor = SharedDataDescriptor("첫 번째 값")
    
>>> client1 = ClientClass()
>>> client1.descriptor
'첫 번째 값'
>>> client2 = ClientClass()
>>> client2.descriptor
'첫 번째 값'
>>> client2.descriptor = "client2를 위한 값"
>>> client2.descriptor
'client2를 위한 값'
>>> client1.descriptor
'client2를 위한 값'
```

> 이렇게 한 객체의 값을 변경하면 모든 객체의 값이 변경되는 이유는 `ClientClass.descriptor` 가 고유하기 때문

이런 문제를 해결하기 위해서는 데이터를 바로 저장하는 것이 아니라 각 인스턴스의 값을 보관했다가 반환하는 형식이어야한다.  

이는 `__dict__`, `setattr()`, `getattr()`을 이용하여 구현할 수 있는데,  

생각해 볼 것에서 살펴본 바와 같이 `setattr()`, `getattr()`는 사용할 수 없으므로 우리는 `__dict__`를 불가피하게 사용해야한다.  



### 약한 참조 사용

이미 앞서 디스크립터는 객체의 사전에 값을 저장 및 조회해야함에 대해 이야기하였다.  

그런데 `__dict__`를 사용하지 않으려는 경우애는 어떤 대안이 있을까?  

그 방법은 디스크립터 객체가 직접 내부 매핑을 통해 각 인스턴스의 값을 보관하고 반환하는 것이다.  

하지만 이 방법은 앞서 생각해볼 것에서 언급한대로 순환 종속성이 생길 수 있다.  

따라서 파이썬의 `weakref` 라이브러리를 활용해 약한 참조를 사용해야한다.  

한 가지 주의할 점은 객체가 `__hash__` 메서드 구현이 가능해야한다는 것이다.  

(만약 가능하지 않다면 `WeakKeyDictionary`에 매핑할 수 없기 때문)

구현해보자면 다음과 같겠다.  

```python
from weakref import WeakKeyDictionary

class DescriptorClass:
    def __init__(self, initial_value):
        self.value = initial_value
        self.mapping = WeakKeyDictionary()
        
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.mapping.get(instance, self.value)
    
    def __set__(self, instance, value):
        self.mapping[instance] = value
```

> 단, 이렇게 될 경우 인스턴스 객체는 더이상 속성을 보유하지 않으며 대신 디스크립터가 속성을 보유한다.  
>
> (다소 논란의 여지가 있으며 개념적 관점에서 완전히 정확하지 않을 수 있다.)



### 디스크립터 사용의 장점

#### 코드 재사용

처음부터 이야기한 바와 마찬가지로 디스크립터는 코드 중복을 피하기 위한 강력한 추상화 도구이다.  

디스크립터는 프로퍼티가 필요한 구조가 반복되는 곳에 사용하기 좋다.  

프로퍼티는 디스크립터의 특수한 개념으로 디스크립터가 더 다양하고 복잡한 작업에 사용될 수 있다.  

디스크립터는 또 다른 재사용 도구인 데코레이터의 `3의 규칙(Three instance rule)`을 적용한다.  

디스크립터는 비지니스 코드가 아닌 구현 코드가 많이 포함되어야하며, 비지니스 로직에서 사용할 새로운 객체나 데이터 구조를 정의하는 것과 비슷하다.  



#### 클래스 데코레이터 피하기

"데코레이터를 사용해 코드개선하기" 에서 이벤트 객체의 직렬화 방식을 결정하기 위해 아래와 같이 두 개의 클래스 데코레이터를 사용하여 구현했었다.  

```python
@Serializaion(
    username=show_original,
    password=hide_field,
    ip=show_original,
    timestamp=format_time,
)
@dataclass
class LoginEvent:
    username: str
    password: str
    ip: str
    timestamp: datetime
```

이는 디스크립터를 사용하여 아래와 같이 바꿀 수 있다.  

```python
from functools import partial
from typing import Callable

class BaseFieldTransformation:
    def __init__(self, trasformation: Callable[[], str]) -> None:
        self._name = None
        self.trasformation = transformation
        
    def __get__(self, instance, owner):
        if instance is None:
            return self
        raw_value = instance.__dict__[self._name]
        return self.transformation(raw_value)
    
    def __set_name__(self, owner, name):
        self._name = name
        
    def __set__(self, instance, value):
        instance.__dict__[self._name] = vlaue
        
ShowOriginal = partial(BaseFieldTransformation, transformation=lambda x: x)
HideField = partial(
    BaseFieldTransformation, transformation=lambda x: "**민감한 정보 삭제**"
)
FormatTime = partial(
    BaseFieldTransformation, transformation=lambda ft: ft.strftime("%Y-%m-%d %H:%M")
)

class LoginEvent:
    username = ShowOriginal()
    poassword = HideField()
    ip = ShowOriginal()
    timestamp = FormatTime()
    
    def __init__(self, username, password, ip, timestamp):
        self.username = username
        self.password = password
        self.ip = ip
        self.timestamp = timestamp
        
    def serialize(self):
        return {
            "username": self.username,
            "password": self.password,
            "ip": self.ip,
            "timestamp": self.timestamp,
        }
```

또한 `__init]__()`과 `serialize()` 메서드를 구현한 `BaseEvent`클래스를 만들고 그것을 상속받으면  

```python
class LoginEvent(BaseEvent):
    username = ShowOriginal()
    password = HideField()
    ip = ShowOriginal()
    timestamp = FormatTime()
```

 이렇게 더 깔끔하게 클래스를 작성할 수 있다.  

결과적으로 각 이벤트 클래스는 더 작고 간단해지며 클래스 데코레이터보다 훨씬 간단하고 이해가 쉽다.  

  

## 파이썬 내부에서의 디스크립터 활용

그렇다면 파이썬에서는 디스크립터를 어떻게 활용하고 있을까? 몇 가지를 살펴보자. 



### 함수와 메서드

파이썬의 함수 역시 디스크립터 객체에 해당한다.  

함수는 기본적으로 `__get__` 메서드를 구현하였기 때문에 클래스 안에서 메서드처럼 동작할 수 있다.  

```python
class MyClass:
    def method(self, ...):
        self.x = 1
```

이런 메서드는 실제로 다음과 같이 정의하는 것과 같다.

```python
class MyClass: pass

def method(myclass_instance, ...):
    myclass_instance.x = 1
    
method(MyClass())
```

메서드는 객체를 수정하는 또 다른 함수일 뿐이며 객체 안에서 정의되어 객체에 바인딩되어있다고 말한다.  



### 메서드를 위한 빌트인 데코레이터

공식문서에 설명된 바와 같이 `@property`, `@classmethod`, `@staticmethod` 데코레이터는 디스크립터이다.  

`@property`를 클래스에서 직접 호출하면 계산할 속성이 없으므로 프로퍼티 객체 자체를 반환하며  

`@classmethod`를 사용하면 데코레이팅 함수에 첫 번째 파라미터로 메서드를 소유한 클래스를 넘겨주고  

`@staticmehtod`를 사용하면 정의한 파라미터 이외의 파라미터를 넘기지 않도록 한다.  



### 슬롯

파이썬의 `__slots__` 매직 메서드를 사용하면 클래스가 기대하는 특정 속성만을 정의하며, 정의되지 않은 동적 속성에 대해 AttributeError를 발생시킨다.  

한 마디로 클래스가 정적이 되고 더이상 `__dict__`속성을(=동적 속성을) 갖지 않게 된다.  

이렇게 객체의 사전이 없어지게 될 때, 속성을 불러오는 방법으로 디스크립터를 택하고 있다.  

슬롯에 정의된 이름마다 디스크립터를 만들어 값을 저장하는 방식으로 말이다.  

슬롯을 이용한 객체는 고정된 필드 값만 저장하면 되므로 메모리를 덜 사용한다는 장점이 있지만,  

파이썬의 동적인 특성을 없애므로 사용에 유의해야한다.  



## 데코레이터를 디스크립터로 구현하기

사용자 정의 데코레이터를 개발하면서 발생하는 문제를 디스크립터로 해결할 수 있다.  

데코레이터를 디스크립터로 만들기 위한 일반적인 방법은 `__get__ `메서드를 구현하고  

`types.MethodType`을 사용해 데코레이터 자체를 객체에 바인딩된 메서드로 만드는 것이다.  

유의할 점은 데코레이터를 객체로 구현하거나 클래스로 정의해야한다는 점이다.  

함수는 이미 `__get__`메서드가 존재하므로 함수로 구현 시 데코레이터가 정상적으로 동작하지 않을 수 있다.  
