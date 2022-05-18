# [Chapter .2] 파이썬스러운(Pythonic) 코드

## Pythonic 하다는건 뭘까?

프로그래밍에서 관용구(idiom)는 특정 작업을 수행하기 위해 코드를 작성하는 특별한 방법이다.

이 관용구는 언어에 따라 다른데, 파이썬의 고유한 메커니즘을 따른 코드를 `파이썬스럽다`고 한다.



### Pythonic한 코드를 작성해야하는 이유?

일반적으로 관용적인 방식으로 코드를 작성했을 때 성능이 좋고 코드도 짧으며 이해가 쉽다.

또한 동일한 패턴과 구조에 익숙해지면 실수를 줄이고 본질에 집중할 수 있다.

~~(대충 다 좋다는 얘기)~~



그렇다면 pythonic한 코드의 근본 아이디어들에 대해 살펴보자.



## 인덱스와 슬라이스

### 파이썬의 인덱싱

파이썬의 일부 데이터나 구조에서 요소에 접근할 때 인덱스를 활용할 수 있다.

그런데 파이썬은 다른 언어와 달리 `음수`를 사용하여 `배열의 끝에서 부터 접근`이 가능하다.

```python
nums = (1, 1, 2, 3, 5, 8, 13, 21)

>>> nums[-1]
21
>>> nums[-5]
3
```



### 파이썬의 슬라이싱

또한 파이썬은 `slice`를 사용하여 `특정 구간의 요소`를 구할 수 있다.

슬라이싱에는 시작 인덱스, 끝 인덱스, 간격을 설정해줄 수 있다.

단, 슬라이스의 시작 인덱스는 포함되지만 끝 인덱스는 제외되며

시작 인덱스나 끝 인덱스를 작성하지 않을 경우 처음이나 끝으로 동작한다.

```python
nums = (1, 1, 2, 3, 5, 8, 13, 21)

>>> nums[2:4]
(2, 3)
>>> nums[:-3]
(1, 1, 2, 3, 5)
>>> nums[::2]
(1, 2, 5, 13)
```



### 자체 시퀀스 생성

이런 파이썬의 인덱싱과 슬라이싱은 `__getitem__`이라는 매직 메서드로 동작한다.

`__getitem__`은 `myobject[key]`와 같은 형태를 사용할 때 사용되는 메서드이다. 

특히 `__getitem__`과 `__len__`을 사용하여 시퀀스나 이터러블 객체를 만들지 않고 키를 통해 객체의 특정 요소를 가져올 수 있다.

~~(이거 때문에 이름이 자체시퀀스인듯..)~~

[^Magic Method]: 클래스 안에 정의할 수 있는 스페셜 메소드, 클래스를 파이썬의 빌트인 타입과 같은 동작을 하도록 해줌, 메소드 이름 앞뒤에 더블 언더스코어를 붙임



## 컨텍스트 관리자

컨텍스트 관리자(Context Manager)는 크게 두 가지 경우에 유용한 파이썬의 기능이다. 



### 리소스 관리

일반적으로 파일이나 소켓 연결을 열었을 때 할당된 리소스를 모두 해제해주어야 하는데, 

이 때 생각하지 못한 예외나 오류가 발생할 수도 있다.

이를 사전에 모두 처리하는 것은 어렵지만, 파이썬에서는 `with문`(PEP-343)을 사용해 파이썬스럽게 구현할 수 있다.

```python
with open(filename) as fd:
    process_file(fd)
```

블록의 마지막이 실행되고 나면 컨텍스트가 종료되며, 오류가 있더라도 종료되므로 안전하게 실행할 수 있다.



### 코드 분리

주요 동작의 전후에 작업을 실행하려고 할 때나 독립적으로 코드를 분리해야 할 때 사용할 수 있다.



### 컨텍스트 관리자 구현

컨텍스트 관리자는 일반적으로 `__enter__`와 `__exit__` 두 개의 매직 메서드만 구현하면 되지만, contextlib 모듈을 사용하여 더 쉽게 구현할 수도 있다.

`__enter__` 메서드가 호출되면 새로운 컨텍스트로 진입하게 되며

컨텍스트의 마지막 문장이 끝나면 컨텍스트가 종료되며 처음 호출한 컨텍스트 관리자 객체의 `__exit__` 메서드를 호출한다.

```python
class dbhandler_decorator(contextlib.ContextDecorator):
    def __enter__(self):
        stop_database()
    
    def __exit__(self, ext_type, ex_value, ex_traceback):
        start_database()
        
@dbhandler_decorator()
def offline_backup():
    run("pg_dump database")
```

> 이렇게 컨텍스트 관리자를 데코레이터로 지정해주게 되면 
>
> offline_backup함수를 호출만 하더라도 컨텍스트 관리자 안에서 자동으로 실행됨



## 프로퍼티, 속성과 객체 메서드의 다른 타입들

프로퍼티에 대해 알아보기 전에 파이썬에서의 밑줄에 대해 잠깐 살펴보고 가자.



### 파이썬에서의 밑줄

일반적으로 파이썬에서의 변수나 메소드 이름 앞의 밑줄은 private를 의미한다.

_(단, private를 의미만 할 뿐 private하게 만들 수는 없다. 자세한 내용은 뒤에 설명)_

그런데 밑줄 두 개는 전혀 다른 `name mangling`이라는 것을 실행한다.

`name mangling`이란 말그대로 이름을 만드는 것인데, `_<class-name>__<attribute-name>` 형태의 이름을 만든다.

이런 이름을 만드는 이유는 **클래스가 여러 번 확장되더라도 충돌 없이 오버라이드를 하기 위한 것**이다.

따라서 가끔 `name mangling`이 전혀 다른 이름을 만들어내어 본래의 이름으로는 해당 속성에 접근할 수 없게 되기 때문에 

**밑줄 두 개를 작성하는 것이 정말 private하게 만든다고 생각하는 사람이 있는데 이는 매우 잘못된 생각이다.**

~~(이중 밑줄은 파이썬스러운 코드가 아니라고 한다)~~

```python
class Connector:
    def __init__(self, source):
        self.source = source
        self._timeout1 = 60
        self.__timeout2 = 120
    
    def connect(self):
        print(f"connecting with {self._timeout}s")
        
conn = Connector("postgresql://localhost")
>>> conn.connect()
connecting with 60s
>>> conn._timeout1
60
>>> conn.__timeout2
AttributeError: 'Connector' object has no attribute '__timeout2'
>>> conn._Connector__timeout2
120
```



### 프로퍼티

#### 프로퍼티 사용

프로퍼티는 아래의 두 경우 처럼 객체의 어떤 속성에 대한 접근을 제어하기 위해 사용한다.

- 객체에 값을 저장해야 하는 경우
- 객체의 상태나 다른 속성의 값으로 어떤 계산을 하려고 하는 경우

타 언어에서 getter-setter를 만드는 것과 동일하다.

`@property` 데코레이터는 일반적인 getter와 역할이 같으며

`@<property>.setter`데코레이터는 setter와 역할이 같다.

```python
class User:
    def __init__(self, username):
        self.username = username
        self._email = None
       
    @property
    def email(self):
        return self._email
    
    @email.setter
    def email(self, new_email):
        if not is_valid_email(new_email):
            raise ValueError("유효한 이메일이 아니므로 사용할 수 없음.")
       	self._email = new_email
        
>>> user = User("Erin")
>>> user.email = "erin"
유효한 이메일이 아니므로 사용할 수 없음.
>>> user.email = "erin@2digit.io"
>>> user.email
'erin@2digit.io'
```



#### 프로퍼티의 장점

프로퍼티를 사용하면 명령-쿼리 분리 원칙(command and query separation)을 따르기도 용이하다.

`@property` 데코레이터는 응답하기 위한 query이며

`@<property>.setter`데코레이터는 무언가를 하기 위한 command이다. 

[^Command Query Separation(CQS)]: Betrand Meyer에 의해 정리된 개념. 질의(query)와 명령(command)을 정확히 분리하는 것을 목적으로 함, 질의는 결과만 반환하고 상태는 변화시키지 않으며 명령은 결과는 반한하지 않고 상태만 변경시킴



### 파이썬에서의 프로퍼티

일반적인 프로그래밍 언어는 public, private, protected 세 가지 프로퍼티를 가지지만

**파이썬은 모든 프로퍼티와 함수가 public하다.** 따라서 호출자가 **모든 객체의 속성을 호출할 수 있다.**

`밑줄(언더스코어)`을 사용하여 다른 언어처럼 private를 의미할 수 있지만, 여전히 호출은 할 수 있다.



## 이터러블 객체

이터러블 객체를 살펴보기에 앞서서 이터러블과 이터레이터를 구분해보자.

- 이터러블: `__iter__` 매직 메서들을 구현한 객체
- 이터레이터: `__next__` 매직 메서드를 구현한 객체

~~(자세한 내용은 9회차에 기린님이 멋지게 다뤄주실 예정)~~



파이썬은 기본적으로 반복가능한 리스트, 튜플 등이 있다.

`for e in myobject:` 형태로 객체를 반복할 수 있는지 확인하기 위해

객체가 `__iter__`나 `__next__` 중 하나를 포함하는지와 

객체가 시퀀스이고 `__len__`과 `__getitem__`를 모두 가졌는지를 검사한다.

## 컨테이너 객체

`__contains__()` 메서드를 구현한 객체, 일반적으로 Boolean값을 반환하도록 구현

해당 키워드는 `in` 키워드가 발견될 때 호출됨.

### 사용 예
x, y를 멤버변수로 갖고 있는 coord가 그리드의 영역에 있는지 검사하고 표시하고 싶을 때, 일반적인 구현은 다음과 같음

```python
# less Pythonic
def mark_coordinate(grid, coord):
    if 0 <= coord.x < grid.width and 0 <= coord.y < grid.height:
        grid[coord] = MARKED
```
▲ 직관적으로 x, y가 무엇을 하는 지 알아보기 힘듦


```python
# more Pythonic
class Boundaries:
    def __init__(self, width, height):
        self.width = width
        self.height = height
       
    def __contains__(self, coord):
        x, y = coord
        return 0 <= x < self.width and 0 <= y < self.height
        

class Grid:
    def__init__(self, width, height):
        self.width = width
        self.hegiht = height
        self.limit = Boundaries(width, height) # 의도를 직관적으로 설명하였음.
    
    def __contains__():
        return coord in self.limits

```

```python
# Usage
def mark_coordinate(grid, coord):
    if coord in grid:
        grid[coord] = MARKED
```
▲ in절을 통해 직관적으로 Grid 안에 있는지 체크하는 듯한 느낌을 받을 수 있음.

### 장점
1. 외부에서 사용할 때 해당 코드들은 마치 파이썬이 문제를 해결한 것처럼 보임
2. 구성이 간단하고 위임을 통해 문제를 해결함 (객체들이 모두 최소한의 논리를 사용)

## 객체의 동적 속성

동적으로 어떤 값을 자료 구조에 담고 싶다면 그에 좋은 객체가 이미 파이썬에 존재한다.
바로 Dictionary이다

```python
>>> car_info = dict()
>>> dict['color'] = 'red'
>>> dict['name'] = 'Ferarri'
>>> car_info['color']
'red'
>>> car_info['name']
'Ferarri'
```
딕셔너리는 불변의 값, 즉 수정이 불가능한 자료를 키로 사용하여 자료를 저장하는 해쉬맵 자료구조이다.(파이썬의 딕셔너리는 매우 효율적이라고 알려져 있다.)

> 특히 동적인 자료구조를 저장하는데 있어 딕셔너리(≒ Hash Map)의 경우 다른 자료구조에서 거의 유일하게 키(key)라는 값을 이용해서 사용자가 제어할 수 있는 자료구조이니 알아두는 편이 좋다.

단 위의 예시는 사물을 구현한 것이기에 딕셔너리에 저장하기 보단, 다음과 같이 클래스로 구현하면 여러 메소드를 구현 할 수 있을 것이다.

```python
class Car:
    def __init__(self, name, color):
        self.name = name
        self.color = color
 
my_car = Car("Ferarri", "red")
print( f" name: {my_car.name} color: {my_car.color}")
```

클래스로 정의하니 확실히 가독성이 좋아졌고, 메서드의 확장도 충분히 가능해보인다. 그렇다면 인스턴스의 멤버변수는 파이썬이 어떻게 알아낼까? 

### \_\_getattr\_\_

my_car.name을 호출하면 파이썬은 객체의 사전에서 name을 찾아서 `__getattribute__`를 호출한다. 
객체에 찾고 있는 속성이 없는 경우 속성 이름을 파라미터로 전달하여 `__getattr__('name')`을 호출한다.<br>
이 값을 사용하면 반환 값을 제어할 수도 있고, 심지어는 새로운 속성을 만들어 낼 수 있다.<br>

```python
class DynamicAttributes:
    def __init__(self, attribute):
        self.attribute = attribute
        
    def __getattr__(self, attr):
        if attr.startswith("fallback_ "):
            name = attr.replace( "fallback_", "")
            return f"(fallback resolved] {name}"
        raise AttributeError(f"{self.__class__ .__name__}에는 {attr} 속성이 없음 . ")
```
#### 호출
```python
>>> dyn = DynamicAttributes("value") 
>>> dyn.attribute
'value'
```
▲ attribute 호출

```python
>>> dyn.fallback_test 
'[fallback resolved] test'
```
▲ 멤버변수에 없는 값을 호출
> 내부적으로 `dyn.__getattr__('fallback_test')`가 호출되어 처리된다.

```python
>>> dyn.__dict__["fallback_new"] = "new value" 
>>> dyn.fallback_new
'new value'
```
▲ 새로운 멤버변수를 동적으로 생성하기

```python
>>> getattr(dyn, "something" , "apple" ) 
'apple'
```
`__getattr__`은 파이썬 내장 함수 `getattr()`에도 영향을 미친다.
> 파이썬 내장함수 `getattr(instance, attribute_name, default_value)`는 `instance`의 멤버변수에 `attribute_name` 이라는 이름을 가진 멤버변수가 있는지를 검사하고, 없는 경우 `default_value`를 반환하는 함수이다.

내장함수 `getattr`은 `dyn.something`을 호출하고, `__getattr__('something')`을 실행시킨다. 단, `something`은 `fallback_`으로 시작하지 않기 때문에 `AttributeError`가 발생하고, `getattr`은 `AttributeError`를 감지하여 `default_value`로 설정된 `apple`를 반환한다. 

```python 
>>> getattr(dyn, "fallback_hello" , "apple" ) 
'(fallback resolved] hello'
```
▲ `if attr.startswith("fallback_"):` 의 조건을 통과하였기 때문에 값을 반환한 `__getattr__`함수는 `AttributeError`를 야기하지 않는다.

## 호출형(callable) 객체

클래스 명 뒤에 괄호`()`를 붙이면 생성자 메서드`__init__()`이 실행되며, 결과물인 `instance`를 반환한다.

`__call__` 함수를 구현하면 `instance` 변수 뒤에 괄호가 올 경우 동작을 정의할 수 있다.

```python
from collections import defaultdict
class CallCount:
    def __init__(self):
        self._counts = defaultdict(int)
    def __call__(self, argument): 
        self._counts[argument] += 1 
        return self._counts[argument]
```

#### 호출

```python
>>> cc = CallCount() 
>>> cc(1)
1
>>> cc(2)
1
>>> cc(1) 
2
>>> cc(1) 
3
>>> cc("something")
1
```
▲ `CallCount` 클래스의 인스턴스 변수 `cc`를 통해 `__call__()` 함수를 호출

> 후반부에 데코레이터 생성 시 해당 메서드를 구현하면 더 편리함을 알게 될 것이다. 

## 파이썬에서 피해야 할 점
해당 부분은 비교적 쉽게 발견할 수 있다. 많은 IDE에서 이를 경고하고 있기 떄문이다.

### 변경 가능한(mutable)파라미터 기본 값(⭐️)
파라미터에 디폴트 값을 설정하는 것은 매번 파라미터를 설정해야 하는 개발자의 피로를 줄여준다. 다만 수정이 가능한 변수로 설정하면 문제가 될 수 있다. 정확히는 값을 수정할 수 있는 변수라고 하겠다.
```python
def say_name(name = "Giraffe"):
    print(name)
```
이 함수에서 문제될 것이 있는가? C++을 배운 혹자는 `string`은 문자 하나를 담는 자료형 `char`의 포인터 형태이기 때문에 mutable 하지 않냐고 반문할 수 있다.<br>
_(C++ 포인터는 특정 값에 접근이 가능하며, 수정 역시 가능하다.)_<br>
<br>
하지만 답은 문제될 것이 없다. <br>

#### (+ɑ) 왜?
왜냐하면 `name[0] = "R"` 과 같은 조작이 파이썬 문자열에서 불가능하기 때문이다. Python은 C++을 사용하여 만들어졌으며 python `str` 역시 C++ `string`의 확장이다. C++ `string`의 경우 `const char *` 타입의 포인터 변수로 구현되어 있는데 `const`는 상수의 선언이며, 자체로 수정 불가능한 값을 만든다. 따라서 이는 **선언과 동시에 이루어지는 초기화 이외의 값 변경을 금지하는 상수의 선언**을 뜻한다.<br>
<br>
**따라서 문자열은 주소가 없는 자료형(말 그대로 값)을 의미하는 원시 자료형(Primitive Type)처럼 취급된다.**

```python
def say_shopping_cart(cart: list = []):
    ...
    if "apple" not in cart:
        print("사과는 꼭 사야하니 추가합니다.")
        cart.append("apple")
    ...
    
    # 쇼핑카트 출력
    print(cart)
```

이 코드는 앞서 말한 코드와는 조금 다르다. cart는 디폴트로 빈 리스트를 담고 있다. 하지만 앞서 문자열을 설명한 것과는 달리 리스트는 일반 포인터이다. 포인터는 주소값을 가르키며, **함수가 종료되어도 사라지지 않는다.** <br>

> _포인터를 제대로 이해하기 위해서는 힙메모리, 스텍 메모리 등 함수의 메모리 구조까지 알아야 하기 때문에 클린 코드를 배울 정도라면 이정도는 알 것이라 여기고 설명을 생략한다._

포인터를 담는 메모리의 특성 덕분에 매개변수로 선언된 리스트 변수 cart역시 함수가 종료되어도 살아있으며, 다시 함수를 호출했을 때 호출한 함수의 매개변수로 default값을 사용한다면, 다시 함수가 호출되었을 때 아래와 같이 문제가 될 수 있다.<br>

#### 1. 첫 호출
```python
>>> say_shopping_cart()
'사과는 꼭 사야하니 추가합니다.'
['apple']
```
#### 2. 재 호출
```python
>>> say_shopping_cart()
['apple']
```
> 원래 1, 2번의 출력이 같아야 하나 다르다.

1번에서는 default로 `cart`에는 빈 리스트(`[]`)가 초기화 되고 함수 종료시에는 `cart`에는 `'apple'` 문자열이 담긴 리스트로 끝이 났다. 재호출 시 cart의 포인터가 바뀌지 않았기 떄문에 default로 
`['apple']`이 초기화 된 배열이 들어가며, 예상치 못한 결과가 생긴다.<br>
<br>
따라서 개발자는 항상 매개변수의 default에는 변경 불가능한 값을 선언하는 것이 좋다 _(예시와 같은 경우 `None`으로 default를 설정하는 것이 좋다.)_

### 내장(built-in) 타입 확장

파이썬 내장 빌트인 타입은 클래스의 메서드를 서로 호출하지 않기 때문에 하나를 오버라이드하면 나머지에는 반영되지 않아서 예기치 못한 결과가 발생한다.<br>
<br>
**따라서 dict, list, set과 같은 파이썬 내장 빌트인 클래스를 직접 확장하는 것은 피하도록 하자.**
