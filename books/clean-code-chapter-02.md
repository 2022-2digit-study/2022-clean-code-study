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

