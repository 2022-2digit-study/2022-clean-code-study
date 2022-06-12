# [Chapter. 7] 제네레이터 사용하기

**제너레이터는 한 번에 하나씩 구성요소를 반환해주는 이터러블을 생성해주는 객체**이다. <br>
> 제너레이터는 파이썬에서 고성능이면서도 메모리를 적게 사용하는 반복을 위한 방법으로 (PEP—255) 아주 오래 전 2001년에 소개되었다. 

## 목적

메모리를 절약하는 것. 거대한 요소를 한꺼번에 메모리에 저장하는 대신 특정 요소를 어떻게 만드는지 아는 객체를 만들어서 필요할 때마다 하나씩만 가져오도록 한다.<br>
<br>
이는 하스켈과 같은 다른 함수형 프로그래밍 언어가 제공하는 것과 비슷한 방식으로 게으른 연산(lazy computation)을 통해 무거운 객체를 시용할 수 있도록 한다.<br>
게으른 연산의 특성을 가졌기 때문에 무한 시퀀스를 사용할 수도 있다.<br>

## 제네레이터를 사용하지 않았을 때의 사용 예

* CSV 필드에는 <purchase_date>, <price> 두 개의 필드만 존재.
* 모든 구매정보를 받아 필요한 지표를 구해주는 객체
* for 루프의 각 단계에서 지표를 업데이트 하면서 구현
    
```python
class PurchasesStats:
    def __init__(self, purchases):
        self.purchases = iter(purchases) 
        self.min_price: float = None 
        self.max_price: float = None 
        self._total_purchases_price: float = 0.0 
        self._total_purchases = 0
        self._initialize()
        
    def _initialize(self): 
        try:
            first_value = next(self.purchases) 
        except Stoplteration:
            raise ValueError("no values provided")
            
        self.min_price = self.max_price = first_value 
        self._update_avg(first_value)
        
    def process(self):
        for purchase_value in self.purchases:
            self._update_min(purchase_value) 
            self._update_max(purchase_value)
            self._update_avg(purchase_value)
        return self

    def _update_min(self, new_value: float):
        if new_value < self.min_price: 
            self.min_price = new_value

    def _update_max(self, new_value: float): 
        if new_value > self.max_price: 
            self.max_price = new_value
    
    @property
    def avg_price(self) :
        return self._total_purchases_price / self._total_purchases

    def _update_avg(self, new_value: float): 
        self._total_purchases_price += new_value 
        self._total_purchases += 1

    def __str__(self): 
        return (
            f"{self.__class__.__name__}({self .min_ price}, ",
            f"{self.max_price}, {self.avg_price})"
        )
```
    
위에서 만든 객체 `PurchasesStats`은 모든 구매정보(`purchases`)를 받아서 필요한 계산을 하도록 했다. <br>

아래는 만들어진 모든 정보를 로드해서 어딘가에 담아서 반환하는 함수이다.

```python
def _load_purchases(filename): 
    purchases = []
    with open(filename) as f: 
        for line in f:
            *_, price_raw = line.partition(",") 
            purchases.append(float(price_raw))
    
    return purchases
```
위 함수 `_load_purchases`은 `filename`을 가진 파일에서 모든 정보를 읽어서 리스트에 저장한다.

### 문제점

위 객체와 함수는 문제점이 있다. 모든 데이터를 리스트로 불러와서 처리한다는 점이다. 

파일의 내용을 리스트로 불러와서 담는 과정에서 비효율을 느낄 수 있다. 

왜냐하면 위 객체에서 데이터를 처리하는 방식이 한번에 한 행만을 처리하도록 구현되었기 때문인데 특히 위와 같은 예시(`_load_purchases()`)는 메모리가 제한적인 마이크로 프로세서와 같은 상황에서 더더욱 절망적일 수 있다.
	
많은 데이터를 불러와야 하는데, 리스트에 모두 담지 못하는 경우가 생길 수 있기 때문이다.

## 제네레이터를 사용한 예
	
제네레이터는 이러한 상황에서 구원을 해줄 수 있을 것이다. 파일의 전체 내용을 리스트에 보관하는 대신, 필요한 값만 그 때 그 때 가져오는 것이다.

```python
def load_purchases(filename): 
	with open(filename) as f:
	for line in f:
		*_, price_raw = line.partition(",") 
		yield float(price一raw)
```

위와 달라진 점이 크게 없다. 단지 거대한 정보를 담고 있던 `purchases`가 사라졌으며, `return` 키워드 대신 `yield` 키워드가 붙어있을 뿐이다.

함수를 대화형 인터프리터에서 실행시켜 보자.
```python
>>> load_purchases("file")
<generator object load_purchases at 0x...>
```

반환 값이 리스트 형식이 아닌 `generator object` 라는 새로운 형식으로 반환됨을 알 수 있다.
	
사실 바뀐건 `yield` 밖에 없는데 말이다. 
	
그렇다. 어떤 함수던 `yield` 키워드를 사용하면 제네레이터 함수가 되며, `yield`가 포함된 함수를 호출하면 제네레이터의 인스턴스를 만든다.

주목할 점은 이 함수의 사용코드가 그대로라는 점이다. 위의 `PurchasesStats`은 제네레이터를 사용해도 그대로 사용할 수 있으며, 지표 역시 그대로 사용할 수 있다.

이는 `for loop`이 알다시피 이터러블을 지원하는 객체를 순회하기 때문이며, 
`generator`은 이터러블하기 때문이다. 이터러블 인터페이스를 따르면 투명하게 요소를 반복하는 것이 가능하다.

## 제네레이터 표현식

앞서 알아본 것처럼 제네레이터를 사용하면 많은 메모리를 절약할 수 있다. 
제네레이터는 이터레이터이므로 `list`나, `tuple`, `set`처럼 많은 메모리를 필요로 하는 `iterable`이나, `container`의 대안이 될 수 있다.

`comprehension`에 의해 정의될 수 있는 `list`, `set`, `dictionary`처럼 제네레이터도 제네레이터 표현식으로 정의할 수 있다.
> _제네레이터 표현식을 Generator Comprehension으로 불러야 한다는 주장도 있다. 아래에서는 표준 이름을 따라 제네레이터 표현식이라고 칭한다._

```python
# List Comprehension
>>> [x**2 for x in range{10)]
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
	
# Generator Expression
>>> (x**2 for x in range(10)) 
<generator object <genexpr> at 0x...>

# Generator Expression in Function using Iterable Parametor
>>> sum(x**2 for x in range(10)) 
285
```
	

## The Ideal Iteration

### `enumerate()`
흔히 사용하는 함수로 enumerate() 함수가 있다. 보통 리스트를 순회할 때 인덱스를 함께 이용하기 위해서 사용하며 실제 사용 예는 아래와 같다.

```python	
>>> list(enumerate("abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]
```
	
내부 동작을 살펴보기 위해서 조금 더 저수준에서 이와 유사한 객체를 만들어 보자. 이 객체는 단순히 시작값을 입력하면 무한 시퀀스를 만드는 역할을 할 것이다.
	
```python
class NumberSequence:
	def __ init__ (self, start=0):
	self .current = start
	
	def next(self):
		current = selfq.current 
		self.current += 1 
		return current
```
이 인터페이스에 기반을 두어 클라이언트를 작성하면 명시적으로 `next()` 함수를 호출해야 한다.
```python
>>> seq = NumberSequence() 
>>> seq.next()
0
>>> seq.next()
1
>>> seq2 = NumberSequence(10) 
>>> seq2.next()
10
>>> seq2.next()
11
```

그러나 이 코드로 enumerate() 함수를 사용하도록 재작성할 수는 없다. 

왜냐하면 일반 파이썬 의 for 루프를 사용하기 위한 인터페이스를 지원하지 않기 때문이다. 
이는 또한 이터러블 형태 의 파라미터로는 사용할 수 없다는 것을 뜻한다. 

```python
>>> list(zip(NumberSequence(), "abcdef")) 
Traceback (most recent call last):
File "... line 1, 1n <module>
TypeError: zip argument #1 must support iteration
```

이 문제를 해결하려면 `__iter__()` 매직 메서드를 구현하여 객체가 반복 가능하게 만들어야한다. 
또한 `next()` 메서드를 수정 하여 `__next__()` 매직 메서드를 구현하면 객체는 이터레이터가 된다.

#### 매직 매서드를 통한 구현
```python
class SequenceOfNumbers:
	def __init__(self, start=0): 
		self.current = start
	
	def __next__(self):
		current = self.current 
		self.current += 1 
		return current
	
	def __iter__(self): 
		return self
```

이렇게 하면 요소를 반복할 수 있을 뿐 아니라 `.next()` 메서드를 호출할 필요도 없다. 
	
왜냐하면 **`__next__()` 메서드를 구현했으므로 `next()` 내장 함수를 사용할 수 있기 때문**이다.
	
#### 호출
```python
>>> list(zip(SequenceOfNumbers(), "abcdef"))
[(0, 'a'), (1, 'b'), (2, 'c'), (3, 'd'), (4, 'e'), (5, 'f')]
>>> seq = SequenceOfNumbers(100) 
>>> next(seq) # 객체 멤버 함수가 아님: 파이썬 내장함수 next()
100
>>> next(seq)
101
```

_근데 next() 함수는 뭘까??_

### `next()` Method

사실 파이썬의 for문은 정말 특이한 형태이며, 다른 언어의 for문과는 꽤 다르다.

다른 언어를 먼저 배웠다면 해당 형식의 for문은 매우 익숙할 것이다.
	
#### C++, Java
```cpp
int[] nums = {1, 2, 3, 4, 5};
for(int = 0; i < 5; i++){
	cout << nums[i] << endl;
}
```

#### JavaScript
```js
let nums = [1, 2, 3, 4, 5];
for(let i = 0; i < 5; i++){
	console.log(nums[i])
}
```

#### Python
```python
nums = [1, 2, 3, 4, 5]
for(i in range(len(nums))):
	print(nums[i])
```

먼저 위의 코드들은 배열의 인덱스를 통해 접근하는 코드로 출력은 모두 동일하다.
	
단 접근 방식에서 매우 큰 차이가 있는데, javascript, C++과 같은 언어들의 경우는 `i`가 정해지는 주체가 변수이다.
즉 `i`라는 변수가 있고, for loop가 한번 수행될 때마다 `i`변수가 1씩 더해가면서 인덱스를 지정하는 방식이다.
i가 기본형 변수(primitive variable)이기 때문에 i를 조건에 따라 중간에서 수정하거나, 갱신하는 등의 작업이 가능하다.
	
하지만 파이썬은 다르다. 파이썬은 `range(len(nums))` 함수에서 `0 ~ len(nums)`범위의 리스트를 생성한 후 `i`를 해당 변수에 지정하여 순회하는 형식이다.
따라서 `range(len(nums))`는 `[0, 1, 2, 3, 4]`와 같은 반복 가능한 객체이며, 
i를 루프 안에서 수정한 들, 다음 루프에는 다음 순회할 변수가 새롭게 지정되기 떄문에 이전의 수정은 영향을 주지 못한다.


따라서 next() 메서드는 파이썬의 특별한 형식에 알맞게 이터레이터를 다음으로 이동시키는 함수이다. 
즉, 이터러블(`i`)을 다음 요소로 이동시키고 기존의 값을 반환한다.

```python
>>> word = iter{"hello")
>>> next(word)
'h'
>>> next(word)
'e' 
>>> ...
>>> next(word)
'o'
>>> next(word)
Traceback (most recent call last):
	File "<stdin>" , line 1, in <module> 
StopIteration
>>>
```
이터레이터가 더이상의 값을 가지고 있지 않다면 `StopIteration` 예외가 발생한다. 
이 예외는 반복이 끝났음을 나타내며, 사용할 수 있는 요소가 더 이상 없음을 나타낸다.

이 문제를 해결하고 싶다면 `StopIteration` 예외를 캐치하는 것 외에도 `next()` 함수의 두 번째 파라미터에 기본 값을 제공할 수도 있다. 
이 값을 제공하면 `StopIteration을` 발생시키는 대신 기본 값을 반환한다.

```python
>>> next(word, "default value") 
'default value'
```

## `itertools` 모듈

itertools 모듈을 사용하여 이터러블을 조금 더 파이썬에 맞게 사용할 수 있다.

만약 특정 기준을 넘은 값에 내해서만 연산을 하려면 어떻게 해야 할까? 가장 간단한 방법은 while문 안에 조건문을 추가하는 것이다.

```python
# ...
	def process(self):
		for purchase in self.purchases: 
			if purchase > 1000.0:
```
	
이것은 파이썬스럽지 않을 뿐 아니라, 나쁜 코드이다. 이것은 수정사항을 잘 반영할 수 없다. 

위의 코드는 수정할 때 다음과 같은 고민을 하게 할 수 있을 것이다.

1. 만약 기준수치가 변경된다면 파라미터로 전달해야 할까? 
2. 기준 수치가 둘 이상 필요하다면? 
3. 만약 조건이 특정 기준 이하가 되면 어떻게 해야할까? 
4. 람다함수를 넘겨야 하나?

클린 코드는 융통성이 있어야 하고 외부 요인에 결합력이 높아서는 안 된다. 이러한 요구사항은 다른 곳에서 해결되어야 한다.

코드를 수정하는 대신 그대로 유지하고 클라이언트 클래스의 요구사항이 무엇이든 그에 맞게 필터링하여 새로운 데이터를 만든다고 가정하자.

> 예를 들어 1,000개 넘게 구매한 이력의 처음 10개만 치리하려고 하면 다음과 같이 하면 된다.

```python
>>> from itertools import islice
>>> purchases = islice(filter(lambda p: p > 1000.0, purchases), 10) 
>>> stats = PurchasesStats(purchases).process() # ...
```
> `filter(condition_function, iterable_object)`: `iterable_object`에서 `condition_function` 함수의 조건에 맞는 요소를 이터러블한 형태로 반환한다.

> `itertools.islice(iterable_object, start, end)`: `iterable_object`에서 `start ~ end`까지의 인덱스에 해당하는 객체를 가져온다.

이런 식으로 필터링을 해도 메모리의 손해는 없다. 왜냐하면 모든 것이 제너레이터이므로 게으르게 평가된다. 

즉 마치 전체에서 필터링한 값으로 연산을 한 것처럼 보이지만, 실제로는 하나 씩 가져와서 모든 것을 메모리에 올릴 필요가 없는 것이다.
