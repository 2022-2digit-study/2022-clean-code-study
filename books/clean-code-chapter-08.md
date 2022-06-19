# [Chapter 8. 단위 테스트와 리팩토링]





## 디자인 원칙과 단위 테스트

`단위 테스트`는 다른 코드의 일부분이 유효한지를 검사하는 코드이다. 

`단위 테스트`는 애플리케이션의 핵심을 검증하는 보조 수단이 아니라, 소프트웨어의 핵심이 되는 필수적인 기능이다.  

`단위 테스트`는 단위 테스트를 작성한 `.py` 파일을 만들고 이 파일을 도구에서 호출하는 방식으로 실행할 수 있다.

테스트 도구에서 파일의 내용을 호출하면 테스트가 실행되며, 테스트가 실패하면 프로세스는 오류 코드와 함께 종료된다.  

(일반적으로 테스트에서 성공하면 점(.)을, 실패하면 F를, 예외가 있으면 E를 출력함)



### 단위 테스트의 특징

`단위 테스트`는 다음과 같이 세 가지 특징이 있다.  

**1. 격리**

- 다른 외부 에이전트와 완전히 독립적이어야 함.

  (즉, 데이터베이스 연결이나 HTTP 요청을 하지 않아야 함)

- 테스트 자체가 독립적이어야 함.

  (즉, 이전 상태에 관계없이 임의의 순서로 실행될 수 있어야 함)

**2. 성능**

- 신속하게 실행되어야 함
- 반복적으로 여러 번 실행될 수 있도록 설계해야 함

**3. 자체 검증**

- 단위 테스트의 실행만으로 결과를 결정할 수 있어야 함

  (즉, 단위 테스트를 처리하기 위한 추가 단계가 없어야 함)



### 다른 자동화 테스트

이 책의 범위를 벗어나지만 다음과 같은 자동화 테스트도 있다.  

#### 통합 테스트(integtration test)

한 번에 여러 컴포넌트를 테스트하며, HTTP 요청이나 데이터베이스 연결 등 다양한 환경이 제대로 작동하는지 확인하는데 필요한 모든 작업을 수행하는 테스트이다.  

단위 테스트에서 발견하기 어려운 환경버그 등을 찾을 수 있다.  

#### 인수 테스트(acceptance  test)

유스케이스(use case)를 활용하여 사용자의 관점에서 시스템의 유효성을 검사하는 테스트이다.  

비지니스에 초점을 두고 다른 의사소통집단(기획자 등)으로부터 시나리오를 인수받아 개발한다.  

소프트웨어를 인수하기 전 명세한 요구사항대로 잘 작동하는지 검증하는 것이다.  

  

  

이 두 가지 테스트가 이 책의 범위를 벗어나는 이유는 단위 테스트의 중요한 특성인 `속도`를 잃게 되기 때문이다.  

개발자는 테스트 스위트를 만들고 코드에 수정이 생길 때마다 반복적으로 단위 테스트와 리팩토링을 할 수 있어야 한다.  

하지만 이 두 테스트는 실행하는데 많은 시간이 걸리며 따라서 덜 자주 실행한다.  

[^테스트 스위트]: 함수나 메서드를 테스트하는 단위 테스트와 달리 클래스를 테스트하는 것, 여러 단위 테스트의 집합임



### 단위 테스트와 애자일 소프트웨어 개발

애자일 소프트웨어 개발 방법론의 핵심은 **작동하는 소프트웨어의 작은 구성요소를 신속하게 지속적으로 제공하여 고객의 만족도를 개선하는 것**이다.  

이를 위해서 변화에 효과적이며 유연하고 확장 가능한 소프트웨어를 개발해야 하고 

단위 테스트는 이 확장가능한 소프트웨어를 변경한다고 했을 때  변경작업이 명세에 따라 정확하게 동작한다는 것에 대한 증거가 될 수 있다.  

따라서 단위 테스트는 애자일 소프트웨어 개발 방법론을 따르는데 도움이 될 수 있다.  



## 테스트를 위한 프레임워크와 도구

### 단위 테스트 프레임워크와 라이브러리

예제를 통해 단위 테스트를 작성하고 실행하기 위한 두 가지 프레임 워크에 대해 살펴보자.  

```python
from enum import Enum
 
class MergeRequestStatus(Enum):
    OPEN = "open"
    APPROVED = "approved"
    REJECTED = "rejected"
    PENDING = "pending"
    CLOSED = "closed"
    
class MregeRequest:
    def __init__(self):
        self._context = {
            "upvotes": set(),
            "downvotes": set(),
        }
        self._status = MergeRequestStatus.OPEN
        
    def close(self):
        self._status = MergeRequestStatus.CLOSED
            
    @property
    def status(self)::
        if self._context["downvotes"]:
            return MergeRequestStatus.REJECTED
        elif len(self._context["upvotes"]) >= 2:
            return MergeRequestStatus.APPROVED
        return MergeRequestStatus.PENDING
        
    def _cannot_vote_if_closed(self):
        if self._status == MergeRequestStatus.CLOSED:
            raise MergeRequestException("종료된 머지 리퀘스트에 투표할 수 없음")
        
    def upvote(self, by_user):
        self._cannot_vote_if_closed()
        self._context["downvotes"].discard(by_user)
        self._context["upvotes"].add(by_user)
        
    def downvote(self, by_user):
        self._cannot_vote_if_closed()
        self._context["upvotes"].discard(by_user)
        self._context["downvotes"].add(by_user)    
```

> 한 명 이상의 사용자가 변경 내용에 동의하지 않은 경우 merge request가 reject됨
>
> 아무도 반대하지 않은 상태에서 두 명 이상의 개발자가 동의하면 merge request가 approved됨
>
> 이외의 상태는 pending임
>
> 사용자가 머지 리퀘스트를 종료할 수 있으며 종료 시 더이상 투표할 수 없음, 투표 시 에러 발생



#### unittest

테스트 시나리오를 다루기에 적합하며, 모든 종류의 테스트를 작성할 수 있는 풍부한 API를 제공하여 단위 테스트에 적합하다.  

unittest 모듈은 JUnit을 기반으로 하기 때문에 객체 지향적이며, 따라서 테스트는 객체를 사용해 작성된다.  

`unittest.TestCase`를 상속하여 테스트 클래스를 만들며 `test_`로 시작하는 메서드에 테스트할 조건을 정의하면 된다.  

unittest API는 비교를 위해 다양한 메서드를 제공하며, `assertEquals(<actual>, <expected>[, message])`와 같이 에러가 발생한 경우에 대한 메세지를 지정할 수도 있다.  

| Method                                                       | Checks that          | New in |
| ------------------------------------------------------------ | -------------------- | ------ |
| assertEqual(a, b)                                            | a == b               |        |
| assertNotEqual(a, b)                                         | a != b               |        |
| assertTrue(x)                                                | bool(x) is True      |        |
| assertFalse(x)                                               | bool(x) is Fasle     |        |
| assertIs(a, b)                                               | a is b               | 3.1    |
| assertIsNot(a, b)                                            | a is not b           | 3.1    |
| assertIsNone(x)                                              | x is None            | 3.1    |
| asserIsNotNone(x)                                            | x is not None        | 3.1    |
| assertIn(a, b)                                               | a in b               | 3.1    |
| assertNotIn(a, b)                                            | a not in b           | 3.1    |
| assertIsInstance(a, b)                                       | isinstance(a, b)     | 3.2    |
| assertNotIsInstance(a, b)                                    | not isinsatnce(a, b) | 3.2    |
| assertRaises(exception, callable, *args, **kwds)             |                      | 3.1    |
| assertRaisesRegex(exception, regex, callable, *args, **kwds) |                      | 3.1    |
| assertWarns(warning, callable, *args, **kwds)                |                      | 3.2    |
| assertWarnsRegex(warning, regex, callable, *args, **kwds)    |                      | 3.2    |

아래는 unittest 라이브러리를 이용하여 단위테스트를 실행하는 예제이다.  

```python
class TestMergeRequestStatus(unittest.TestCase):
    
    def test_simple_rejected(self):
        merge_request = MergeRequest()
        merge_request.downvote("maintainer")
        self.assertEqual(merge_request.status, MergeRequestSataus.REJECTED)
        
    def test_just_created_is_pending(self):
        self.assertEqual(MergeRequest().status, MergeRequestSataus.PENDING)
    
    def test_pending_awaiting_review(self):
        merge_request = MergeRequest()
        merge_request.upvote("core-dev")
        self.assertEqual(merge_request.status, MergeRequestSataus.PENDING)
    
    def test_approved(self):
        merge_request = MergeRequest()
        merge_request.upvote("dev1")
        merge_request.upvote("dev2")
        self.assertEqual(merge_request.status, MergeRequestSataus.APPROVED)
    
    def test_cannot_upvote_on_closed_merge_request(self):
        self.merge_request.close()
        self.assertRaises(
        	MergeRequestException, self.merge_request.upvote, "dev1"
        )
        
    def test_cannot_down_vote_on_closed_merge_request(self):
        self.merge_request.close()
        self.assertRaisesRegex(
        	"종료된 머지 리퀘스트에 투표할 수 없음", self.merge_request.upvote, "dev1"
        )
```

##### 테스트 파라미터화

이제 머지 리퀘스트가 정상동작하는지 테스트한다고 할 때,   

가장 좋은 방법은 컴포넌트를 다른 클래스로 분리하고 컴포지션을 사용해 가져오는 것이다.  

따라서 다음과 같이 변형하여 테스트를 진행한다.  

```python
class AcceptanceThreshold:
    def __init__(self, merge_request_context: dict) -> None:
        self._context = merge_request_context
        
    def satatus(self)::
        if self._context["downvotes"]:
            return MergeRequestStatus.REJECTED
        elif len(self._context["upvotes"]) >= 2:
            return MergeRequestStatus.APPROVED
        return MergeRequestStatus.PENDING

class MregeRequest:
    ...           
    @property
    def status(self)::
        if self._status == MergeRequestStatus.CLOSED:
            return self._status
        
        return AcceptanceThreshold(self._context).status()
```

이를 통해 새로운 클래스에 다음과 같이 테스트를 작성할 수 있다.  

```python
class TestAcceptanceThreshold(unittest.TestCase):
    def setUp(self):
        self.ficture_data = (
            (
                {"downvotes": set(), "upvotes": set()},
                MergeRequestStatus.PENDING
            ),
            (
                {"downvotes": set(), "upvotes": {"dev1"}},
                MergeRequestStatus.PENDING
            ),
            (
                {"downvotes": "dev1", "upvotes": set()},
                MergeRequestStatus.REJECTED
            ),
            (
                {"downvotes": set(), "upvotes": {"dev1", "dev2"}},
                MergeRequestStatus.APPROVED
            ),
        )
        
    def test_status_resolution(self):
        for context, expected in self.fixture_data:
            with self.subTest(context=context):
                status = AcceptanceThreshold(context).status()
                self.assertEqual(status, expected)
```

> setUp() 메서드에 테스트 전반에 걸쳐 사용될 데이터 픽스처를 정의



#### pytest

`pytest`는 테스트 프레임워크로 `assert`구문을 사용해 조건을 검사하는 것이 가능하기 때문에 보다 자유롭다.  

또한 `pytest` 명령어를 통해 탐색가능한 모든 테스트를 한 번에 실행한다.(unittest로 작성한 테스트까지..!)

이전의 예제는 `pytest`를 통해 다음과 같이 다시 작성할 수 있다. 

```python
def  test_simple_rejected():
    merge_request = MergeRequest()
    merge_request.downvote("maintainer")
    assert merge_request.status = MergeRequestStatus.REJECTED
    
def test_just_created_is_pending():
    assert MergeRequest().status == MergeRequestSataus.PENDING
    
def test_pending_awaiting_review():
    merge_request = MergeRequest()
    merge_request.upvote("core-dev")
    assert merge_request.status == MergeRequestSataus.PENDING
    
def test_invalid_types():
    merge_request = MergeRequest()
    pytest.raises(TypeError, merge_request.upvote, {"invalid-object"})
    
def test_cannot_vote_on_closed_merge_request(self):
    merge_request = MergeRequest()
    merge_request.close()
    pytest.raises(MergeRequestException, merge_request.upvote, "dev1")
    with pytest.raises(
    	MergeRequestException,
        match = "종료된 머지 리퀘스트에 투표할 수 없음",
    ):
        merge_request.downvote("dev1")
```

##### 테스트 파라미터화

pytest로 파라미터화 된 테스트를 하려면 `pytest.mark.parametrize`데코레이터를 사용해야한다.  

데코레이터의 첫 번째 파라미터는 테스트 함수에 전달할 파라미터의 이름을 나타내는 문자열이며,  

두 번째 파라미터는 해당 파라미터에 대한 각각의 값이며 반복 가능해야 한다.  

``` python
@pytest.mark.parametrize("context, expected_status", (
    (
        {"downvotes": set(), "upvotes": set()},
        MergeRequestStatus.PENDING
    ),
    (
        {"downvotes": set(), "upvotes": {"dev1"}},
        MergeRequestStatus.PENDING
    ),
    (
        {"downvotes": "dev1", "upvotes": set()},
        MergeRequestStatus.REJECTED
    ),
    (
        {"downvotes": set(), "upvotes": {"dev1", "dev2"}},
        MergeRequestStatus.APPROVED
    ),
))
        
def test_acceptance_threshold_status_resolution(context, expected_status):
    assert AcceptanceThreshold(context).status() == expected_status
```



##### 픽스처(Fixture)

`fixture`는 테스트 사전/사후에 사용가능한 리소스 또는 모듈을 뜻하는 것으로,  

pytest에서는 재사용하고자하는 객체를 정의하고 `@pytest.fixture` 데코레이터를 적용할 경우  

이 픽스처를 사용하기 원하는 테스트에 파라미터로 픽스처의 이름을 전달하면 pytest가 이를 활용할 수 있다.  

```python
@pytest.fixture
def rejected_mr():
    merge_request = MergeRequest()
    
    merge_request.downvote("dev1")
    merge_request.upvote("dev2")
    merge_request.upvote("dev3")
    merge_request.downvote("dev4")
    
    return merge_request

def test_simple_rejected(rejected_mr):
    assert rejected_mr.status == MergeRequestStatus.REJECTED
    
def test_rejected_with_approvals(rejected_mr):
    rejected_mr.upvote("dev2")
    rejected_mr.upvote("dev3")
    assert rejected_mr.status == MergeRequestStatus.REJECTED
    
def test_rejected_to_pending(rejected_mr):
    rejected_mr.upvote("dev1")
    assert rejected_mr.status == MergeRequestStatus.PENDING
    
def test_rejected_to_approved(rejected_mr):
    rejected_mr.upvote("dev1")
    rejected_mr.upvote("dev2")
    assert rejected_mr.status == MergeRequestStatus.APPROVED
```



### 코드 커버리지

`Code Coverage`는 테스트 도중 코드의 어떤 부분이 실행되었는지(얼마나 많은 코드가 실행되었는지) 알려주는 지표이다.  

이는 테스트에서 어떤 부분을 다뤄야할지, 어떤 부분이 개선되었는지를 알 수 있게 해준다.  

특히, 파이썬같이 직접 실행해보지 않으면 에러를 확인하기 어려운 인터프리터 언어에서 매우 중요하다.  

여러 기능 중 가장 권장되는 것은 테스트되지 않은 행을 알려주는 기능이며,   

이를 통해 커버되지 않은 코드를 확인하고 추가로 단위 테스트를 작성할 수 있다. 

즉, 코드 커버리지는 **휴먼 에러를 최대한 방지할 수 있도록 도와줄 수 있다.**

코드 커버리지가 높은 것은 좋은 것이지만, 클린 코드를 위한 조건으로는 부족하다. (이것이 맹점이라 지적하고 있음)  

왜냐하면 라인이 실행되었다는 것이 가능한 모든 조합에 대해 테스트되었다는 것을 의미하지 않기 때문이다.  

따라서 코드를 테스트하는데 있어서 사각지대를 찾기 위한 도구로 커버리지를 사용하지만, 이를 높이는 것은 궁극적인 목표가 될 수 없다.  

  

### 목(mock) 객체

목 객체는 원하지 않는 부작용으로부터 테스트 코드를 보호하는 좋은 방법이다.  

외부의 다른 서비스나 모듈 등 실제 사용하는 것들을 사용하지 않고 테스트 대상 시스템을 목 객체로 교체하여 테스트의 효용성을 높일 수 있다.  



#### Mock 객체 사용하기

단위 테스트에서의 테스트 더블에는 더미(dummy), 스텁(stub), 스파이(spy), 목(mock)과 같은 다양한 타입의 객체가 있다. 

[^테스트 더블]: 테스트 스위트에서 실제 코드를 대신해 실제인 것처럼 동작하는 코드

이 중에서 목 객체는 가장 일반적인 유형의 객체이므로 목 객체를 살펴보자.  

`목(mock) 객체`는 호출 시 응답해야하는 값이나 행동을 특정할 수 있으며  

Mock 객체는 내부에 호출 방법을 기록하고 나중에 이 정보를 사용하여 동작을 검증한다.  

파이썬 표준 라이브러리는 unittest.mock 모듈에서 `Mock`과 `MagicMock` 객체를 제공하며,  

전자는 모든 값을 반환하도록 설정할 수 있는 테스트 더블이며,   

후자는 여기에 매직 메서드까지 지원하는 테스트 더블이다.  

`Mock`객체에서 매직 메서드를 사용하려고하면 에러가 발생한다.  



##### 테스트 더블 사용 예

목객체를 사용하여 애플리케이션에 머지 리퀘스트의 빌드 상태를 알리는 컴포넌트를 추가해보자.  

빌드가 끝나면 머지 리퀘스트 아이디와 빌드 상태를 파라미터로 하여 객체를 호출하고  

그러면 특정 엔드포인트에 POST 요청을 보내 최종 머지 리퀘스트의 상태를 업데이트 하도록 하자.  

```python
from datetime import datetime

import requests
from constants import STATUS_ENDPOINT

class BuildStatus:
    """Continuous Integration 도구에서의 머지 리퀘스트 상태"""
    
    @staticmethod
    def build_date() -> str:
        return datetime.utcnow().isofoormat()
    
    @classmethod
    def notify(cls, merge_request_id, status):
        build_status = {
            "id": merge_request_id,
            "status": status,
            "build_at": cls.build_date(),
        }
        response = requests.post(STATUS_ENDPOINT, json=build_status)
        response.raise_for_status() # 200이 아닐 경우 예외 발생
        return response
```

이 클래스는 외부 모듈에 의존성이 매우 크며 API를 실제로 호출할 필요 없이 적절한 정보가 API 에 잘 전달되었는지가 중요하다.  

또한 시간을 비교하는 부분이 있는데, 이는 datetime을 패치할 수 없으므로 build_date를 정적 메서드로 래핑해야한다.  

따라서 다음과 같이 테스트를 실행할 수 있다.  

```python
from unittest import mock

from constants import STATUS_ENDPOINT
from mock_2 import BuildStatus

@mock.path("mock_2.requests")
def test_build_notification_sent(mock_requests):
    build_date = "2019-01-01T00:00:01"
    with mock.patch.("mock_2.BuildStatus.build_date", return_value=build_date):
        BuildStatus.notify(123, "OK")
        
    expected_payload = {"id": 123, "status": "OK", "build_at": build_date}
    mock_requests.post.assert_called_with(
    	STATUS_ENDPOINT, json=expected_payload
    )
```





## 리팩토링

`리팩토링`은 소프트웨어 유지 관리에서 중요한 활동이다.  

언제든 새로운 기능을 지원하기 위해서는 요구사항을 수용할 수 있도록 코드를 리팩토링하여 보다 일반적인 형태로 만드는 것이다.  

일반적으로 다음과 같은 두 가지 경우에 코드를 리팩토링한다.  

1. 구조를 개선하여 보다 나은 코드를 만드려는 경우
2. 좀 더 일반적인 코드로 수정하여 가독성을 높이려는 경우

중요한 것은 **리팩토링 이전과 이후에 컴포넌트가 동일한 기능을 유지해야한다는 것**이다.  



### 코드의 진화

단위 테스트에서 제어할 수 없는 의존성이 있는 것들을 `mock.patch`함수를 사용해 지시한 객체 대신 Mock객체를 돌려주게 하였다.  

그런데 이 방식의 단점은 모의하려는 객체의 경로를 문자열로 제공해야한다는 것이다.  

이 상태에서 리팩토링이 진행될 경우 모든 곳을 업데이트하지 않으면 테스트가 실패하게 된다.  

이는 설계상의 문제로 notify() 메서드가 구현 세부사항에 해당하는 requests에 의존하고 있다.  

따라서 이를 더 작은 메서드로 나누는 리팩토링을 통해 의존성 역전원칙을 적용하여 requests 모듈이 제공하는 것과 같은 인터페이스를 지원하도록 해야한다.  

아래의 예시를 보자.  

```python
from datetime import datetime
from constants import STATUS_ENDPOINT

class BuildStatus:
    endpoint = STATUS_ENDPOINT
    
    def __init__(self, transport):
        self.transport = transport
        
    @staticmethod
    def build_date() -> str:
        return datetime.utcnow().isoformat()
    
    def compose_payload(self, merge_request_id, status) -> dict:
        return {
            "id": merge_request_id,
            "status": status,
            "built_at": slef.build_date(),
        }
        
    def deliver(self, payload):
        response = self.trasport.post(self.endpoint, json=payload)
        response.raise_for_status()
        return response
    
    def notify(self, merge_request_id, status):
        return self.deliver(self.compose_payload(merge_request_id, status))
```

>notify를 분리하여 compose와 deliver로 나누고  
>
>compose_payload()라는 새로운 메서드를 추가하고  
>
>transport라는 의존성을 주입하였음  

필요하다면 교체된 테스트 더블을 사용한 객체의 픽스처를 노출하는 것도 가능하다.  

```python
@pytest.fixture
def build_status():
    bstatus = BuildStatus(Mock)
    bstatus.build_date = Mock(return_value="2018-01-01T00:00:01")
    return bstatus
    
def test_build_notification_sent(build_status):
    build_status.notify(1234, "OK")
    
    expected_payload = {
        "id": 1234,
        "status": "OK",
        "built_at": build_status.build_date(),
    }
    
    build_status.transport.post.assert_called_with(
    	build_status.endpoint, json=expected_payload
    )
```



## 단위 테스트에 대한 추가 논의

자동화된 단위테스트에 대해 다음과 같은 의문이 생길 수 있다.  

1. 테스트 시나리오를 충분히 검증했으며 누락된 것이 없다는 것을 어떻게 확신할 수 있는가?
2. 누가 이 테스트가 정확하다고 판단할 수 있는가?

이에 대한 답변을 알아보자.  

### 속성 기반 테스트

`속성 기반(Property-based) 테스트`는 1번 질문에 대한 답변이 될 수 있다.  

이는 테스트를 실패하게 만드는 데이터를 찾는 것이다.  

 `hypothesis`라이브러리를 통해 속성 기반 테스트를 수행할 수 있으며,  라이브러리에 코드에 유효한 가설을 정의하면 된다.  

  

### 변형 테스트

`변형 테스트`는 2번 질문에 대한 답변이 될 수 있다.  

테스트는 우리가 작성한 코드가 정확하다는 것을 입증해주는 방법이다. 그렇다면 테스트가 정확하다는 것을 입증해주는 방법은 무엇일까?  

역설적이게도 정답은 바로 상용 코드이다.  

코드에 잘못된 부분이 있었다면 이는 단위 테스트에서 실패했어야한다.  

하지만 이런 일이 발생하지 않았다는 것은 테스트에 누락된 부분이 있거나 올바른 체크가 되지 않았음을 의미한다.  

따라서 변형 테스트는 원래 코드가 돌연변이 코드(mutant)로 변형되게하고 테스트를 실행하며,  

돌연변이 코드가 실험에서 생존하면 테스트가 정확하지 않다고 판단할 수 있다.  



## 테스트 주도 개발

TDD(Test-Driven Development)은 기능의 결함으로 실패하게 될 테스트를 상용화 전에 미리 작성해야 한다는 것이다.  

테스트를 먼저 작성한 다음 코드를 작성해야 하는 이유는 **실용적인 관점에서 코드를 볼 수 있으며 기본적인 기능 테스트를 누락할 가능성이 낮아지기 때문**이다.  

테스트 주도 개발은 다음과 같은 3단계로 이뤄진다.  

1. 구현할 내용을 단위 테스트로 작성
2. 해당 조건을 충족시키는 최소한의 필수 코드 구현
3. 리팩토링을 통한 코드 개선

이 사이클을 `red-green-refactor`라고도 부른다.  
