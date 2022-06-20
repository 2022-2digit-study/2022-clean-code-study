# [Chapter. 9] 일반적인 디자인 패턴

디자인 패턴은 갱 오브 포(Gang of Four - GoF : Design Patterns : Reusable Object— Oriented Software)로 유명한 책의 소개와 함께 소프트웨어 공학에 널리 보급되어왔다.

디자인 패턴 자체가 무조건 이를 따라야 하는 교본 같은 아니다. 그러나 자주 하는 디자인 패턴에는 이유가 있는 법이다. 

또한 이는 특정 프로그래밍 언어에 종속된 것이 아니다. 오히려 객체를 사용하는 모든 어플리케이션에서 상호작용하는 방법에 관한 보다 일반적인 개념에 가까울 것이다.

## 3가지 패턴

디자인의 패턴은 크게 생성(Creational), 구조(Structural), 행동(Behavioral) 패턴 중 하나로 분류된다. 또한 이것의 확장 버전이나 변형버전의 패턴 역시 존재한다.

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

이제 이 로직을 반복한다면 그저 GitFetcher클래스 처럼 인스턴스를 클래스 변수로 등록하기만 하면 된다. DRY(Don't Repeat Yourself)원칙을 자연스럽게 준수할 수 있다.


### Borg 패턴
이전의 솔루션은 대부분의 경우에서 잘 작동하나, 죽어도 싱글턴을 사용해야 하는 경우라면 최후의 대안이 있다.

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



