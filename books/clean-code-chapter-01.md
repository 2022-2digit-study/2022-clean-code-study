# [Chapter .1] 소개, 코드 포매팅과 도구

---

## 먼저 프로그래밍 언어부터.

먼저 프로그래밍 언어에 대해 정의를 내려봅시다.<br>
<br>

### 아래에는 두 가지 명제가 있습니다.

무엇이 맞는 명제일까요?<br>
<br>
`1. 프로그래밍 언어는 인간의 아이디어를 컴퓨터에게 전달하는 것이다.`<br>
`2. 프로그래밍 언어는 인간의 아이디어를 다른 개발자에게 전달하는 것이다`<br>
<br>
**프로그래밍 언어를 컴파일하고 실행시키는 주체는 컴퓨터**이나, 프로그램은 단 한번 실행하고 멈추는 것이 아니라, **지속적으로 유지보수하고 발전해나가는 존재**입니다.<br>
따라서 프로그래밍 언어의 본질은 2번에 있습니다.
> _"수십 년 동안 프로그래밍 언어라는 것은 인간의 아이디어를 컴퓨터에 전달하기 위해 사용하는 언어라고 생각해왔다. 그러나 그건 틀린 생각이다. 이러한 생각은 진실이 아니라 진실의 일부 이다. **프로그래밍 언어의
진정한 의미는 아이디어를 다른 개발자에게 전달하는 것**이다." (19p)_

---

## 그럼 클린코드는 뭔데?

클린코드를 정의하는 여러 문서들이 있습니다. 특히 여러 [블로그](https://hypers84.tistory.com/2460)에도 클린코드의 5가지 원칙을 말하면서, 클린코드에 대한 설명을 자주 찾아볼 수
있습니다.<br>
> 그러나 저자는 `"클린코드는 이런거야!"` 하고 정의를 내리기 보다, 책을 통해 스스로 **좋은 코드**와 **나쁜 코드**를 구분하는 것을 권장합니다.

### + 클린코드 X 코드 포매팅

가끔 코드 포매팅을 잘하는 것이 곧 클린코드라고 말하는 것을 볼 수 있습니다.<br>
<br>
그러나 [PEP-8](https://peps.python.org/pep-0008/)과 같은 지침에 따라 **코드를 포매팅 하고 구조화하는 것은 클린코드의 최소요건**이며 코드 포매팅이 클린코드라는 것은 틀린 명제입니다.

<details>
    <summary style="color: aquamarine; font-weight: bold;">PEP-8?</summary>
    <div>
        Python Enhancement Proposal의 약어, 파이썬을 위한 제안서를 의미한다. 이중 8번은 코딩 컨벤션에 대한 내용을 다룬다.
    </div>
</details>

### 일관성

분명 PEP-8과 같은 코딩 가이드를 준수하는 것은 소프트웨어의 일관성을 유지하는 것에 도움을 줍니다.<br>
<br>
특히 일관성 있는 코드는 오류를 감지하기 쉽게하며,<br>
신속하게 패턴을 파악하여 개발자의 이해를 높일 수 있습니다.

#### Q. 어떤게 더 읽기 쉽나요?

```css
@charset "UTF-8";/*!
 * Bootstrap v5.2.0-beta1 (https://getbootstrap.com/)
 * Copyright 2011-2022 The Bootstrap Authors
 * Copyright 2011-2022 Twitter, Inc.
 * Licensed under MIT (https://github.com/twbs/bootstrap/blob/main/LICENSE)
 */:root{--bs-blue:#0d6efd;--bs-indigo:#6610f2;--bs-purple:#6f42c1;--bs-pink:#d63384;--bs-red:#dc3545;--bs-orange:#fd7e14;--bs-yellow:#ffc107;
 --bs-green:#198754;--bs-teal:#20c997;--bs-cyan:#0dcaf0;--bs-black:#000;--bs-white:#fff;--bs-gray:#6c757d;--bs-gray-dark:#343a40;--bs-gray-100:#f8f9fa;
 --bs-gray-200:#e9ecef;--bs-gray-300:#dee2e6;--bs-gray-400:#ced4da;--bs-gray-500:#adb5bd;--bs-gray-600:#6c757d;--bs-gray-700:#495057;
 --bs-gray-800:#343a40;--bs-gray-900:#212529;--bs-primary:#0d6efd;--bs-secondary:#6c757d;--bs-success:#198754;--bs-info:#0dcaf0;--bs-warning:#ffc107;
 --bs-danger:#dc3545;--bs-light:#f8f9fa;--bs-dark:#212529;--bs-primary-rgb:13,110,253;--bs-secondary-rgb:108,117,125;--bs-success-rgb:25,135,84;
 --bs-info-rgb:13,202,240;--bs-warning-rgb:255,193,7;--bs-danger-rgb:220,53,69;--bs-light-rgb:248,249,250;--bs-dark-rgb:33,37,41;
 --bs-white-rgb:255,255,255;--bs-black-rgb:0,0,0;--bs-body-color-rgb:33,37,41;--bs-body-bg-rgb:255,255,255;--bs-font-sans-serif:system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue","Noto Sans","Liberation Sans",Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol","Noto Color Emoji";
 --bs-font-monospace:SFMono-Regular,Menlo,Monaco,Consolas,"Liberation Mono","Courier New",monospace;--bs-gradient:linear-gradient(180deg, rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0));--bs-body-font-family:var(--bs-font-sans-serif);--bs-body-font-size:1rem;--bs-body-font-weight:400;--bs-body-line-height:1.5;--bs-body-color:#212529;--bs-body-bg:#fff;--bs-border-width:1px;--bs-border-style:solid;--bs-border-color:#dee2e6;
 --bs-border-color-translucent:rgba(0, 0, 0, 0.175);--bs-border-radius:0.375rem;--bs-border-radius-sm:0.25rem;--bs-border-radius-lg:0.5rem;
 --bs-border-radius-xl:1rem;--bs-border-radius-2xl:2rem;--bs-border-radius-pill:50rem;--bs-heading-color: ;
 --bs-link-color:#0d6efd;--bs-link-hover-color:#0a58ca;--bs-code-color:#d63384;--bs-highlight-bg:#fff3cd}
```

▲ 난독화가 적용된 구조화 되지 않은 코드.

```css
@charset "UTF-8";
/*!
 * Bootstrap v5.2.0-beta1 (https://getbootstrap.com/)
 * Copyright 2011-2022 The Bootstrap Authors
 * Copyright 2011-2022 Twitter, Inc.
 * Licensed under MIT (https://github.com/twbs/bootstrap/blob/main/LICENSE)
 */
:root {
    --bs-blue: #0d6efd;
    --bs-indigo: #6610f2;
    --bs-purple: #6f42c1;
    --bs-pink: #d63384;
    --bs-red: #dc3545;
    --bs-orange: #fd7e14;
    --bs-yellow: #ffc107;
    --bs-green: #198754;
    --bs-teal: #20c997;
    --bs-cyan: #0dcaf0;
    --bs-black: #000;
    --bs-white: #fff;
    --bs-gray: #6c757d;
    --bs-gray-dark: #343a40;
    --bs-gray-100: #f8f9fa;
    --bs-gray-200: #e9ecef;
    --bs-gray-300: #dee2e6;
    --bs-gray-400: #ced4da;
    --bs-gray-500: #adb5bd;
    --bs-gray-600: #6c757d;
    --bs-gray-700: #495057;
    --bs-gray-800: #343a40;
    --bs-gray-900: #212529;
    --bs-primary: #0d6efd;
    --bs-secondary: #6c757d;
    --bs-success: #198754;
    --bs-info: #0dcaf0;
    --bs-warning: #ffc107;
    --bs-danger: #dc3545;
    --bs-light: #f8f9fa;
    --bs-dark: #212529;
    --bs-primary-rgb: 13, 110, 253;
    --bs-secondary-rgb: 108, 117, 125;
    --bs-success-rgb: 25, 135, 84;
    --bs-info-rgb: 13, 202, 240;
    --bs-warning-rgb: 255, 193, 7;
    --bs-danger-rgb: 220, 53, 69;
    --bs-light-rgb: 248, 249, 250;
    --bs-dark-rgb: 33, 37, 41;
    --bs-white-rgb: 255, 255, 255;
    --bs-black-rgb: 0, 0, 0;
    --bs-body-color-rgb: 33, 37, 41;
    --bs-body-bg-rgb: 255, 255, 255;
    --bs-font-sans-serif: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", "Noto Sans", "Liberation Sans", Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
    --bs-font-monospace: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
    --bs-gradient: linear-gradient(180deg, rgba(255, 255, 255, 0.15), rgba(255, 255, 255, 0));
    --bs-body-font-family: var(--bs-font-sans-serif);
    --bs-body-font-size: 1rem;
    --bs-body-font-weight: 400;
    --bs-body-line-height: 1.5;
    --bs-body-color: #212529;
    --bs-body-bg: #fff;
    --bs-border-width: 1px;
    --bs-border-style: solid;
    --bs-border-color: #dee2e6;
    --bs-border-color-translucent: rgba(0, 0, 0, 0.175);
    --bs-border-radius: 0.375rem;
    --bs-border-radius-sm: 0.25rem;
    --bs-border-radius-lg: 0.5rem;
    --bs-border-radius-xl: 1rem;
    --bs-border-radius-2xl: 2rem;
    --bs-border-radius-pill: 50rem;
    --bs-heading-color: ;
    --bs-link-color: #0d6efd;
    --bs-link-hover-color: #0a58ca;
    --bs-code-color: #d63384;
    --bs-highlight-bg: #fff3cd
}
```

▲ 정리 도구를 통해 포매팅된 코드

<br>
<br>
그러나 가이드를 지키지 않아도 코드는 얼마든지 깔끔하게 적을 수 있습니다.

```python
# PEP-8 가이드에 어긋나는 변수 선언: "변수 선언 시, 할당 연산자 앞/뒤에는 공백문자가 하나씩만 포함되어야 한다."
my_first_name = "Giraffe"
my_car        = "Ferrari"
```

중요한 것은 이렇게 가이드를 지키지 않고 열을 옮겼을 뿐인데도 얼마든지 깔끔하게 코드를 적을 수 있다는 것이 아닙니다. 오히려 이런 포매팅을 하는 자동화 도구를 만든다면 이런 가이드를 준수하도록 만드는 것이 매우 어렵겠죠.<br>
<br>
따라서 코드 포매팅은 코드를 깔끔하게 보기 위한 최소요건이지 이 자체가 클린코드라는 명제는 따라서 **틀린 명제**가 됩니다.

--- 
## 클린코드가 왜 중요해?

### Q. 어떤 도로를 달리고 싶으신가요?

**1. 균열이 가있는 도로**

![img_1](https://t1.daumcdn.net/cfile/blog/2770C336566A6D5D04)
<br>
**2. 뻥 뚫린 고속도로**

![img_2](https://img1.daumcdn.net/thumb/R658x0.q70/?fname=https://t1.daumcdn.net/news/202004/13/yonhap/20200413115201692ourh.jpg)

### 이런 도로의 비교는 **소프트웨어 개발**과 꽤 비슷합니다.

`1번의 경우`는 오래 방치되어 곳곳에 균열이 생겼습니다. 운전자는 균열을 피해서 돌아가야 하기 때문에 시간이 지체될 것이고, 예정한 시간보다 늦어질 수 있겠죠. <br>
<br>
`2번의 경우`는 그냥 도로를 달리면 됩니다. 이렇게 관리가 잘 된 도로라면 오히려 예정된 시간보다 빨리 도착할 수 있겠죠?<br>
<br>
이처럼 _클린 코드를 쓴다._ 라는 것은 다음과 같은 기대효과를 가질 수 있습니다.

<details>
    <summary style="font-weight: bold;">유지 보수 향상</summary>
    <div></div>
</details>
<details>
  <summary style="color: aquamarine;font-weight: bold">기술부채 감소</summary>
  <div> 
    <p> 기술 부채란 나쁜 결정이나 적당한 타협의 결과로 생긴 소프트웨어적 결함을 말한다.</p>
  </div>
</details>
<details>
    <summary style="color: aquamarine; font-weight: bold;">애자일 개발을 통한 효과적인 작업 진행</summary>
    <div>소프트웨어 개발 방법에 있어서 아무런 계획이 없는 개발 방법과 계획이 지나치게 많은 개발 방법들 사이에서 타협점을 찾고자 하는 방법론</div>
</details>

---

## 설명, 어떻게 해야 하는가

굳이 설명을 하지 않아도 되는 코드가 좋은 코드겠지만, 자신이 썼던 코드를 이해할 때나, 다른 사람이 썼던 코드를 이해해야할 때 주석이 있으면 조금 더 편리한 것이 사실입니다.<br>
<br>
**그럼, 코드를 설명하는 방법은 어떤 것이 있을까요?**

---
### 1. 주석: Comment

> _"주석은 코드로 아이디어를 제대로 표현하지 못했음을 나타내는 것이다."_ (p.24)


### 주석이 좋지 않은 이유
1. 주석은 초보자로 하여금 읽기 어렵게 한다 (코드에 포함되어 있기 때문)
2. 개발자는 주석을 업데이트 하는 것을 깜빡하는 경우가 많으므로, 현재와 일치하지 않을 경우 다른 위험을 초래할 수 있다.

### 주석을 허용하는 경우
* 외부 라이브러리에 오류가 있을 경우

---

### 2. 함수의 이름과 파라미터의 이름
함수의 이름과 파라미터의 이름을 충분히 설명적이라면 매우 좋습니다. 하지만 이상과 현실은 다를 수 있죠.<br>
<br>
예)
```python
# 이 개발자는 길이 있으면 집에 가고, 길이 없으면 근처 찜질방으로 가는 함수를 작성하고 싶다.
def if_way_is_exist_go_home_or_go_near_sauna():
    ...
    The Logic
    ...
```

---

### 3. Docstring (👍)

`docstring`은 각 함수나 클래스에 추가할 수 있는 설명 **문서**입니다.

<img width="662" alt="image" src="https://user-images.githubusercontent.com/59782504/167643208-fff5c257-6e6f-45d3-ace9-b969bb6cef6c.png">

▲ docstring 예

### 선언
`docstring`은 간편하게 함수, 모듈, 클래스의 선언부 아래에 멀티라인 주석(`""" """`)을 붙임으로써 추가할 수 있습니다. (함수, 클래스의 경우 들여쓰기는 필수입니다.)
1. 함수
```python
def some_func():
    """Hello Function Docstring"""
```
2. 클래스
```python
class SomeClass():
    """Hello Class Docstring"""
```
3. 모듈
```python
# some_module.py
"""Hello Module Docstring"""
```

### 호출
파이썬을 실행을 통해 `docstring`에 적용되어 있는 값을 알고 싶다면 `__doc__`를 통해 호출할 수 있습니다.

`some_module.__doc__`<br>
`some_function.__doc__`<br>
`SomeClass.__doc__`

### Docstring이 왜 좋아?

사용법은 충분히 알아보았으니 왜 `docstring`을 권장하는지 알아보겠습니다.<br>
<br>
가장 큰 이유는 **파이썬이 동적 타이핑을 하기 때문입니다.** 파이썬은 파라미터 타입이나 반환 타입 체크를 강요하지 않습니다. 다른 언어를 배워본 적이 있다면, **파라미터나, 리턴 타입을 통해서 어떤 데이터가 들어오고 나가는 지 추론할 수 있다**는 것을 알 수 있습니다.

> 하지만 우리의 파이썬은 그렇지 않죠.

단순히 Python 코드와 Java 코드를 비교해보겠습니다.
```java
String getUserName(User user){
    
    ...
    
    return userName
}
```

```python
def get_user_name(user):
    
    ...
    
    return user_name
```

위의 자바 코드에서는 유저 객체를 매개변수로 받아 문자열 데이터 타입인 `String`으로 반환하는 것을 직관적으로 알 수 있습니다.<br>
<br>
하지만 아래 파이썬 코드는 조금 다릅니다. `get` 이라는 키워드를 보아 뭔가 유저의 이름을 가져오는 것을 알 수 있습니다.<br>
하지만 그 유저의 이름이 어떤 유저의 이름만을 위해 만들어 진 객체인지, 아니면 단순 문자열인지는 **알 수 없습니다.**<br> 
> 일례로 사실 아래 코드의 어떤 모듈에는 이런 객체들이 선언되어있을 지도 모릅니다.<br>
```python
class UserName:
    def __init__(self, last_name, first_name):
        self.last_name = last_name
        self.first_name = first_name
```
> 이런 객체를 반환하도록이요.

<br>
물론, 함수명이나 파라미터 명에서 충분히 되어 있다면 이를 보완할 수 있을 것입니다. 그게 아니더라도 이는 코드를 보면 쉽게 알 수 있을 수도 있습니다. <br>
다만 게으른 개발자에게 있어 이는 너무나도 고독한 처사입니다. 주석을 남겼더라면, 더 좋은 방법일 수 있겠죠.<br>
하지만 모듈을 하나하나 찾아서 들어가서 주석을 찾아보는 것은 여전히 번거롭습니다. <br>
<br>
이런 번거로운 개발자에게는 그것보다는 파이썬의 대화형 인터프리터를 통해 바로 실행하여 쉽게 알 수 있는, 아래와 같은 `docstring` 이 답이 될지도 모르겠습니다.

```python
def get_user_name(user):
    """
        Description: 
            유저의 객체를 통해 유저의 이름을 알아내는 함수 입니다.
        Param:
            some_module.user.User 클래스의 객체를 파라미터로 받습니다.
        Return:
            some_module.user_name.UserName 객체를 반환합니다.     
    """
    
    ...
    
    return user_name
```

```
    #Python Interpreter
    $ get_user_name??
    -----------------------------------------------------
    # Result
    -----------------------------------------------------
    Description: 
        유저의 객체를 통해 유저의 이름을 알아내는 함수 입니다.
    Param:
        some_module.user.User 클래스의 객체를 파라미터로 받습니다.
    Return:
        some_module.user_name.UserName 객체를 반환합니다.    
    -----------------------------------------------------
```

### Docstring은 완벽한 도구야?

아닙니다. 위에 제가 적은 코드 예시와 가이드들을 살펴보아도, `docstring`이 효과적으로 동작하기 위해서는 보다 상세한 설명을 기입해야 합니다.<br>
<br>
다만 그러면 그럴 수록, 필연적으로 코드보다 `docstring`이 크기가 커지고 맙니다.<br>
<br>
어떤 좋은 가이드를 봐도, 어떤 블로그를 보아도 이상적인 설명의 해답은 **설명 없이 코드만 보아도 직관적으로 이해가 되는 코드**입니다. ~~(물론 천국에 있는 코드겠지요)~~


### + 클린코드에서 권장하는 Docstring 예시)

<img width="344" alt="image" src="https://user-images.githubusercontent.com/59782504/167872580-f54625c3-ced7-4550-8652-da5bb9acdcea.png">

Docstring에 대해 간략하게 알아보지만 여러 방법으로, 여러 개발사에서 docstring가이드에 대한 부분을 명시해 놓았기 때문에, 이는 개발자가, 프로젝트 팀원끼리 자신의 프로젝트에 맞는 docstring 가이드를 선택하는 것이 가장 좋은 방법인 것 같습니다.

[여러가지 Docstring 가이드 보러가기](https://www.datacamp.com/tutorial/docstrings-python)

---

### 4. 어노테이션(👍): Annotation

사실 자바라는 언어를 접했다면 어 익숙하네? 라고 생각할 수 있습니다.<br>
하지만 자바에서의 어노테이션과 파이썬의 어노테이션은 많이 다른 듯 합니다.

### 어노테이션이 뭐야?

파이썬의 어노테이션은 확실히 설명을 위해서 만들어져 있습니다. 바로 우리가 원하던 **"타입"** 에요 <br>
**어노테이션은 힌트를 활성화시켜주는 역할을 수행**합니다.<br>
<br>
직관적으로 알아보기 위해 바로 코드를 보도록 하죠

```python
def get_user_name(user: User) -> UserName:
    """
        Description: 
            유저의 객체를 통해 유저의 이름을 알아내는 함수 입니다.
        Param:
            some_module.user.User 클래스의 객체를 파라미터로 받습니다.
        Return:
            some_module.user_name.UserName 객체를 반환합니다.     
    """
    
    ...
    
    return user_name
```

아까 작성한 코드에 타입 힌트(Type Hint)를 명시하였습니다. 이로써 아까 적어둔 Java 코드와 비슷해졌죠? 이처럼 어노테이션을 명시한다면 어떤 타입을 파라미터로 받는지, 어떤 타입을 리턴하는지 쉽게 알 수 있습니다. 개발자가 위의 코드를 본다면 아! `User` 인스턴스를 파라미터로 받아서 `UserName` 인스턴스로 반환한다는 것을 알 수 있을 것입니다.<br>
<br>

### 강제...로...?

이렇게 말했으니 이렇게 질문할 수 있습니다. _"그럼 타입힌트를 명시하면 실행시키면 타입과 다를 때 에러가 나는거야?"_ <br>
<br>
답을 말하자면 **아닙니다.**

`PEP484`에서는 이에 대한 답을 들을 수 있습니다.

> _"파이썬은 여전히 동적인 타입의 언어로 남을 것이다. 타입 힌트를 필수로 하자거나 심지어 관습으로 하자는 것은 전혀 아니다."_

### 확인

어노테이션은 인터프리터나 코드에서 타입 힌트를 명시한 함수에 `some_func.__annotations__`을 통해 확인할 수 있습니다.

```
$ get_user_name.__annotations__

>> {'user': some_module.user.User, 'return': some_module.user_name.UserName}
```

### 활용

이렇게 쉽게 어노테이션으로 타입과 클래스 위치까지 확인할 수 있다면, `docstring`의 많은 부분을 대체할 수 있을 것입니다.
```
"""
    Description: 
        유저의 객체를 통해 유저의 이름을 알아내는 함수 입니다.
    Param:
        some_module.user.User 클래스의 객체를 파라미터로 받습니다.
    Return:
        some_module.user_name.UserName 객체를 반환합니다.     
"""
```
이런 단순 위치 설명형 `docstring`이라면
```python
def get_user_name(user: User) -> UserName:
    """유저의 객체를 통해 유저의 이름을 알아내는 함수 입니다."""
    ...
    return user_name

```
이렇게 바꿀 수 있겠죠

### 더 알아두면 좋은 것.

많은 `Lint`들을 활용하면 타입 힌트를 명시할 때 타입을 검사하고 타입 힌팅에 명시한 클래스와 호환되지 않는 타입의 경우 경고를 통해 개발자에게 지침을 줄 수도 있습니다.<br>
<br>

<img width="546" alt="image" src="https://user-images.githubusercontent.com/59782504/167868440-89e3fa77-8f2e-48cb-bf5a-8266b0546bb8.png">

> 많이 사용하는 JetBrains계열의 IDE PyCharm에서는 명시한 타입 힌트를 어기는 데이터 타입을 발견했을 때 다음처럼 노란줄을 통해 알려주기도 합니다.

<br><br>

**+ 타입힌트에 명시한 파라미터를 이용할 때 해당 객체의 메소드를 사용할 수 있습니다.**

<img width="525" alt="image" src="https://user-images.githubusercontent.com/59782504/167868918-637ee268-0791-4821-ac89-c379ec453e2a.png">

> ▲ 타입힌트가 없을 때: 무엇을 추천해 줄지 IDE가 알 수 없다.

<br><br>

<img width="569" alt="image" src="https://user-images.githubusercontent.com/59782504/167868766-0af8318f-b27f-4476-9b40-dba59ee7f8d9.png">

> ▲ 타입힌트가 있을 때: 타입 힌트를 통해 리스트의 메서드를 추천해주는 것을 알 수 있다.
