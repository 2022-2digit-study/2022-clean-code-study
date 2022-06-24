# [Chapter 10. 클린 아키텍처]



## 클린 코드에서 클린 아키텍처로

지금까지 살펴본 내용을 시스템의 관점에서 다시 보자.  

### 아키텍처에서의 관심사의 분리

컴포넌트는 한 가지 작업만을 수행해하도록 관심사의 분리가 되어야했다.  

여태까지 살펴본 어플리케이션 내부의 컴포넌트는 

1. 가능한 작게 유지해야하며
2. 하나의 잘 정의된 책임을 가져야하고
3.  수정이 쉬워 확장성이 좋아야하며
4. 높은 응집력과 낮은 결합력을 가져야한다

고 했었다.   

이처럼 코드에서 말하는 컴포넌트는 응집력이 높은 유닛(클래스 등)이 될 수 있었다.  

그렇다면 아키텍처에서 말하는 컴포넌트는 어떤 것일까?  

아키텍처에서 말하는 컴포넌트는 시스템에서 **작업 작업 단위로 취급할 수 있는 모든 것**을 의미한다.  

대형 시스템을 monolithic하게 구현하는 것은 single source of truth처럼 동작하여 시스템의 모든 것을 책임지게 되고 의도하지 않은 결과를 유발할 수 있다.  

코드도 모든 것을 한 곳에 두면 관리가 어려운 것처럼 컴포넌트도 그렇다.  

컴포넌트는 시스템의 일부분이면서 어플리케이션 자체가 되어서는 안된다.  

또한 컴포넌트는 자신만의 주기를 가지고 배포되어야하며, 나머지 시스템 컴포넌트와 독립적이어야한다.  

사실 컴포넌트라는 말은 매우 모호하지만, 파이썬 프로젝트의 경우 패키지가 될 수 있다.(서비스까지도 될 수 있음)  



### 아키텍처에서의 추상화

추상화란 세부 구현사항을 최대한 숨기는 것이었다.  

여태까지 살펴본 코드레벨의 추상화는 코드 그 자체로 문서화가 되는 표현력을 가지되 문제 본질에 대한 해결책을 제시하도록 해야했다.  

코드 레벨에서처럼 시스템 도메인 문제에서도 추상화란 세부 구현사항을 최대한 숨기는 것이며, 

그 자체로 시스템이 어떻게 되는지 설명할 수 있어야 한다. (Screaming Architecture)

[^Screaming Architecture]: 소프트웨어의 구조가 소리치듯이 구조만 봐도 어떤 서비스 인지 알 수 있어야 한다는 의미를 가지고 있다. 이 구조는 소프트웨어가 무슨 기능을 하는지에 초점을 맞춰 구조를 설계한다.

특히,  디스크 저장 방법이나 라이브러리같은 세부사항이 아닌 시스템이 무엇을 하는가가 중요하다.  

4장의 SOLID원칙에서 의존성 역전 원칙(DIP)을 적용하는 것도 도움이 되지만, 이것으로는 충분하지 않다.  

객체를 추상화하는 것 이상의 추상화 계층이 필요하며 이는 세부적인 코드레벨 추상화와의 차이점이다.  

아키텍처의 관점에서는 파이썬의 abc모듈이나 덕타이핑을 지원하지 않기 때문에, 의존성 전체를 추상화해야한다.  

따라서  컴포넌트를 애플리케이션에서 임포트하여 사용하되 애플리케이션의 로직을 몰라야하며,  

데이터베이스도 애플리케이션 자체에 대해 아무것도 몰라야한다. (Hexagonal Architecture)

[^Hexagonal Architecture]: 어플리케이션의 코어가 각 어댑터와 상호작용하기 위해 특정 포트를 제공하는 아키텍처

![헥사고날아키텍처](https://user-images.githubusercontent.com/55227276/175467200-99180df3-34a0-4c7c-82a8-e9366e094051.png)



## 소프트웨어 컴포넌트

대규모 시스템을 확장할 때는 조직적인 문제가 발생한다.  

각 소프트웨어 저장소는 애플리케이션에 속해있으며, 각 애플리케이션은 시스템의 일부를 소유하고 있는 팀에서 관리를 하고 있기 때문이다.  

따라서 대규모 시스템이 잘 작동하기 위해서는 모든 부분이 인터페이스화 되는 것이 좋다. 

이런 방법 두 가지를 살펴보자.   



### 패키지

파이썬의 패키지는 소프트웨어를 배포하고 일반적인 방식으로 코드를 재사용하기 위한 편리한 방법이다.  

빌드한 패키지를 아티팩트 저장소에 업로드하고 이를 다운로드하는 방식을 사용한다.  

이렇게 함으로써 코드의 재사용성을 높이고 개념적인 무결성을 얻을 수 있다. 

파이썬 패키징의 핵심은 다음과 같다.  

1.  플랫폼에 독립적이며 로컬 설치에 의존하지 않는지 테스트하고 검증을 해야한다.
2. 단위 테스트를 패키지에 같이 배포하지 않는다.
3. 의존성을 분리한다.
4. 가장 많이 요구되는 명령에 대해서 진입점을 만드는 것이 좋다.

일반적인 패키지 구조를 살펴보자. 

```
├── Makefile
├── README.rst
├── setup.py
├── src
|   ├── apptool
|   ├── common.py
|   ├── __init__.py
|   └── parse.py
└── tests
    ├── integration
    └── unit
```

> setup.py: 프로젝트의 모든 중요 정의(요구사항, 의존성, 이름, 설명 등)가 들어있는 파일(중요)
>
> src/apptool: 작업하고 있는 라이브러리의 이름, 일반적으로 필요한 것들은 모두 여기에 배치

setup.py 파일의 예를 살펴보자.

```python
from setuptools import find_package, setup

with open("README.rst", "r") as longdesc:
    long_description = longdesc.read()
    
setup(
    name = "apptool",
    description="패키지 설명", 
    long_description=long_discription,
    author="저자이름",
    version="0.1.0",
    packages=find_packages(where="src/"),
    package_dir={"": "src"}
)
```

> setup함수의 name인자는 저장소에서 사용할 패키지 이름을 지정하며, 이 이름을 사용해 설치할 수 있음
>
> setup함수의 version인자는 배포에 중요한 정보가 되며 패키지를 지정하는데 사용됨
>
> find_package함수는 자동으로 모든 패키지를 검색하며, src/ 디렉토리에서 검색을 함

패키지는 다음 명령어를 통해 빌드할 수 있다.  

```bash
$ VIRTUAL_ENV/bin/pip install -U setuptools wheel
$ VIRTUAL_ENV/bin/python setup.py sdist bdist_wheel
```

패키지에 여러 운영체제 라이브러리가 설치되어야하는 경우 setup.py에 논리를 작성하여 빌드하는 것이 좋은데,  

이렇게 할 경우 무언가 잘못되었을 때 설치 프로세스 초기에 오류를 발생시켜 사용자가 확인하게 할 수 있으며 의존성을 보다 빨리 확인하여 진행할 수 있다.  

의존성에 따른 장애물은 Docker이미지를 만들어 플랫폼을 추상화하는 것으로 해결할 수 있으며, 이는 다음 섹션에서 알아보자. 



### 컨테이너

이 장에서 말하는 컨테이너는 파이썬 컨테이너와 다른 것이며 특정한 제약사항을 가지고 격리된 상태로 실행되는 프로세스를 의미한다.  

컨테이너는 패키지와 또 다른 형태의 소프트웨어 전달 수단으로 본 책에서는 구체적으로는 도커의 컨테이너를 말한다. 

_(물론 알다시피 도커가 애플리케이션을 컨테이너화하는 유일한 기술은 아니다.)_

컨테이너는 애플리케이션을 생성할 때 사용하며 애플리케이션을 독립적인 컴포넌트로 관리할 수 있다.  

컨테이너를 만드는 이유는 작은 컴포넌트를 만들기 위함이다.  

도커 컨테이너는 실행할 이미지가 필요하며, 우리가 만드는 이미지는 다른 컨테이너의 기본 이미지로 사용될 수 있다.  

이는 다른 많은 컨테이너에서 공유할 수 있는 공통적인 기반 컨테이너를 만드는 것이다.  

공통적인 기반 컨테이너에는 운영체제에서 필요한 것들을 포함해 의존성을 가진 모든 것들을 설치할 수 있어야한다.  

따라서 컨테이너를 사용하면 기존에 있는 다른 라이브러리들의 의존성에 따라 실패하는 상황을 줄일 수 있다.  

즉, 컨테이너는 공통의 기반 컨테이너를 통해 애플리케이션이 표준적인 실행방법을 갖도록 도와주며  

환경에 따른 시나리오를 재현하거나 새로운 멤버의 개발환경 구축과 같은 것들을 줄여 개발 프로세스를 수월하게 한다.  



## 유스케이스

애플리케이션 컴포넌트를 어떻게 구성해야하는지를 예제를 통해 확인해보자.  

예제는 음식을 배달하는 애플리케이션이며, 배달 상태를 추적하는 서비스를 가지고 있다.  

애플리케이션에서 REST API로 특정 주문의 상태를 질문하면 자세한 설명이 포함된 JSON응답을 반환한다고 하자.  

다음과 같은 두 가지에 초점을 맞추고 추상화 및 캡슐화를 통해 파이썬 패키지로 만들어보자.  

1. 특정 주문에 대한 정보를 얻고 클라이언트에 제공
2. 유지보수가 쉽고 확장가능한 형태

![주요로직](https://user-images.githubusercontent.com/55227276/175473787-20a67b06-5544-4246-a6a3-e8a7068f470e.png)

### 코드

#### 도메인

```python
from typing import Union

class DispatchedOrder:
    """방금 수신한 배달 주문"""
    status = "dispatched"
    def __ init __ (self, when):
        self._when = when
        
    def message(self) -> dict:
        return {
            "status": self.status,
            "msg" : "주문 시각 {0}" .format(
                self . _w hen . isoformat ( )
            ),
        }
        
class OrderlnTransit:
    """배달 중인 주문"""
    status = "in transit"
    def __ init __ (self, current_location) :
        self . _ current_ location = current_ location
        
    def message(self) -> dict:
        return {
            " status " : self.status ,
            "msg": "배달중 ...（현재 위치: {})".format(self._current_location),
        }
        
class OrderDelivered:
    """배달 완료 주문"""
    status = "delivered "
    def __ init __ (self, delivered_at):
        self._delivered_at = delivered_at
        
    def message(self) -> dict:
        return {
            "status": self.status,
            "msg" : "배달 완료 시각 {0}" . format(self._delivered _a t.isoformat()),
        }
        
class DeliveryOrder :
    def __ init __ (
        self,
        delivery_id: str,
        status: Un ion[DispatchedOrder, OrderlnTrans it, OrderDeli vered],
    ) -> None:
        self._delivery_id = delivery_ id
        self._status = status
        
    def message(self) -> dict:
        return {"id" : self._delivery_id, **self._status.message()}
```

> 이 코드를 보는 것만으로 클라이언트의 모습을 상상할 수 있음
>
> 클라이언트는 내부 협업 객체로, `_status` 를 가지고 있는 `DelivertyOrder`객체를 생성할 것이며, `message()` 메서드를 통해 상태 정보를 사용자에게 전달할 것으로 보임



#### 애플리케이션

```python
from storage import DBClient, DeliveryStatusQuery, OrderNotFoundError
from web import NotFound, Vi ew, app, register_route

class DeliveryView(View) :
    async def _get(self, request, delivery_ id: int):
        dsq = DeliveryStatusQuery(int(delivery_id), await DBClient())
        try:
            result = await dsq.get()
        except OrderNotFoundError as e:
            raise NotFound(str(e)) from e
        return result.message()
    
register_route(DeliveryView, "/status/<delivery_ id:int>")
```

> 본 코드에서 어떤 프레임워크가 사용되었는지, 데이터가 어떤 곳(파일, DB 등)에서 왔는지 등에 대해 알 수 있는가?
>
> 이런 것을 알 수 없다는 이유는 이런 것들이 세부사항에 해당하며, 이 세부사항들은 캡슐화되어야하는 것이 맞기 때문
>
> 패키지를 내부까지 살펴보지 않으면 알 수 없는 것 들임

이처럼 추상화는 코드를 조금 더 선언적으로 만든다.  

선언형 프로그래밍(declarative programming)에서는 해결하려는 문제가 아닌 해결하는 방법을 선언한다.  

결과(배달 상태)를 객체에 로드하려는 작업을 하려면 선언형 프로그래밍에서는 그저 배달 상태를 알고 싶다고 선언만 하면 된다.



#### 어댑터 디자인 패턴

우리는 패키지 내부를 보지 않고도 패키지가 애플리케이션의 세부 기술에 대한 인터페이스처럼 동작할 것이라고 생각하고 있다.  

이 애플리케이션은 패키지 내부 객체에 어뎁터 디자인 패턴의 구현이 있을 것이라고 생각할 수 있다.  

애플리케이션에서 사용하는 의존성은 반드시 API를 따라야 하며, 이는 어댑터 패턴을 통해 이룰 수 있기 때문이다.  



### 서비스

서비스를 만들기 위해 도커 컨테이너에서 파이썬 애플리케이션을 시작하자. 

컨테이너는 운영체제 수준의 의존성을 포함해 필요한 모든 의존성을 설치해야한다.  

도커 컨테이너 디렉토리 구조를 먼저 살펴보자.

```
├── Dockerfile
├── libs
|   ├── README.rst
|   ├── storage
|   └── web
├── Makefile
├── README.rst
├── setup.py
└── statusweb
    ├── __init__.py
    └── service.py
```

> libs: 의존성이 위치하는 장소
>
> Makefile, setup.py: 명령어를 포함
>
> statusweb: 애플리케이션 파일 포함

애플리케이션을 설명하는 다음과 같은 setup.py파일이 있다고 하자.

```python
from setuptools import find_packages, setup

with open("README.rst", "r") as longdesc:
    long_description = longdesc.read( )

install_requires = ["web", "storage"]

setup (
    name="delistatus",
    description="배달 상태 확인",
    long_description=long_description ,
    author="개발팀",
    version="0.1.0",
    packages=find_ packages(),
    install_requires=install_requires,
    entry_points={
        "console_scripts": [
            "status-service = statusweb.service:main"
        ],
    },
)
```

> setup함수의 install_requires인자는 의존성을 선언하는 부분에서 패키지를 만들 때 필요로하는 것을 모두 설치해야 하므로 작성
>
> setup함수의 entry_points 인자는 진입점을 만드는 것이며, 패키지 설치 시 모든 의존성과 함께 이 디렉토리를 공유, 등호의 왼쪽(status-service)이 진입점의 이름임

가상환경은 의존성을 포함하고 있는 특정 프로젝트의 디렉토리 구조이며 다음 두 가지 하위 디렉토리에 대해 알아보자.  

```
<virtual-env-root>/lib/<python-version>/site-packages
<virtual-env-root>/bin
```

> lib에는 해당 가상환경에 설치된 모든 라이브러리가 들어있음(예제에서는 web, storage패키지와 이 두 패키지의 의존성 패키지가 여기에 있을 것임)
>
> bin에는 바이너리 파일과 해당 가상환경이 활성화 상태일 때 사용할 수 있는 명령어가 들어있음, 바이너리 파일은 진입점을 선언하면 배치됨

마지막으로Dockerfile의 정의를 살펴보자.

```dockerfile
FROM python:3.6.6-alpine3.6
RUN apk add --update \
    python-dev \
    gcc \
    musl-dev \
    make
WORKDIR /app
ADD . /app

RUN pip install /app/libs/web/app/libs/storage
RUN pip install /app

EXPOSE 8080
CMD ("/usr/local/bin/status-service"]
```

> 파이썬 이미지를 기반으로하고, 운영체제 의존성을 설치
>
> `pip install`명령으로 작업 디렉토리에 애플리케이션 복사
>
> 도커 진입점이 패키지의 진입점 호출

많은 서비스와 의존성을 가진 복잡한 시나리오에서는 단순 컨테이너 이미지를 실행하지 않고 `docker-compose.yml`파일을 선언하여 실행한다.  

`docker-compose.yml`파일에는 모든 서비스와 기본 이미지, 상호작용 등을 정의할 수 있다.  

  

### 분석

구현된 예제를 통해 결론을 내려보자.  

어떤 아키텍처나 구현도 완벽하지 않으며, 이와 같은 솔루션이 모든 경우에 유용할 수는 없고 또 여러 환경에 상당 부분 의존하게 된다.  

소프트웨어 엔지니어링에는 한계가 있으며, 이 책에서 설명한 원리를 준수하는 것이 불가능하거나 실용적이지 않을 수 있다.  

염두할 것은 이 책에서 언급하는 것은 원칙일 뿐 법이 아니라는 것이다.  

프레임워크를 추상화하는 것이 불가능하거나 실용적이지 않더라도 문제가 되지 않는다.  

마지막으로 파이썬의 철학을 다시 새겨보자.  

_**Parctical beats Purity**_

