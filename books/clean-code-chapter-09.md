# [Chapter. 9] 일반적인 디자인 패턴

디자인 패턴은 갱 오브 포(Gang of Four - GoF : Design Patterns : Reusable Object — Oriented Software)로 유명한 책의 소개와 함께 소프트웨어 공학에 널리 보급되어왔다.

디자인 패턴 자체가 무조건 이를 따라야 하는 교본 같은 아니다. 그러나 자주 하는 디자인 패턴에는 이유가 있는 법이다. 

또한 이는 특정 프로그래밍 언어에 종속된 것이 아니다. 오히려 객체를 사용하는 모든 어플리케이션에서 상호작용하는 방법에 관한 보다 일반적인 개념에 가까울 것이다.

## 3가지 패턴

디자인의 패턴은 크게 생성(Creational), 구조(Structural), 행동(Behavioral) 패턴 중 하나로 분류된다. 
> 이것의 확장 버전이나 변형버전의 패턴 역시 존재하나, 다루지 않는다.

### 염두할 점
**모든 패턴을 알 필요는 없다.** 

이는 아래의 두 가지 이유를 따른다.
1. 이미 파이썬에서 보이지 않는 단계에서 구현된 상태이기 때문이다.
> 따라서 꼭 보이지 않더라도 적절히 적용될 수 있다.
2. 모든 패턴이 일반적인 것은 아니기 때문이다.
> 특별한 경우에만 사용되는 디자인 패턴이 존재한다.

따라서 소개할 패턴은 가장 자주 `출현`하는 디자인 패턴을 살펴본다.

#### 출현
앞서 출현이라는 말을 썼다. 이는 나타나지 않을 수 있다는 말이며, 명심할 점은 디자인 패턴을 강제로 적용하는 것이 Best가 아니란 점을 알아야 한다.

어떤 개념이건 항상 자신의 어플리케이션에 적용하기 위해서는 해당 패턴이 알맞은지 선행적인 검토가 필요하다.

## 생성(Creational) 패턴

소프트웨어 공학에서 생성 패턴은 **객체를 인스턴스화 할 때의 복잡성을 최대한 추상화**하기 위 한 것이다.

기본 객체 생성의 형태는 디자인을 복잡하게 만들거나 문제를 유발할 수 있다. 생성패턴은 문제를 어떻게든 제어함으로써 문제를 해결하고자 하는 것이다.
 
### 싱글턴

싱글턴은 클래스당 하나의 인스턴스를 사용하고 공유하도록 하는 객체 생성 패턴 중 하나로 파이썬 클린코드에서는 이를 권장하지 않는다.
~~(다른 언어 기반의 웹 프레임 워크의 경우 싱글턴이 기본적으로 컨테이너 클래스에 기능으로 탑재되어 있는 경우가 많다.)~~

```python
class MyClass:
    def get_instance():
        if self.obj is None:
            self.obj = MyClass()
        return self.obj
```

#### 단점
아래와 같은 이유로 객체를 싱긍턴 패턴에 의해 생성하는 것은 피해야 한다.
* 단위 테스트가 어렵다.
> 인스턴스가 하나만 생성된다는 것은 다른 단위테스트에 해당 인스턴스가 영향을 받을 수 있다는 것과 동의어이다.

#### 해결법
싱글턴을 사용하지 않고 해결하는 방법중 가장 쉬운 방법은 모듈을 사용하는 것이다. 

모듈에 객체를 생성할 수 있으며, 모듈을 `import`한 모든 곳에서 사용할 수 있다.

**파이썬에서 모듈은 이미 싱글턴이라는 것을 의미한다.**
> 즉 여러 번 `import` 하더라도 `sys.modules`에 로딩되는 것은 항상 한 개다.

### 모노 스테이트 패턴(SNGMONO)
모노 스테이트 패턴은 싱글턴인지 아닌지에 상관 없이 일반 객체처럼 많은 인스턴스를 만들어야 한다는 것이다.
> 모든 멤버변수를 static(python에서는 클래스 변수)으로 선언한 경우 해당 인스턴스는 모노 스테이트 패턴을 따랐다고 본다.

#### 장점
* 투명한 방법으로 정보를 동기화하기 때문에 사용자는 내부에서 어떻게 동작하는지 전혀 신경쓰지 않아도 된다.
* 에러가 발생할 가능성이 적다.

#### 모노 스테이트 패턴을 활용한 예 (Feat. Discriptor)
```python
class GitFetcher:
    _current_tag = None
    
    def __init__(self, tag) :
        self.current_tag = tag
    
    @property
    def current_tag(self) :
        if not self._current_tag:
            raise AttributeError( "tag가 초기화되지 않음" )
        return self._current_tag
    
    @current_tag.setter
    def current_tag(self, new_tag):
        self.__class__._current_tag = new_tag
    
    def pull(self):
        logger.info( "%s에서 풀", self.current_tag)
        return self.current_tag
   
   
   
>>> f1 = GitFetcher(0.1)
>>> f2 = GitFetcher(0.2)
>>> f1.current_tag = 0.3
>>> f2.pull()
0.3
>>> f1.pull()
0.3
```

디스크럽터를 생성하는 데코레이터를 활용하여, `f1.current_tag = 0.3` 에 닿는 순간 디스크럽터를 통해 인스턴스에 동기화되는 것을 알 수 있다.

이는 실제 디스크럽터를 구현하면 조금 많은 코드가 필요하지만, 구체적인 책임을 분리하여 각각이 응집력을 갖게 하므로 하나의 책임을 준수할 수 있게 된다. (SRP)

```python
class SharedAttribute:
    def __init__(self, initial_value=None) :
        self.value = initial_va lue
        self._name = None
    
    def __get__(self, instance, owner) :
        if instance is None:
            return self
        if self.value is None :
            raise AttributeError(f"{ self._name} was never set ")
        return self.value
    
    def __set__(self, instance, new_value):
        self.value = new_value
    
    def __set_name__(self, owner, name) :
        self._name = name


# 활용하기
class GitFetcher:
    current_tag = SharedAttribute()
    current_branch = SharedAttribute()
    
    def __init__(self, tag, branch=None):
        self.current_tag = tag
        self.current_branch = branch
    def pull(self):
        logger.info( "%s에서 플" , self.current_tag)
        return self.current_tag
```

이제 이 로직을 반복한다면 그저 `GitFetcher`클래스 처럼 인스턴스를 클래스 변수로 등록하기만 하면 된다. DRY(Don't Repeat Yourself)원칙을 자연스럽게 준수할 수 있다.


### Borg 패턴
이전의 솔루션은 대부분의 경우에서 잘 작동하나, 죽어도 싱글턴을 사용해야 하는 경우라면 최후의 대안이 있다.

바로 Borg패턴이다.

Borg 패턴은 앞서 설명했던 모노스테이트 패턴의 일종으로 같은 클래스의 모든 인스턴스가 모든 속성을 복제하는 객체를 만드는 것이다.
> _실제로 다른 블로그나, 레퍼런스에서는 모노스테이트 패턴의 다른 말로 Borg 패턴을 소개하기도 한다._

```python
class BaseFetcher:
    def __init__(self, source):
        self.source = source


class TagFetcher(BaseFetcher):
    _attributes = {}
    
    def __init__(self, source) :
        self.__dict__ = self.__class__._attributes
        super().__init__(source)
    
    def pull(self):
        logger.info("%s태그에서 풀", self.source)
        return f"Tag = {self.source}"


class BranchFetcher(BaseFetcher):
    _attributes = {}
    
    def __init__(self, source):
        self.__dict__ = self.__class__._attributes
        super().__init__(source)
    
    def pull(self):
        logger.info("%s 브랜치에서 풀", self.source)
        return f"Branch = {self.source}"
```

엄청난 것처럼 보이지만 실상은 클래스 변수를 두고 해당 클래스 변수를 멤버변수의 `__dict__` 속성에 계속 갱신해 줄 뿐이다.

#### 주의점
해당 패턴은 의도치 않게 다른 클래스의 객체에도 영향을 미칠 수 있다. 
> 때문에 많은 사람들은 이것을 패턴보다는 관용구에 가깝다고 생각한다.

이를 해결하기 위한 방법으로는 이를 믹스인(Minin) 클래스로써 활용하는 것이다.
믹스인 클래스의 인스턴스로 이를 활용하게 된다면, 클래스 별로 생성된 다른 객체이기 때문에, 상속 대상 클래스에 영향을 끼치지 않는다.

```python
class SharedAllMixin:
    def __init__(self, *args, **kwargs):
        try:
            self.__class__._attributes
        except AttributeError: 
            self.__class__._attributes = {}

        self.__dict__ = self.__class__._attributes 
        super().__init__(*args, **kwargs)


class BaseFetcher:
    def __init__(self, source):
        self.source = source


class TagFetcher(SharedAllMixin, BaseFetcher):
    def pull(self):
        logger.info("%s 태그에서 풀", self.source) 
        return f"Tag = {self.source}"


class BranchFetcher(SharedAllMixin, BaseFetcher): 
    def pull(self):
        logger.info("%s 브랜치에서 폴", self.source) 
        return f"Branch = {self.source}"
```

### 빌더(Builder) 패턴
빌더 패턴은 객체의 복잡한 초기화를 추상화하는 흥미로운 패턴이다.

해당 패턴의 핵심은 객체를 생성하는 부분과, 객체를 사용하는 부분을 분리하는 것이다.

사용자가 필요로하는 모든 보조객체를 직접 생성하여 메인 객체에 전달하는 것이 아니라, 한번에 모든 것을 처리해주도록 추상화를 해야한다

빌더 객체는 클래스 메서드와 같은 사용자 인터페이스를 제공하며, 사용자는 최종 객체에 대한 모든 정보를 해당 인터페이스에 파라미터로 전달하면 된다.

> _해당 예시는 파이썬으로 구현할 수 있으나, 적절하지 않다. 따라서 Java 코드로 대체한다._

아래와 같이 생성을 해야하는 부분이 굉장히 복잡한 `Person` 클래스가 있다

```java
class Person {    
    private String name;    
    private String age;    
    private String sex;    
    private String phoneNumber;    
    private String address;    
    private String job;        
    
    public Person(String name) {        
    this(name, "");    
    }        
    
    public Person(String name, String age) {        
    this(name, age, "");    
    }
    
    public Person(String name, String age, String sex) {        
    this(name, age, sex, "");    
    }        
    
    public Person(String name, String age, String sex, String phoneNumber) {
    this(name, age, sex, phoneNumber, "");    
    }        
    
    public Person(String name, String age, String sex, String phoneNumber, String address) {        
    this(name, age, sex, phoneNumber, address, "");    
    }        
    
    public Person(String name, String age, String sex, String phoneNumber, String address, String job) {        
    this.name = name;        
    this.age = age;        
    this.sex = sex;        
    this.phoneNumber = phoneNumber;        
    this.address = address;        
    this.job = job;    
    }
}
```

생성자 오버로딩을 통해 많은 경우를 포함하고 있으나, 새로운 생성자가 필요할 경우 개발자는 새로운 생성자를 일일히 추가해야만 할 것이다.

이제 Bulider 패턴을 적용해보자
```java
// Person.java
class Person{   
    private String name;
    private String age;    
    private String sex;    
    private String phoneNumber;    
    private String address;    
    private String job;      
    
    public Person(String name, String age, String sex, String phoneNumber, String address, String job) {        
        this.name = name;        
        this.age = age;        
        this.sex = sex;        
        this.phoneNumber = phoneNumber;        
        this.address = address;        
        this.job = job;    
    }     
 
    @Override    
    public String toString() {        
    return "Person [name=" + name + ", age=" + age + ", sex=" + sex + ", phoneNumber=" + phoneNumber + ", address=" + address + ", job=" + job + "]";         }
}

// PersonBuilder.java
class PersonBuilder{
    private String name;    
    private String age;    
    private String sex;    
    private String phoneNumber;    
    private String address;    
    private String job;            
    
    public PersonBuilder setName(String name) {        
        this.name = name;                
        return this;      
    }        
    
    public PersonBuilder setAge(String age){        
        this.age = age;                
        return this;   
    }        
    
    public PersonBuilder setSex(String sex){        
        this.sex = sex;                
        return this;   
    }        
    
    public PersonBuilder setPhoneNumber(String phoneNumber){        
        this.phoneNumber = phoneNumber;                
        return this;    
    }        
    
    public PersonBuilder setAddress(String address){        
        this.address = address;                
        return this;    
    }        
    
    public PersonBuilder setJob(String job){
        this.job = job;                
        return this;    
    }        
    
    public Person build() {        
        Person person = new Person(name, age, sex, phoneNumber, address, job);                
        return person;    
    }   
}
```

자바 코드가 긴 탓도 있지만, 사실 엄청 간단해졌다. 위의 경우 멤버변수가 초기화 될 경우 모든 경우에 대비해야 한다면 생성자는 멤버 변수의 수 n의 팩토리얼 개수만큼 필요할 것이다.

심지어 파라미터의 타입이 같은 변수들의 경우, 변수 타입이 중복될 때 적절한 생성자를 만들어낼 수도 없다.

그러나 해당 패턴은 `build()` 함수를 통해 `Person` 객체를 만들기 때문에, **자유롭게 멤버변수를 초기화 하고, 객체를 만들 수 있다.**

파이썬에서는 keyword argument라는 강력한 도구가 있기 때문에 이런 패턴이 강력하게 필요하진 않으나, 복잡한 생성 코드가 필요한 경우 유용하게 이와 유사하게 사용할 수 있을 것이다.

#### Reference
https://ktko.tistory.com/entry/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4Build-Pattern

#### 용도
여러 사용자가 사용하는 인터페이스 같은 것을 노출해야 하는 경우 구현한다. 

1. 프레임워크
2. 라이브러리
3. API 디자인



## 구조(Structural) 패턴

구조 패턴은 인터페이스를 복잡하게 하지 않으면서도 기능을 확장하여 더 강력한 인터페이스 또는 객체를 만들어야 하는 상황에 유용하다.

이러한 패턴의 가장 큰 장점은 **향상된 기능을 깔끔하게 구현할 수 있다는 것**이다.

### 어댑터(Adapter) 패턴(= Wrapper 패턴)
> _"어댑터 패턴은 아마도 가장 단순하면서도 유용한 디자인 패턴일 것이다." p.285_ 

이 패턴은 호환되지 않는 두 개 이상의 객체에 대한 인터페이스를 동시에 사용할 수 있게 한다.

개발자는 다형성을 이용해 여러 클래스나 모델을 사용하게 된다. 

예를 들어 데이터를 가져오는 `fetch()` 메서드가 있고 해당 메서드를 유지하면 코드를 크게 바꿀 필요가 없다고 생각해보자.
그러나 새로운 데이터 소스를 추가해야 한다면, 기존 객체는 호환되지 않을 수도 있고, Open API에서 가져오는 메서드라면 수정 권한이 없는 경우도 있다.

이때 어댑터 패턴은 이에 답이 될 수 있다.

어댑터 패턴을 구현하는 방식은 크게 2가지 방식이 있다.

#### 1. 사용하려는 클래스를 상속받는 클래스를 만들기

```python
from _adapter_base import UsernameLookup

class UserSource(UsernameLookup):
    def fetch(self, user_id, username):
        user_namespace = self._adapt_arguments(user_id, username)
        return self.search(user_namespace)
    
    @staticmethod
    def _adapt_arguments(user_id, username):
        return f"{user_id}:{username}"
```

`UsernameLookup`을 상속받아서 `fetch()`메서드를 사용할 수 있게 되었다. 추가적으로 `user_id`, `user_name` 변수도 적절히 사용할 수 있게 되었다.

그런데 위와 같은 방식에는 문제가 있다. 

앞장에서 살펴보았듯 상속은 **is a** 관계가 성립해야 하는 것이 개념적으로 가장 옳다고 하였다.

과연 `UserResource`는 `Usernamelookup`인가?

#### 2. 사용하려는 클래스를 포함(Composition)시키기

```python
class UserResource
    ...
    def fetch(self, user_id, user_name):
        user_namespace = self._adapt_arguments(user_id, username) 
        return self.username_lookup.search(user_namespace)
```

`UserResource`는 `UsernameLookup`을 가지고 있는가? 맞다.
개념적으로나, 구현으로나 훨씬 간편하게 적용할 수 있다.

### 컴포지트 패턴

클래스는 객체를 정의해 놓은 기본 클래스가 있을 수 있고 이를 묶어서 사용하는 컨테이너 클래스가 있을 수 있다.
컴포지트 패턴은 기본 클래스와 컨테이너 클래스에 동일한 연산을 수행하고 싶을 때 사용할 수 있다.

아래는 상품정보를 담은 `Product` 클래스와 상품을 묶은 `ProductBundle` 클래스의 예가 있다.

```python
class Product:
    def __init__(self, name, price):
        self._name = name 
        self._price = price
    
    @property
    def price(self):
        return self._price

class ProductBundle: 
    def __init__(self, name, perc_discount, *products: Iterable[Union[Product, "ProductBundle"]]) -> None:
        self._name = name 
        self._perc_discount = perc_discount 
        self._products = products
        
    @property
    def price(self):
        total = sum(p.price for p in self._products) 
        return total * (1 - self ._perc_discount)

```

`price()` 메서드를 프로퍼티로 설정하고 컨테이너, 기본 클래스에 동일하게 연산을 처리하도록 하였다. 

해당 연산은 `ProductBundle` 클래스에서 각 `self._products`의 개별요소에 접근하여 `price()` 메서드를 재귀적으로 호출하여 가격을 구하고 있기 때문에, 동일한 연산이라도 오류가 생기지 않는다.

### 데코레이터 패턴

먼저 말할 것이 있다. 파이썬 데코레이터와는 전혀 다르다.

이 패턴을 사용하면 상속을 하지 않고도 객체의 기능을 동적으로 확장할 수 있다. 보다 유연한 객체를 만들려고 할 때 다중 상속의 좋은 대안이 될 수 있다.

파이썬은 덕타이핑을 지원하기 때문에 새로운 기본 클래스를 만들어서 클래스 계층 구조에 새로 편입시킬 필요가 없다. 따라서 같은 이름의 메서드를 만들어서 기능을 확장해 나갈 수 있다.


이렇게 **동일한 인터페이스를 가지고 결과를 향상시키거나, 결합할 수 있도록 패턴을 만드는 것을 데코레이터 패턴**이라고 하며, **새롭게 만들어진 기능을 추가하는 단계를 데코레이션**이라고 한다.

```python
class DictQuery:
    def __init__(self, **kwargs):
        self._raw_query = kwargs

    def render(self) -> dict: 
        return self._raw_query


class QueryEnhancer:
    def __init__(self, query: DictQuery):
        self.decorated = query

    def render(self):
        return self.decorated.render()
    
    
class RemoveEmpty(QueryEnhancer) : 
    def render(self):
        original = super().render()
        return {k : v for k, v in original.items() i f v}


class CaseInsensitive(QueryEnhancer):
    def render(self):
        original = super().render()
        return {k: v.lower() for k, v in original.iterns()}



>>> original = DictQuery(key="value", empty="", none=None, upper="UPPERCASE", title="Title")
>>> new_query = CaseInsensitive(RemoveEmpty(original))
>>> original.render()
{'key': 'value', 'empty': '', 'none': None, 'upper': 'UPPERCASE', 'title': 'Title'} 
>>> new_query.render()
{'key': 'value', 'upper': 'uppercase', 'title': 'title'}
```
기본 클래스인 `DictQuery`를 토대로 동일한 인터페이스를 가지는 클래스 `RemoveEmpty`, `CaseInsensitive`, `QueryEnhencer`를 구현하여 기능을 추가하고 있다.

호환성을 위해 각 클래스는 `render()`메서드의 형태를 그대로 유지했기 때문에 클라이언트는 코드를 수정하지 않아도 된다.

### 파사드(Facade) 패턴
> facade는 건물의 가장 중요한 면을 가리키는 단어로 보통은 정면을 말한다. 소프트웨어 공학에서는 복잡한 시스템을 가려주는 단일 통합 창구 역할을 하는 객체를 말한다.

![image](https://user-images.githubusercontent.com/59782504/174620230-3925f072-2c80-49d9-8317-4a5b1a0b6125.png)

파사드는 객체 간 상호 작용을 단순화하려는 많은 상황에서 유용하다. 객체에 대한 모든 연결을 만드는 대신 파사드 역할을 하는 중간 객체를 만드는 것이 파사드 패턴의 핵심이다.

파사드는 허브 또는 단일 참조점(single point of reference)의 역할을 한다.

새로운 객체가 다른 객체에 연결하려고 할 때마다 연결해야 하는 N개의 객체에 대해 N개의 인터페이스를 만들어야 한다면 너무 복잡할 것이다.

이럴 때 단지 파사드와 대화하게 하고 파사드에서 적절히 요청을 전달해주면 편리할 것이다. 
외부 오브젝트의 입장에서는 파사드 내부의 모든 내용이 완전히 불투명해야한다.

#### 장점
1. 결합력을 낮추어 준다.
2. 인터페이스의 개수를 줄일 수 있게 한다.
3. 보다 나은 캡슐화를 지원할 수 있게 하므로 간단한 디자인을 유도한다.

#### 용도
* 도메인 문제 개선
* 단일 인터페이스 제공 시, 단일 진리점 또는 코드의 진입점 역할을 하여 클라이언트가 노출된 기능을 쉽게 사용할 수 있게 함.
* 기능만 노출하고 모든 것은 인터페이스에 숨길 수 있게 함.
> 세부 코드는 원하는 만큼 리펙토링이 가능
* 클래스나 객체에만 한정된 것이 아닌 패키지에도 적용 가능함. _(`__init__.py`가 단일 진입점 역할을 한다.)_

## 행위(Behavioral) 패턴
행동 패턴은 **객체가 어떻게 협력해야하는지, 어떻게 통신해야하는지, 런타임 중에 인터페이스는 어떤 형태여야 하는지에 대한 문제를 해결하는 것**을 목표로 한다.

### 책임 연쇄 패턴
해당 패턴은 특정 클래스나 객체가 처리가 불가능할 경우 후계자에게 전달하고 이런 과정을 반복하는 패턴을 뜻한다.
말 그대로 책임이 전가 되는 것이라고 생각하면 쉽다.

```python
import re


class Event: 
    pattern = None
    
    def __init__(self, next_event=None): 
        self.successor = next_event

    def process(self, logline: str): 
        if self.can_process(logline):
            return self._process(logline)
        if self.successor is not None:
            return self.successor.process(logline)
            
    def _process(self, logline: str) -> dict: 
        parsed_data = self._parse_data(logline) 
        return { 
            "type": self.__class__.__name__, 
            "id" : parsed_data["id"],
            "value" : parsed_data["value"],
        }
        
    @classmethod
    def can_process(cls, logline: str) -> bool:
        return els.pattern.match(logline) is not None
    
    @classmethod
    def _parse_data(cls, logline: str) -> dict:
        return els.pattern.match(logline).groupdict()


class LoginEvent(Event):
    pattern = re.compile(r"(?P<id>\d+):\s+login\s+(?P<value>\S+)")


class LogoutEvent(Event):
    pattern = re.compile(r"(?P<id>\d+):\s+logout\s+(?P<value>\S+)")
```

이전에 살펴보았던 Event 클래스와 동일하나 후계자(`successor`)개념이 추가되었다.
후계자는 현재 이벤트가 로그라인을 처리할 수 없는 경우에 대비한 추가 이벤트 객체이다

이후 event 객체를 만들고 처리해야 할 특정 순서로 정렬하면 된다.
이벤트 객체는 모두 `process()` 메서드를 가지고 있으므로, 메시지에 대한 다형성을 가지고 있으며, 정렬 순서는 클라이언트가 마음대로 바꿀 수 있다.

```python
>>> chain = LogoutEvent(LoginEvent())
>>> chain.process("567: login User")
{'type': ·'LoginEvent 'id': '567', 'value': 'User'}

```

이 솔루션은 충분히 유연하며 모든 조건들이 상호 배타적이 라는 특성을 그대로 유지하고 있다. 
판별 로직의 충돌이 없고 동일한 데이터를 하나 이상의 핸들러가 처리하지 않는다면 어떤 순서로 이벤트를 처리하는지는 상관이 없다.

그런데 사용자가 런타임 중에 우선순위를 변경하고 싶으면 어떻게 해야 할까? 새로운 솔루션을 사용하면 이러한 추가 요구사항도 만족시킬 수 있다. 
왜냐하면 런타임 중에 책임을 연결(`chain`)하면 되기 때문이다.

```python
class SessionEvent(Event):
    pattern = re.compile(r"(?P<id>+):+log(inIout)+(?P<value>+)")

>>> chain = SessionEvent(LoginEvent(LogoutEvent()))
```

### 템플릿 메서드 패턴
해당 패턴을 적절히 구현하면 코드의 재사용성을 높여주고 객체를 보다 유연하게 하여 다형성을 유지하면서도 코드를 쉽게 수정할 수 있다

어떤 행위를 정의할 때 특정한 형태의 클래스 계층구조를 만드는 것이다.
공통적인 로직을 부모 클래스의 `public` 메서드로 구현하고 그 안에서 내부의 다른 `private` 메서드들을 호출하는 것이다.
템플릿에서 호출하는 다른 내부의 메서드들은 파생 클래스에서 수정될 수 있지만 템플릿에 있는 기본 공통 로직은 모두 재사용된다.

```python
import re


class Event: 
    pattern = None
    
    def __init__(self, next_event=None): 
        self.successor = next_event

    def process(self, logline: str): 
        if self.can_process(logline):
            return self._process(logline)
        if self.successor is not None:
            return self.successor.process(logline)
            
    def _process(self, logline: str) -> dict: 
        parsed_data = self._parse_data(logline) 
        return { 
            "type": self.__class__.__name__, 
            "id" : parsed_data["id"],
            "value" : parsed_data["value"],
        }
        
    @classmethod
    def can_process(cls, logline: str) -> bool:
        return els.pattern.match(logline) is not None
    
    @classmethod
    def _parse_data(cls, logline: str) -> dict:
        return els.pattern.match(logline).groupdict()


class LoginEvent(Event):
    pattern = re.compile(r"(?P<id>\d+):\s+login\s+(?P<value>\S+)")


class LogoutEvent(Event):
    pattern = re.compile(r"(?P<id>\d+):\s+logout\s+(?P<value>\S+)")
```

위에서 다루었던 코드다.

알고 보면 `Event`클래스의 파생 클래스는 패턴만 정의할 뿐이고 실제 공통로직은 부모클래스인 `Event`클래스에서 `process()`함수를 통해 이루어진다. 
메서드 내부에서는 `can_process()`와 `_process()`라는 두 개의 보조 메서드가 호출된다.

`private` 메서드들은 클래스 속성에 의존한다. 
따라서 새로운 타입에서 기능을 확장하려면 단지 자식 클래스에서 정규식으로 `pattern` 속성을 재정의하기만 하면 된다.

이 패턴은 자신만의 라이브러리나 프레임워크를 만들 때에도 유용하다. 이런 방법으로 로직을 정리하면 사용자에게 클래스의 일부 행동을 쉽게 수정하도록 할 수 있다.

### 커맨드 패턴
커맨드 패턴(command pattern)은 수행해야 할 작업을 요청한 순간부터 실제 실행 시까지 분리할 수 였는 기능을 제공한다. 또한 클라이언트가 발행한 원래 요청을 수신자와 분리할 수도 있다. 

특정 작업을 추상화할 때 많이 사용하는 패턴으로, 예를 들어 연주한다(`play()`)라는 메서드가 있다면, 바이올린(`Violin`)을 연주할 수도 있고, 피아노(`Piano`)를 연주하게 될 수도 있다.

근데 기타(`Guiter`)클래스는 조율한다(`tune()`)라는 메서드가 추가적으로 있다고 가정하자

그리고 연주와 조율을 하는 손(`Hand`)라는 클래스가 있다고 하자. 일반적인 방법으로는 다음과 같이 구현해야 할 것이다.

```python

class Piano:
    def play(self):
        print("피아노 빰바밤")
        
class Guiter:
    def tune(self):
        print("조율을 합니다 끼익끼익")

class Violin:
    def play(self):
        print("바이올린 따라리라~")

class Hand:
    def __init__(self):
        self.piano = Piano()
        self.guiter = Guiter()
        self.violin = Violin()
    

>>> hand = Hand()
>>> hand.piano.play()
"피아노 빰바밤"
>>> hand.guiter.tune()
"조율을 합니다 끼익끼익"
>>> hand.violin.play()
"바이올린 따라리라~"
```

만약 새로운 악기가 추가되면 어떨까? 아니, 만약 손이 할 수 있는 범위내로 상정한다면 요리가 추가될 수도 있다.
요리는 중국요리와 일본 요리별로 구현해야 할 지 모른다.

이 때 커맨드 패턴을 적용하면 표준적이며, 추상화된 클래스를 만들 수 있다.
세 악기 클래스는 `InstrumentCommand`라는 클래스를 상속받고 `InstrumentCommand` 클래스에는 실행시키는 `run()`메서드가 있다고 가정하자. 그리고 각 함수를 다음과 같이 구현한다.

```python
class InstrumentCommand:
    def run(self):
        pass
        
class Piano(Command):
    def play(self):
        print("피아노 빰바밤")
    
    #Override
    def run(self):
        self.play()
        
class Guiter(Command):
    def tune(self):
        print("조율을 합니다 끼익끼익")
        
    #Override
    def run(self):
        self.tune()

class Violin(Command):
    def play(self):
        print("바이올린 따라리라~")
      
    #Override  
    def run(self):
        self.play()

class Hand:
    def __init__(self, command):
        self.command = command
    
    def set_command(command:Command):
        self.command = command
  
>>> my_hand = Hand(Piano())
>>> my_hand.command.run()
"피아노 빰바밤"
>>> my_hand.set_command(Violin())
>>> my_hand.command.run()
"바이올린 따라리라~"
>>> my_hand.set_command(Guiter())
>>> my_hand.command.run()
"조율을 합니다 끼익끼익"
```

이처럼 `my_hand` 인스턴스는 그저 `run()`메서드만 실행하고, `InstrumentCommand` 클래스를 상속받은 자식클래스의 인스턴스를 `set_command()`에 주입하는 것으로 설사 메서드 명이 다를 지라도 동일한 인터페이스로 다양한 구현을 할 수 있는 것을 알 수 있었다.

이때 `run()`메서드를 실행시키는 `Hand`클래스는 호출자(Invoker)클래스, `InstrumentCommand`클래스는 명령(Command)클래스, `InstrumentCommand`클래스를 상속받아 실제 명령을 구현하는 클래스를 리시버 클래스라고 하며, 해당 구성으로 각 객체의 동작을 캡슐화 시키는 방식을 커맨드 패턴이라고 정의한다.

> 물론 파이썬의 경우 덕 타이핑을 지원하기 때문에 함수명만 맞추어 주어도 일관적인 동작을 할 수 있으나, 해당 방식은 인터페이스가 정해져 있을 경우 특정 작업을 일관적인 메서드를 통해 하고 싶을 때 사용하면 유용하다.

### 상태(State) 패턴

상태 패턴은 구체화를 도와주는 대표적인 소프트웨어 디자인 패턴이다.

일반적으로 상태는 상수형태로 선언하여 관리하는 경우가 많다. 이 경우, 문자열이거나, 열거형을 주로 사용한다.

그런데 만약 어떤 상태에 따라 특정 행동을 해야할 경우, 상수로 분기를 하기에는 한 클래스에서 너무 많은 책임을 가지게 될 수 있다.

예를 들어 MergeRequest클래스가 있고, 상태(open, close 등)에 따라 특정 동작을 해야 한다면 이를 열거형이나 상수형으로 구현한다면 다음과 같은 문제가 생길 수 있다.

1. 상태를 판별하는 함수에 과중한 책임이 실린다.
> 상태에 따라 특정 메서드를 실행하도록 구현하여야 한다.
2. 과도한 if, elif 문으로 가독성을 해칠 수 있다.
3. 특정 기능이 추가될 때마다 유지보수 비용이 올라가며, 조건이 변경되면 메서드 로직 전체를 수정해야 할 경우가 생길 수 있다.

이에 각 상태를 추상화 시켜 객체로 만들고, 이에 따른 동작을 메서드에 정의한다면 조금 더 좋은 코드를 작성할 수 있을 것이다.

#### 구현 예
```python
class InvalidTransitionError(Exception):
    """도달 불가능한 상대에서 전이할 때 발생하는 예외"""


class MergeRequestState(abc.ABC):
    def __init__(self, merge_request):
        self._merge_request
    
    @abc.abstractmethod
    def open(self):
        ...
        
    @abc.abstractmethod 
    def close(self):
        ...
        
    @abc.abstractmethod 
    def merge(self):
    ...
    
    def __str__(self):
        return self.__class__.__name__
        
        
class Open(MergeRequestState): 
    def open(self):
        self._merge_request.approvals = 0
        
    def close (self): 
        self._merge_request.approvals = 0
        self._merge_request.state = Closed
        
    def merge(self):
        logger.info("%s 머지", self._merge_request)
        logger.info("%s 브랜치 삭제", self._merge_request.source_branch) 
        self ._merge_request.state = Merged


class Closed(MergeRequestState): 
    def open(self):
        logger.info("종료된 머지 리퀘스트 %s 재오픈", self._merge_request) 
        self._merge_request .state = Open

    def close(self): 
        pass

    def merge(self):
        raise InvalidTransitionError("총료된 요청을 머지할 수 없음")


class Merged(MergeRequestState): 
    def open(self):
        raise InvalidTransitionError(“이미 머지 완료 됨")

    def close(self):
        raise InvalidTransitionError(“이미 

    def merge(self):
        pass


class MergeRequest:
    def __init__(self, source_branch: str, target_branch: str) -> None:
        self.source_branch = source_branch 
        self.target_branch = target_branch 
        self._state = None
        self.approvals = 0
        self.state = Open

    @property
    def state(self):
        return self._state

    @state.setter
    def state(self, new_state_cls):
        self._state = new_state_cls(self)

    def open(self):
        return self.state.open()
        
    def close(self):
        return self.state.close()
    
    def merge(self) :
        return self.state.merge()
        
    def __str__(self):
        return f"{self.target_branch}:{self.source_branch}"
```

`MergeRequest` 클래스는 `state` 프로퍼티를 사용하여 상태를 변경시킬 수 있으며, 각 상태 클래스(`MergeRequestState`클래스를 상속받은 자식클래스)는 내부적으로 메서드(`open`, `close`, `merge`) 메서드를 통해 각 상태에 따른 동작이나 조건을 명시할 수 있다.

`MergeRequest`가 모든 처리를 `state`객체에 위임했기 때문에 항상 `self.state.method()`와 같은 형태로 호출하게 된다.

이에 멈추지 않고, `open()`, `close()`, `merge()`와 같은 반복 정의된 함수의 형태를 매직 메서드(`__getattr__()`)을 통해 다음과 같이 간략하게 표현할 수도 있을 것이다.

```python
def __getattr__(self, method):
    return getattr(self.state, method)
```

이처럼 상태를 그저 명세에 머물지 않고 동작의 형태로 구현하여 책임을 분산시키는 형태를 상태(State)패턴이라고 정의한다.

## Null 객체 패턴
메서드의 반환값은 항상 일관성이 있어야 한다는 패턴

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

class UserList:
    def __init__(self)
        self.users = []
    
    def find_user(name) -> User or None: # (X): 반환값이 일관적이지 않음.
        ...
         
    def find_user(name) -> User: # (0): 반환값이 일관적임
        ...
        
        ...
        # 찾는 유저가 없는 경우
        return User("홍길동", -1)
```

### 장점
1. 런타임 시 오류를 피할 수 있음
2. 객체를 유용하게 활용할 수 있음.
3. 코드를 테스트하기 쉬워짐
4. 디버깅에 도움이 될 수 있음

## 디자인 패턴, 흑과 백
> _"어떤 사람들은 디자인 패턴은 타입이 제한된 시스템(또는 퍼스트 클래스 함수(first― class function)을 지원하지 못하는 시스템)이 파이썬에서 일반적으로 하는 것들을 지원해주기 위한 것이므로 굳이 파이썬에 적용하는 것은 오히려 장점보다 단점이 많다고 말한다." - p.305_

> _"또 어떤 사람들은 디자인 패턴이 디자인 솔루션을 강요하면서 더 좋은 디자인이 나올 수 였는 기회를 제한하는 단점이 있다고 말한다." - p.305_

> _"일반적인 솔루션을 만들거나 추상화 를 하기 전에 3회 반복의 법칙을 기억할 필요가 있다." - p.306_

### 모델의 명명

디자인 패턴의 이름을 클래스에 명명하면 명확한 의도를 드러내지 못하기 때문에 이를 이름에 적용하는 것은 좋지 못하다.
> 예를 들어 클래스가 쿼리를 나타낸다면 `Query`, `XXXQuery` 등의 이름으로 나타내야 하지 `XXXQueryDecorator`와 같은 이름은 전혀 의미있는 이름이 아니며, 접미사로 인해 오히려 혼란만 가중될 것이다.



