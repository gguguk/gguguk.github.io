---
title: 오픈 소스 패키지 분석을 위한 __init__.py 알아보기 
author: Gukwon Koo
categories: [Python, Basic]
tags: [python, packages, module, init]
pin: false
math: true
comments: false
date: 2021-10-31 17:09:00 +0900
---

오픈 소스 툴을 사내에 도입하기 위해 분석하던 중에 알게된 파이썬의 \_\_init\_\_.py 파일을 역할을 정리합니다. 여러분들께 친숙한 pandas를 기준으로 설명해보겠습니다.

<br>

## 서론

---

![](/assets/img/post_img/pandas_package_structure.png){:width="250"}_없네...?_

오픈 소스 패키지의 소스 코드를 분석해보신 경험이 있으신가요? 혹시 없으시다면, 최소한 `pd.DataFrame()` 이라는 메서드는 써보신 적이 있을 것이라고 생각합니다. 저 역시도 정말 많이 사용 하는 메서드인데요. 저 코드가 어떻게 오류 없이 실행되는지, 패키지의 파일 구조가 궁금해졌습니다. pandas를 설치하고, 설치된 경로를 따라가면 pandas의 패키지의 구조를 볼 수 있습니다.

그런데 정말 신기합니다. `pd.DataFrame()` 이라는 함수가 정상적으로 동작하려면 루트 디렉터리에 최소한 `DataFrame.py` 같은 모듈 파일이 있어야 하는데, 눈을 씻고 찾아봐도 그런 파일은 찾아볼 수 없습니다. 어떻게 `pd.DataFrame()` 이라는 코드가 오류 없이 작동하는 걸까요?

다음의 코드에서 힌트를 찾을 수 있습니다.

```python
>>> import pandas as pd
>>> print(pd.DataFrame)
pandas.core.frame.DataFrame
```

`pandas/core/frame.py` 파일에 클래스 또는 함수로 DataFrame이라는 객체가 존재한다고 하네요. 그리고 실제로 해당 파일의 내용을 보면 아래와 같이 DataFrame 클래스를 찾을 수 있습니다.

```python
# pandas/core/frame.py
class DataFrame(NDFrame):
  def __init__(
    self,
    data=None,
    index: Optional[Axes] = None,
    columns: Optional[Axes] = None,
    dtype: Optional[Dtype] = None,
    copy: bool = False,
  ):

```

그렇다면 `pd.core.frame.DataFrame`는 우리가 자주 쓰는 `pd.DataFrame` 클래스와 같은 클래스를 가르킬까요? **맞습니다.** `pd.DataFrame`과 `pd.core.frame.DataFrame`은 완벽하게 같은 클래스를 가르킵니다.

```python
>>> print(pd.DataFrame)
pandas.core.frame.DataFrame
>>> print(pd.core.frame.DataFrame)
pandas.core.frame.DataFrame
```

마치 pandas 패키지 제작자들께서 정~말 자주 쓸 것 같은 `pd.DataFrame` 클래스를 쉽게 사용하라고 배려해주신 것 같은 느낌이 들어서 참 감사합니다. 하지만 궁금해집니다. 이게 어떻게 가능하지? **그 비밀은 `__init__.py` 모듈**에 있었습니다. 지금부터 그 비밀을 같이 파해쳐 보시죠!

<br>

## 배경지식

---

### 모듈(module)

> An object that serves as an organizational unit of Python code. Modules have a namespace containing arbitrary Python objects. Modules are loaded into Python by the process of [importing](https://docs.python.org/3/glossary.html#term-importing).

모듈이란 간단히 말해서 다른 파이썬 스크립트에서 import 하는 `.py` 파일입니다. caller[^1]에서 모듈을 임포트하면 해당 모듈 파일이 실행 되면서 해당 모듈 내부의 object(함수, 클래스, 변수 등)가 모듈의 네임스페이스에 할당됩니다. 

물론 모듈 임포트를 어떻게 하냐에 따라서 caller의 네임스페이스에 할당될 수도 있습니다. 네임스페이스에 관련해서는 밑에서 더 자세히 다루겠습니다.

<br>

### 패키지(package)

> A Python [module](https://docs.python.org/3/glossary.html#term-module) which can contain submodules or recursively, subpackages. Technically, a package is a Python module with an `__path__` attribute.

패키지란 여러 모듈들이나 패키지들을 모아 놓은 디렉터리를 말합니다. 디렉터리를 파이썬의 패키지로 인식시키려면 디렉터리 내부에 패키지를 초기화하는 `__init__.py` 파일이 필요합니다. 다만 파이썬 3.3 이후 버전부터는 `__init__.py` 파일이 없어도 자동으로 패키지로 인식하도록 바뀌었습니다. 그럼에도 불구하고 하위 버전과의 호환성을 위해서 패키지 내부에는 `__init__.py` 파일을 작성하는 것이 바람직하다고 생각합니다.

앞서 언급하였지만 패키지는 모듈뿐만 아니라 또 다른 패키지를 포함할 수 있습니다. 패키지를 도입함으로써 프로젝트에서 여러가지 모듈을 계층화 시켜서 관리할 수 있는 이점을 얻을 수 있습니다.

특히 패키지를 이야기 할 때 `__init__.py`을 언급하지 않을 수 없는데요. `__init__.py` 파일은 생각보다 다양한 기능이 있습니다. 또한 여러분들이 아시는 대부분의 오프 소스 패키지를 분석할 때 헷갈리는 부분이 바로 이 `__init__.py` 내부의 코드 때문에 발생합니다. 따라서 `__init__.py`은 밑에서 더 자세히 다루도록 하겠습니다.

<br>

### 네임스페이스(namespace)

> The place where a variable is stored. Namespaces support modularity by preventing naming conflicts. Namespaces also aid readability and maintainability by making it clear which module implements a function.

네임스페이스란 프로그래밍 언어에서 이름(name)에 따라 특정 객체(object)를 구분할 수 있는 범위(scope)를 뜻합니다. 파이썬의 모든 것은 객체입니다. 이 객체를 특정 이름을 가진 범위마다 따로 구분해서 관리하는 것입니다. 

예를 들어 보죠. 다음과 같이 `sub.py` 파일에 `func()` 함수를 선언했습니다.

```python
# sub.py
def func():
  return "This is func() in sub.py"
```

그리고 `main.py` 파일에서는 `sub.py`를 모듈로 임포트하고, 같은 이름을 가진 `func()` 함수를 선언했습니다.

```python
# main.py
import sub
def func():
	return "This is func() in main.py"
```

자, 그럼 main.py에서 자기 자신에서 선언한 func()과 sub.py에 선언된 func()를 둘다 사용할 수 있을까요? 가능합니다. 바로 두 함수의 네임스페이스가 다르기 때문이죠.

```python
# main.py
>>> print(func())
This is func() in main.py
>>> print(sub.func())
This is func() in sub.py
```

먼저, `main.py` 함수에 선언된 `func()`는` main.py`의 네임스페이스에 존재하는 함수입니다. 반면에 `sub.py`에 선언된 `func()`는 `sub.py`의 네임스페이스에 존재합니다. 이처럼 서로 같은 이름을 가진 객체가 서로 다른 네임스페이스에서 관리되므로 변수 이름의 충돌로 발생할 수 있는 문제가 방지되는 효과가 있습니다.

![](/assets/img/post_img/python_namespace.png){:width="500"}_python 네임스페이스_

<br>

## \_\_init\_\_.py

---

### \_\_init\_\_.py 이란?

- 패키지나 패키지 내부의 모듈을 import 할 때, 자동으로 실행됨.
- 디렉터리를 파이썬 패키지로 인식시킴.
  - 파이썬 3.3 이후 버전에서는 implicit name space 활용하므로 큰 의미 없음
- 

### 오픈 소스 패키지에서 활용하는 방법

- 디렉터리를 패키지로 인식
- `from <package_name> import *` 구문의 작동 방식 결정

<br>

## pandas 패키지에서 \_\_init\_\_과 이름공간을 활용하는 방법

---

pandas에서는 __init__과 이름공간을 활용하여 자주 사용하는 객체를 쉽게 접근 하도록 해 놓은 것 같습니다. 서론에서 살펴본 것 처럼 아무래도 `pd.DataFrame()`로 접근하는 것이 `pd.core.frame.DataFrame()`로 접근 하는 것 보다 훨씬 편하죠. 정말 자주 사용되는 객체이니까요.

<br>

## Reference

---

- [Python glossary - module](https://docs.python.org/3/glossary.html#term-module)
- [Python glossary - package](https://docs.python.org/3/glossary.html#term-package)
- [Python glossary - namespace](https://docs.python.org/3/glossary.html#term-namespace)
- [python 3.7 document - Packages](https://docs.python.org/3.7/tutorial/modules.html#packages)
- [Python import: Advanced Techniques and Tips](https://realpython.com/python-import/)
- [What’s \_\_init\_\_ for me?](https://towardsdatascience.com/whats-init-for-me-d70a312da583)
- [Python Modules and Packages - Python packages](https://realpython.com/python-modules-packages/#python-packages)
- [Python Modules and Packages – Package Initialization](https://realpython.com/python-modules-packages/#package-initialization)
- [Python Modules and Packages - Importing * From a Package](https://realpython.com/python-modules-packages/#importing-from-a-package)
- [[파이썬] 내장 함수 dir 사용법](https://www.daleseo.com/python-dir/)
- [[Python] 네임스페이스 개념 정리](https://hcnoh.github.io/2019-01-30-python-namespace)

<br>

## Footnote

[^1]: 모듈을 임포트한 `.py` 파일을 말합니다. 예를 들어 `main.py` 파일에서 `mymodule.py`을 임포트 했다면(`import mymodule`) `main.py`가 **caller**입니다.