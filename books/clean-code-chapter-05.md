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

이론적으로 함수 데코레이터는 어떤 종류의 로직에도 적용 가능하다사용 예 ㅈ
```python
class ControlledException(Exception):
  """도메인에서 발생하는 일반적인 예외"""

def retry(operation):

    @wraps(operation)
    def wrapped(*args, **kwargs):
        last_raised = None 
        RETRIES_LIMIT = 3

        for _ in range(RETRIES_LIMIT) :
            try:
                return operation(*args, **kwargs)
            except ControlledException as e:
                logger.info("retrying %s", operation.__qualname__) 
                last_raised = e
                
        raise last_raised
        
    return wrapped
```
