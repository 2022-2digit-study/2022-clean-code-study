# [Chapter. 7] 제네레이터 사용하기

영단어 generate는 "생성하다"라는 의미이다. 특정 개념과 같은 것들에서 이름은, 해당 개념의 많은 것을 의미한다. 그렇다면 generator은 무언가를 생성하는 녀석일 것이다.

**제네레이터는 한 번에 하나씩 구성요소를 반환해주는 이터러블을 생성해주는 객체**이다. <br>
> 제네레이터는 파이썬에서 고성능이면서도 메모리를 적게 사용하는 반복을 위한 방법으로 [PEP—255](https://peps.python.org/pep-0255/)에서 아주 오래 전 2001년에 소개되었다. 

지금은 이터러블이 무엇을 뜻하는지 몰라도 된다. 단 제네레이터가 생성하는 역할을 하는 객체인 것을 알았으니, 무엇을 생성하는지에 초점을 맞추어 진행하면 좋을 것이다.

## 목적

가장 큰 목적은 위에서 언급했듯이 **메모리를 절약하는 것**이다. 

거대한 요소를 한꺼번에 메모리에 저장하는 대신 특정 요소를 어떻게 만드는지 아는 객체를 만들어서 필요할 때마다 하나씩만 가져오도록 한다.<br>
<br>
이는 하스켈과 같은 다른 함수형 프로그래밍 언어가 제공하는 것과 비슷한 방식으로 게으른 연산(lazy computation)을 통해 무거운 객체를 사용할 수 있도록 한다.<br>
게으른 연산의 특성을 가졌기 때문에 무한 시퀀스를 사용할 수도 있다.<br>

## 제네레이터를 사용하지 않았을 때의 사용 예

* CSV 필드에는 `<purchase_date>`, `<price>` 두 개의 필드만 존재.
* 모든 구매정보를 받아 필요한 지표를 구해주는 객체
* `for` 루프의 각 단계에서 지표를 업데이트 하면서 구현
    
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
        except StopIteration:
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
>>> [x ** 2 for x in range{10)]
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
흔히 사용하는 함수로 `enumerate()` 함수가 있다. 보통 리스트를 순회할 때 인덱스를 함께 이용하기 위해서 사용하며 실제 사용 예는 아래와 같다.

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
        current = self.current 
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

그러나 이 코드로 `enumerate()` 함수를 사용하도록 재 작성할 수는 없다. 

왜냐하면 일반 파이썬 의 `for` 루프를 사용하기 위한 인터페이스를 지원하지 않기 때문이다. 
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

사실 파이썬의 `for`문은 정말 특이한 형태이며, 다른 언어의 `for`문과는 꽤 다르다.

다른 언어를 먼저 배웠다면 아래와 같은 형식의 `for`문은 매우 익숙할 것이다.
	
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
즉 `i`라는 변수가 있고, `for` 루프가 한번 수행될 때마다 `i`변수가 `1`씩 더해가면서 인덱스를 지정하는 방식이다.
`i`가 기본형 변수(primitive variable)이기 때문에 일반적인 정수형 변수처럼 `i`를 조건에 따라 중간에서 수정하거나, 갱신하는 등의 작업이 가능하다.
	
하지만 파이썬은 다르다. 파이썬은 `range(len(nums))` 함수에서 `0 ~ len(nums)`범위의 리스트를 생성한 후 `i`를 해당 변수에 지정하여 순회하는 형식이다.
따라서 `range(len(nums))`는 `[0, 1, 2, 3, 4]`와 같은 반복 가능한 객체이며, 
`i`를 루프 안에서 수정한들, 다음 루프에는 다음 순회할 변수가 새롭게 지정되기 떄문에 이전 루프에서의 수정은 이후 루프에 영향을 주지 못한다.


따라서 `next()` 메서드는 파이썬의 특별한 형식에 알맞게 이터레이터를 다음으로 이동시키는 함수이다. 
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

`itertools` 모듈을 사용하여 이터러블을 조금 더 파이썬에 맞게 사용할 수 있다.
### 사용 예
다음과 같은 많은 경우에 `itertools` 모듈이 사용될 수 있을 것이다.

#### 1. 특정 조건에 맞는 행을 뽑아(`filter()`), 그 중 필요한 행만 뽑기(`islice()`)
만약 특정 기준을 넘은 값에 내해서만 연산을 하려면 어떻게 해야 할까? 가장 간단한 방법은 `while`문 안에 조건문을 추가하는 것이다.

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

이런 식으로 필터링을 해도 메모리의 손해는 없다. 왜냐하면 모든 것이 제네레이터이므로 게으르게 평가된다. 

즉 마치 전체에서 필터링한 값으로 연산을 한 것처럼 보이지만, 실제로는 하나씩 가져와서 모든 것을 메모리에 올릴 필요가 없는 것이다.

#### 2. 여러 번 반복하기

`process()`에서는 최소, 최대, 평균 값을 구하기 위해 3번의 반복이 필요하였다. 
`itertools.tee()` 메서드를 사용하면 3번의 반복 없이 분할된 이터러블을 사용해 필요한 연산을 할 수 있다.

```python
def process_purchases(purchases):
    min_, max_, avg = itertools.tee(purchases, 3) return min(min_), max(max_), median(avg)
```

#### 3. 중첩 루프
경우에 따라 1차원 이상을 반복해서 값을 찾아야 할 수 있다. 

간단한 방법으로는 중첩루프를 통해 해결할 수 있다. 

값을 찾으면 순환을 멈추고 `break` 키워드를 호출하게 해야 하며, 만약 두 단계 이상을 벗어나야 한다면 정상적으로 동작시키기 위해서 `flag` 변수를 통해서 동작시킬 수 있을 것이다.

다음은 위 상황에서 피해야할 나쁜 예이다.
```python
def search_nested_bad(array, desired_value): 
	coords = None
    for i, row in enumerate(array):
        for j , cell in enumerate(row) :
            if cell = desired_value: 
                coords = (i, j)
                break
	
        if coords is not None: 
            break

    if coords is None:
        raise ValueError(f"{desired_value} not found")
	
    logger.info("[%i, %i]에서 값 %r 찾음", *coords, desired_value) 
    return coords
```

다음은 지져분했던 종료 플래그 따윈 사용하지 않은 보다 간단하고 컴팩트한 예이다.

```python
def _iterate_array2d(array2d):
    for i, row in enumerate(array2d):
        for j, cell in enumerate(row): 
            yield (i, j), cell

def search_nested(array, desired_value):
    try:
        coord = next(
            coord
            for (coord, cell) in _iterate_array2d(array) 
	        if cell == desired_value
        )
    except Stopiteration:
        raise ValueError("{desired_value} not found")
	
    logger.info("[%i, %i]에서 값 %r 찾음", *coords, desired_value) 
    return coord
```

`_iterate_array2d()` 메서드에서 2차원이였던 배열을 제네레이터를 통해 1차원처럼 변환하며, 
덕분에 `search_nested()` 메서드에서는 단일행 `coord`만 검사하여 깔끔하게 코드를 작성할 수 있었다.
	
이를 이용하면 나중에 더 많은 차원의 배열을 사용하는 경우에도 클라이언트는 그것에 대해 알 필요가 없이 기존 코드를 그대로 사용하면 된다.
이를 통해 파이썬은 이터레이터 객체를 지원하므로 자동으로 투명해진다.

## 파이썬의 이터레이터 패턴

제네레이터는 이터러블 객체의 특별한 경우이지만 파이썬의 반복은 제네레이터 이상의 것으로 훌륭한 이터러블 객체를 만들게 되면 보다 효율적이고 컴팩트하고 가독성이 높은 코드를 작성할 수 있게 된다.

### Iterator VS Iterable

방금까지 이터러블과 이터레이터에 대한 것을 매우 많이 언급하였다.

잠깐 멈추어 생각해보자. 두 개념은 어디서 차이가 있는가?
	
|파이썬 개념|매직 메서드|비고|
|-------|------|-------|
|이터러블(Iterable)|`__iter__()`|이터레이터와 함께 반복 로직을 만든다. 이것을 구현한 객체는 `for ... in ...` 구문에서 사용할 수 있다|
|이터레이터(Iterator)|`__next__()`|한 번에 하나씩 값을 생산하는 로직을 정의한다. 더 이상 생산할 값 이 없을 경우는 `StopIteration` 예외를 발생시킨다. 내장 `next()` 함수를 사용해 하나씩 값을 읽어 올 수 있다|
	
#### 이터러블하지 않은 이터레이터
```python

class SequenceIterator:
    def __init__(self, start=0, step=1):
        self.current = start 
        self.step = step
	
    def __next__(self):
        value = self.current 
        self.current += self.step 
        return value
	
	
>>> si = SequenceIterator(1, 2) 
>>> next(si)
1
>>> next(si)
3
>>> next(si)
5
>>> for _ in SequenceIterator(): pass
...
Traceback (most recent call last):
    ...
TypeError: 'SequenceIterator' object is not iterable
```

> 시퀀스에서 하나씩 값을 가져올 수 있지만 반복할 수는 없다
	
파이썬이 `for` 루프를 만나면 객체가 `__iter__()`를 구현했는지 확인하고 였으면 그것을 사용한다. 그러나 없을 경우는 다른 대비 옵션을 가동한다.

객체가 시퀀스인 경우(_즉, `__getitem__()`과 `__len__()` 매직 메서드를 구현한 경우_)도 반복 가능하다. 
> 이 경우 인터프리터는 `IndexError` 예외가 발생할 때까지 순서대로 값을 제공한다.

```python
# generators_iteration_2.py

class MappedRange:
    """특정 숫자 범위에 대해 맵으로 변환"""

    def __init__(self, transformation, start, end): 
        self._transformation = transformation
        self._wrapped = range(start, end)
	
    def __getitem__(self, index):
        value = self._wrapped.__getitem__(index) 
        result = self._transformation(value) 
        logger.info("Index %d: %s", index, result) 
        return result
	
    def __len__(self):
        return len(self._wrapped)
	
>>> mr = MappedRange{abs, -10, 5) 
>>> mr[0]
Index 0: 10
10
>>> mr[-1]
Index -1: 4
4
>>> list(mr)
Index 0: 10
Index 1: 9
Index 2: 8
Index 3: 7
Index 4: 6
Index 5: 5
Index 6: 4
Index 7: 3
Index 8: 2
Index 9: 1
Index 10: 0
Index 11: 1
Index 12: 2
Index 13: 3
Index 14: 4
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0, 1, 2, 3, 4]
```

이러한 방법이 있다는 것을 알아두는 것은 좋지만 객체가 `__iter__()`를 구현하지 않았을 때 동작하는 대비책임에 주의하자. 

일반적으로는 `__iter__()`메서드를 구현하여 정식 이터러블 객체를 만드는 것이 권장된다.
	
# [Chapter. 7-2] 코루틴(coroutine)
~~사실상 이 챕터의 가장 어려운 부분이며, 새로운 장이라고 해도 마땅한 부분이다.~~

> _"코루틴(coroutine)은 cooperative routine를 의미하는데 서로 협력하는 루틴이라는 뜻입니다. 즉, 메인 루틴과 서브 루틴처럼 종속된 관계가 아니라 서로 대등한 관계이며 특정 시점에 상대방의 코드를 실행합니다."_

![image](https://user-images.githubusercontent.com/59782504/173235591-69746bf3-5080-4b76-a87b-d8f40b4cf9f2.png)

> _Reference: [https://dojang.io/mod/page/view.php?id=2418](https://dojang.io/mod/page/view.php?id=2418)_

제네레이터는 단순 반복 가능한 객체로 활용하는 것이 아니라 코루틴으로도 활용할 수 있다.

### 제네레이터 인터페이스의 메서드
#### `close()`
제네레이터에서 `GeneratorExit`예외를 발생시키는 함수, 따로 처리하지 않으면 제네레이터는 더 이상 값을 생성하지 않으며 반복이 중지된다. 
코루틴이 일종의 자원 관리를 하는 경우 이 예외를 통해서 코루틴이 보유한 모든 자원을 해재할 수 있다.

일반적으로 컨텍스트 관리자를 사용하거나 `finally` 블록에 코드를 배치하는 것과 비슷하지만 이 예외를 사용하면 보다 명확하게 처리할 수 있다.

```python
def stream_db_records(db_handler): 
    try:
        while True:
            yield db_handler.read_n_records(10)
    except GeneratorExit: 
        db_handler.close()
		
		
>>> streamer = stream_db_records(DBHandler("testdb"))
>>> next(streamer)
[(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), ...]
>>> next(streamer)
[(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), ...] 
>>> streamer.close()
INFO:...:closing connection to database 'testdb'
```

####  `throw(ex_type[, ex_value[, ex_traceback]])`
해당 메서드는 현제 제네레이터가 중단된 위치에서 예외를 던진다. 

제네레이터가 예외를 처리했으면 `except`절에 있는 코드가 호출되며, 예외를 처리하지 않았다면 호출자에게 에러가 전파된다.

```python
class CustomException(Exception): 
    pass
    
    def stream_data(db_handler):
        while True: 
            try:
                yield db_handler.read_n_records(10) 
            except CustomException as e:
                logger.warning( "처리 가능한 에러 %r, 계속 진행", e)
            except Exception as e:
                logger.error( "처리할 수 없는 에러 %r, 중단", e) 
                db_handler.close()
                break


>>> streamer = stream_data(DBHandler("testdb")) 
>>> next(streamer)
[(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), (4, 'row 4'), ...] 
>>> next(streamer)
I(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), (4, 'row 4'), ...] 
>>> streamer.throw(CustomException)
WARNING: 처리 가능한 에러 CustomException(), 계속 진행
[(0, 'row 0'), (1, 'row 1'), (2, 'row 2'), (3, 'row 3'), (4, 'row 4'), ...] 
>>> streamer.throw(RuntimeError)
ERROR: 처리할 수 없는 에러 RuntimeError(), 중단 INFO: 'testdb' 데이터베이스 연결 종료
Traceback (most recent call last):
...
StopIteration
```

도메인에서 처리하고 였는 `CustomException` 예외를 받은 경우 제네레이터는 계속 진행된다. 
그러나 그 외 예외는 `Exception`으로 넘어가서 데이터베이스 연결을 종료하고 반복도 종료하게 된다.

#### `send(value)`
현재 제네레이터의 주요 기능은 고정된 수의 레코드를 읽는 것이다.

이제 읽어올 개수를 파라미터로 받아서 호출하도록 수정해보자. 
안타깝게도 `next()` 함수는 이러한 옵션을 재공하지 않는다. 이럴 때 `send()` 메서드를 사용하면 된다.

```python
def stream_db_records(db_handler): 
    retrieved_data = None 
    previous_page_size = 10

    try:
        while True:
            page_size = (yield retrieved_data) or previous_page_size			
            retrieved_data = db_handler.read_n_records(page_size)
			
    except GeneratorExit:
        db_handler.close()
```

이제 `send()` 메서드를 통해 인자 값을 전달할 수 있다. 사실 이 메서드는 제네레이터와 코루틴을 구분하는 기준이 된다. 

`send()` 메서드를 사용했다는 것은 `yield` 키워드가 할당 구문의 오른쪽에 나오게 되고 인자 값을 받아서 다른 곳에 할당할 수 있음을 뜻한다.


이때 `yield`는 2가지의 역할을 수행한다.
1. `retrieved_data`를 호출자에게 보내고 그곳에 멈춘다. 호출자는 `next()` 메서드를 호출하여 다음 라운드가 되었을 때 값을 가져올 수 있다.
2. `retrieved_data`를 호출자에게서 `send()`메서드를 통해 할당받는다 그리고 해당 값을 `page_size`에 할당한다.

코루틴에 값을 전송하는 것은 `yield` 구문이 멈춘 상태에서만 가능하다.<br>
그렇게 되려면 일단 코루틴을 해당 상태까지 이동시켜야 하며, 해당 상태로 이동하기 위한 유일한 방법은 `next()`메소드를 사용하는 것이다.<br>
`send()` 메서드를 호출하여 값을 전달하기 위해서는 적어도 한번은 `next()` 메서드를 호출해야 한다.


▼ `next()`를 호출하지 않고 `send()` 메서드를 호출할 경우
```python
>>> c = coro()
>>> c.send(1)
Traceback (most recent call last):
...
TypeError: can't send non-None value to a just-started generator
```

`@prepare_coroutine` 데코레이터를 사용할 경우 `next()`를 초기에 호출해주지 않아도 코루틴을 생성하자마자 사용할 수 있도록 한다.

```python
@prepare_coroutine
def stream_db_records(db_handler):
    retrieved_data = None page_size = 10
    try:
        while True:
            page_size = (yield retrieved_data) or page_size 
            retrieved_data = db_handler.read_n_records(page_size)
    except GeneratorExit: 
        db_ handler.close()
		
		
		
>>> streamer = stream_db_records(DBHandler("testdb")) 
>>> len(streamer.send(5))
5
```

## 코루틴의 고오급 주제

코루틴은 사실 진보된 고오급 제네레이터라고 할 수 있다. 코루틴은 멋진 제네레이터이다. 

~~_(알고 있는가, 고급으로 가면 갈수록 잘 사용되지 않는다는 것을 크흠..)_~~

반복이란 앞에서 설명한 것처럼 `StopIteration` 예외가 발생할 때까지 `next()` 메서드를 계속해서 호출하는 메커니즘을 말한다.
코루틴은 기술적으로는 제네레이터이지만 반복을 염두예 두고 만든 것이 아니라 나중에 코드가 실행될 때까지 코드의 실행을 멈추는 것을 목표로한다.

우리는 반복에 대해서 코루틴을 알아보았지만, 코루틴은 일반적으로 반복보다는 상태를 중단하는데 초점을 맞추고 있다.
> _(오히려 반복을 목적으로 코루틴을 만드는 것이 이상한 경우인 것이다.)_

코루틴을 사용하여 정보를 처리하고 실행을 일시 중단하는 경우, 경량 스레드(다른 플랫폼 에서는 그린 스레드)라고 생각하는 것이 좋다.
그렇다면 다른 정규 함수를 호출하는 것처럼 값을 반환하는 것이 이해가 된다.

그러나 제네레이터가 일반 함수는 아니므로 `value = generator()`라고 하는 것은 제네레이터 객체를 만드는 것 외에는 아무것도 하지 않을 것이다.
_(`yield`로 만들어진 이터러블 객체)_

이런 일을 간편하기 위해서 [PEP-380](https://peps.python.org/pep-0380/)에서는 `yield from` 이라는 새로운 생성자 구문을 도입하여 문제를 해결했다.

### 코루틴에서 값 반환하기
제네레이터에서 값을 반환(`return`)하면 반복이 즉시 중단된다. 

즉 더 이상 반복을 할 수 없다. 

코루틴은 본래의 의미 체계를 유지하기 위해 `StopIteration` 예외가 발생해도 예외 객체 내에 반환 값이 저장되어 있다. 예외에서 해당 값을 처리하는 것은 호출자의 책임이다.
```python
>>> def generator( ): 
...     yield 1
...     yield 2
...     return 3
...
>>> value = generator() 
>>> next(value)
1
>>> next(value)
2
>>> try: 
...     next(value)
... except Stoplteration as e:
...     print(">>>>>> returned value ", e.value)
...
>>>>>> returned value 3
```

예외 객체를 통해 값을 반환하는 점은 다른 활용성을 열어준 측면에서 흥미로운 기능이다. 

그러나 값을 반환하는 기능을 언어 자체에서 지원해주지 않았다면, 매번 이렇게 구현하는 것은 개발자로 하여금 힘들게 만들었을 것이다. 

### 작은 코루틴에 위임하기 - `yield from`

이제 `value = yield from generator()`와 같이 작성하면 값을 반환받는 것이 가능해졌다.

#### 제네레이터 체인

제네레이터 체인은 여러 제네레이터를 하나의 제네레이터로 합치는 기능을 하는데 중첩된 `for` 루프를 사용해 하니씩 모으는 대신에 서브 제네레이터의 값을 하나로 수집할 수 있게 해준다.
대표적으로 해당 함수는 라이브러리 `itertools`에 `itertools.chain()`이라는 메서드로 구현이 가능하나, `yield from`을 사용함으로써 이와 비슷한 함수를 구현해보자

```python
def chain(*iterables): 
    for it in iterables:
        for value in it: 
            yield value
```

여러 이터러블을 받아서 모두 이동한다. 모두 이터러블이므로 `for ... in` 구문을 지원하므로 개 별 값을 구하려면 중첩 루프를 사용하면 된다.

`yield from` 구문을 사용하면 서브 제네레이터에서 직접 값을 생산할 수 였으므로 중첩 루프를 피할 수 있다. 
```python
def chain(*iterables): 
    for it in iterables:
        yield from it
```

결과는 모두 동일하다.
```python
>>> list(chain("hello", ["world"], ("tuple", " of ", "values."))) 
['h', 'e', 'l', 'l', 'o', 'world', 'tuple', ' of ', 'values.']
```
`yield from` 구문은 **어떤 이터러블에 대해서도 동작**하며 이것을 사용하면 마치 **최상위 제네레이터가 직접 값을 `yield`한 것과 같은 효과**를 보인다.

#### 제네레이터 표현식에서 `yield from` 활용하기
`yield from`은 어떤 형태의 이터러블에서도 동작하므로 제네레이터 표현식도 마찬가지이다.
`yield from`구문을 활용해 입력된 파라미터의 모든 제곱지수를 만드는 제네레이터를 만들어보자.

```python
def all_powers(n, pow):
    yield from (n ** i for i in range(pow + 1))
```

기존의 서브 제네레이터에서 `for`문을 사용해 값을 생산하는 대신 한 줄로 직접 값을 생산할 수 있다.

### 서브 제네레이터에서 반환된 값 구하기

다음 예제는 수열을 생산하는 두개의 중첩된 제네레이터를 호출한다. 각각의 제네레이터는 값을 반환하는데 최상위 제네레이터는 쉽게 반환 값을 확인할 수 있다. 

바로 `yield from` 구문을 사용했기 때문이다.

```python
def sequence(name, start, end):
    logger.info("%s 제네레이터 %i에서 시작", name, start) 
    yield from range(start, end)
    logger.info("%s 제네레이터 %i에서 종료", name, end) 
    return end
    
def main():
    step1 = yield from sequence("first", 0, 5)
    step2 = yield from sequence("second", step1, 10) 
    return step1 + step2
    
>>> g = main() 
>>> next(g)
INFO: generators_yield from_2: first 제네레이터 0에서 시작
0
>>> next(g) 
1
>>> next(g) 
2
>>> next(g) 
3
»> next(g)
4
>>> next(g)
INFO:generators_yieldfrom:first 재레이터 5에서 종료 
INFO:generators_yieldfrom_2:second 제네레이터 5에서 시작
5
>>> next(g) 
6
>>> next(g) 
7
>>> next(g) 
8
>>> next(g) 
9
>>> next(g)
INFO:generators_yieldfrom_2:second 제네레이터 10에서 종료 
Traceback (most recent call last):
    File "<stdin>", line 1, in <module> 
StopIteration : 15
```

`main` 제네레이터의 첫째 행은 내부 제네레이터로 위임하여 생산된 값을 가져온다.<br>
두 번째 제네레이터 역시 종료 시 값(`10`)을 반환하고 그러면 `main` 제네레이터는 이 두 결과의 합(`5 + 10 = 15`)을 반환한다.<br>
이 값은 `StopIteration`에 포함된 값이다.
> `yield from`을 사용하면 코루틴의 종료 시 최종 반환 값을 가져올 수 있다.

### 서브 제네레이터와 데이터 송수신하기 

> _"이제 코루틴의 진정한 강력함을 느낄 수 있게 해주는 멋진 기능을 살펴보자." - p.232_

위에서 살펴본 것처럼 제네레이터는 코루틴처럼 동작할 수 있다.<br>
값을 전송하고 예외를 던지면 코루틴 역할을 하는 해당 제네레이터는 값을 받아서 처리하거나 반드시 예외를 처리해야 한다.
> 이는 서브 제네레이터에 위임한 코루틴에 대해서도 마찬가지이다.

```python
def main():
    step1 = yield from sequence("first", 0, 5)
    step2 = yield from sequence("second", step1, 10) 
    return step1 + step2

def sequence(name, start, end):
    value = start
    logger.info("%s 제네레이터 %i에서 시작", name, value) 
    
    while value < end:
        try:
            received = yield value
            logger.info("%s 제네레이터 %r 값 수신", name, received) 
            value += 1
            
        except CustomException as e:
            logger.info("%s 제네레어터 %s 에러 처리", name, e)
            received = yield "OK" 
    
    return end



>>> g = main() 
>>> next(g)
INFO: first 제네레이터 0에서 시작
0
>>> next(g)
INFO: first 제네레이터 None 값 수신
1
>>> g.send( "첫 번째 제네레이터를 위한 인자 값")
INFO: first 제네레이터 '첫 번째 제네레이터를 위한 인자 값' 값 수신 
2
>>> g.throw(CustomException( "처리 가능한 예외 던지기"))
INFO: first 제네레이터 처리 가능한 예외 던지기 에러 처리
'OK'
>>> next(g)
2
>>> next(g)
first 제네레이터 None 값 수신
3
>>> next(g)
first 제네레이터 None 값 수신
4
>>> next(g)
first 제네레이터 None 값 수신 # first가 종료를 함과 동시에 값을 반환하여
second 제네레이터 5에서 시작 # second 코루틴에 전달되었다.
5
>>> g.throw(CustomException("두 번째 제네레이터를 향한 예외던지기"))
INFO: second 제네레이터 두 번째 제네레이터를 향한 예외 던지기 에러 처리
'OK'
```

위의 예제에서는 실제로 `sequence()` 서브 제네레이터에는 값을 보내지 않고, 오직 `main` 제네레이터를 통해 값을 보냈다. 

그러나 실제 값을 받는 쪽은 내부 제네레이터이다.<br>
`yield from`을 통해 값이 내부 제네레이터인 `sequence`에 전달된 것이다.

첫 번째 코루틴이 끝나면 `step1` 변수에 값을 반환하고, 그 값을 두 번째 코루틴에 입력으로 전달한다.<br>
두 번째 코루틴도 첫 번째 코루틴과 동일하게 `send()`와 `throw()`에 대해 동일한 작업을 한다.

독정 단계에서 `send()`를 호출했을 때 생성 하는 값은 사실 현재 `main` 제네레이터가 멈춰 있던 서브 코루틴에서 생성한 값이다.

## 비동기 프로그래밍

지금까지 살펴본 것들을 활용해 파이썬에서 비동기 프로그램을 만들 수 있다.
> 즉 여러 코루틴 이 특정 순서로 동작하도록 스케줄링을 할 수 있으며, 일시 정지된 `yield from` 코루틴과 통신 할 수 있다

이러한 기능을 통해 논블로킹(non-blocking) 방식으로 병렬 I/0 작업을 할 수 있으며, 이 때 필요한 것은 보통 서드파티 라이브러리에서 구현한 저수준의 제네레이터이다.<br>
해당 라이브러리들은 코루틴이 일시 중단된 동안 실제 I/0 처리를 한다.

**코루틴이 정지된 동안 프로그램은 다른 작업을 할 수 있어 효율적**이다. 

프로그램은 `yield from` 문장에 의해 중단되기도 하고 생산된 값을 받기도 하며 제어권을 주고 받는다.

코루틴과 제네레이터가 기술적으로는 동일하다는 점에서 혼란스러울 때가 있다.<br>
문법적으로 (또는 기술적으로) 이들은 동일하지만 의미적으로는 다르다.

### 주의점

파이썬의 동적 특성으로 인해 이러한 객체를 혼합해서 사용하다가 개발 마지막 단계에서 런타임 오류가 발생하기도 한다.

앞서 `yield from` 구문을 사용한 간단한 예제 로 제시했던 `chain` 함수를 기억해보자. 
이 객체들은 사실 코루틴이 아니었지만 문제가 없이 잘 동작했다. 
그 다음 `yield from`을 사용해 여러 코루틴에게 값을 보내고 예외를 던지고 값을 가져오는 것도 살펴보았다. 

이 둘은 명백히 다른 성격의 사용 예제였다. 만약 다음과 같은 코드가 있다고 생각해보자

```python
result = yield from iterable_or_awaitable()
```
`iterable_or_awaitable`이 반환하는 것이 명확하지 않다. 
단순히 문자열과 같은 `iterable`이어도 문법상 문제가 없다. 

또는 실제 코루틴일 수도 있다. 이러한 유형의 실수는 나중에 큰 비용을 초래하기 마련이다.

### 코루틴을 위해서 추가된 구문
실제로 파이썬 3.5 이전에 코루틴은 `@coroutine` 데코레이터가 적용된 제네레이터일 뿐이었으며 `yield from` 구문을 사용해 호출했다.

그러나 위에서 알아보았듯, `yield from` 구문이 코루틴에서도, 이터러블에서도 사용되는 것이 문제가 될 수 있다.

이를 위해 새로운 구문으로 `await`과 `async def`가 추가되었다. 

#### `await`
`await`은 `yield from`을 대신하기 위한 용도로 `awaitable`객체에 한해서만 동작한다. 
코루틴은 `awaitable`로, `awaitable` 인터페이스에 따르지 않는 객체에 `await`을 호출하면 예외가 발생한다. 

#### `async def`
`async def`는 `@coroutine` 데코레이터를 대신하여 코루틴을 정의하는 새로운 방식이다. 

이것은 호출 시 실제로 객체를 만들어 코루틴 인스턴스를 반환한다.


파이썬 비동기 프로그래밍의 모든 세부사항을 살펴보지는 않았지만 새로운 구문과 타입에도 불구하고 그 근본 원리는 앞서 살펴본 것과 동일하다고 말할 수 있다.
