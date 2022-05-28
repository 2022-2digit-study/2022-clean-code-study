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

