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

오픈 소스 툴을 사내에 도입하기 위해 분석하던 중에 알게된 파이썬의 `__init__.py` 파일을 역할을 정리합니다. 먼저 왜 `__init__.py`의 역할과 기능을 정리하게 되었는지를 말씀드리고, 이를 이해하기 위한 배경지식을 말씀드리겠습니다. 그리고 `__init__.py`의 다양한 역할에 대해 정리합니다. 마지막으로 여러분들께 친숙한 pandas 패키지를 기준으로 `__init__.py`의 기능을 정리하겠습니다.

<br>

## 서론

---

![](/assets/img/post_img/pandas_package_structure.png){:width="250"}_없네...?_

오픈 소스 패키지의 소스 코드를 분석해보신 경험이 있으신가요? 혹시 없으시다면, 최소한 `pd.DataFrame()` 이라는 메서드는 써보신 적이 있을 것이라고 생각합니다. 저 역시도 정말 많이 사용 하는 메서드인데요. 저 코드가 어떻게 오류 없이 실행되는지 궁금해졌습니다. pandas를 설치하고, 설치된 경로를 따라가면 pandas의 패키지의 구조를 볼 수 있습니다.

그런데 정말 신기합니다. `pd.DataFrame()` 이라는 함수가 정상적으로 동작하려면 루트 디렉터리에 최소한 `DataFrame.py` 같은 모듈 파일이 있어야 하는데, 눈을 씻고 찾아봐도 그런 파일은 찾아볼 수 없습니다. 어떻게 `pd.DataFrame()` 이라는 코드가 오류 없이 작동하는 걸까요? 다음의 코드에서 힌트를 찾을 수 있습니다.

```python
>>> import pandas as pd
>>> print(pd.DataFrame)
pandas.core.frame.DataFrame
```

<br>

`pandas/core/frame.py` 파일에 DataFrame이라는 객체가 존재한다고 하네요. 그리고 실제로 해당 파일의 내용을 보면 아래와 같이 DataFrame 클래스를 찾을 수 있습니다.

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

<br>

그렇다면 `pd.core.frame.DataFrame`는 우리가 자주 쓰는 `pd.DataFrame` 와 같은 의미일까요? **맞습니다.** `pd.DataFrame`과 `pd.core.frame.DataFrame`은 완벽하게 같은 클래스를 가르킵니다.

```python
>>> print(pd.DataFrame)
pandas.core.frame.DataFrame
>>> print(pd.core.frame.DataFrame)
pandas.core.frame.DataFrame
```

<br>

마치 pandas 패키지 제작자들께서 정~말 자주 쓸 것 같은 `pd.DataFrame` 클래스를 쉽게 사용하라고 배려해주신 것 같은 느낌이 들어서 참 감사합니다. 하지만 궁금해집니다. 이게 어떻게 가능하지? **그 비밀은 `__init__.py` 모듈**에 있었습니다. 지금부터 그 비밀을 같이 파헤쳐 보시죠!

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

패키지란 여러 모듈들이나 패키지들을 모아 놓은 디렉터리를 말합니다. 디렉터리를 파이썬의 패키지로 인식시키려면 디렉터리 내부에 패키지를 초기화하는 `__init__.py` 파일이 필요합니다 (파이썬 3.3 이후 버전부터는 `__init__.py` 파일이 없어도 자동으로 패키지로 인식하도록 바뀌었습니다).

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

<br>

그리고 `main.py` 파일에서는 `sub.py`를 모듈로 임포트하고, 같은 이름을 가진 `func()` 함수를 선언했습니다.

```python
# main.py
import sub
def func():
  return "This is func() in main.py"
```

<br>

자, 그럼 main.py에서 자기 자신에서 선언한 `func()`과 sub.py에 선언된 `func()`를 둘다 사용할 수 있을까요? **가능합니다**. 바로 두 함수의 네임스페이스가 다르기 때문이죠.

```python
# main.py
>>> print(func())
This is func() in main.py
>>> print(sub.func())
This is func() in sub.py
```

<br>

먼저, `main.py` 함수에 선언된 `func()`는 `main.py` 모듈의 네임스페이스에 존재하는 함수입니다.  반면에 `sub.py`에 선언된 `func()`는 `sub.py` 모듈의 네임스페이스에 존재합니다. 이처럼 서로 같은 이름을 가진 객체가 서로 다른 네임스페이스에서 관리되므로 변수 이름의 충돌로 발생할 수 있는 문제가 방지되는 효과가 있습니다.

![](/assets/img/post_img/python_namespace.png){:width="500"}_python 네임스페이스_

<br>

## \_\_init\_\_.py

---

여러 오픈 소스 코드들을 보다보면 디렉터리에 `__init__.py` 라는 파일을 보신적이 있을실 것입니다. 이 요상한 `__init__.py`  파일은 패키지가 임포트 될 때 초기화(initialize)되어 실행됩니다. 이것은 파이썬의 내부 메커니즘으로서 패키지를 임포트하면 자동으로 해당 패키지의 `__init__.py` 파일이 실행됩니다.

이렇게 자동으로 초기화 되는 매커니즘으로 인해 여러가지 재미있는 일들이 가능해지는데요. 다음과 같은 일들을 할 수 있습니다. 이에 대해서 살펴보도록 하겠습니다.

- 파이썬의 디렉터리를 패키지로 인식
- `from <package_name> import *` 구문의 작동 방식 지정 가능
- 패키지의 네임스페이스에 객체를 할당

<br>

### 디렉터리를 패키지로 인식

가장 기본적인 기능은 디렉터리를 파이썬의 패키지로 인식시키는 것입니다. 이는 파이썬 모듈의 `__name__` 변수와 밀접한 관련이 있는데요. 이와 관련해서는 [여기](https://gguguk.github.io/posts/Python_relative_import_error/)를 참조해주세요. 결론적으로 말씀드리면, 상대 경로 임포트 방식으로 모듈을 import 하려면 모듈의 `__name__` 변수에 해당 모듈의 패키지 구조가 정의 되어 있어야 합니다. 그리고 이때 활용되는 `__init__.py`은 아무런 내용이 없어도 됩니다.

다만 파이썬 3.3 이후 버전부터는 `__init__.py` 파일이 없어도 디렉터리를 패키지로 인식한다고 합니다 ([PEP420](https://www.python.org/dev/peps/pep-0420/)). 그럼에도 불구하고 이전 버전과의 호환성을 위해서는 웬만하면 `__init__.py` 파일을 작성해 두는 것이 좋을 것 같습니다.

<br>

### from \<package_name\> import * 구문의 작동 방식

예시를 보면서 설명 해볼게요. 어떤 프로젝트의 구조가 다음과 같습니다.

```
.
├── main.py
└── sub/
    ├── __init__.py
    ├── first.py
    └── second.py
```

그리고 각각의 모듈은 다음과 같이 구성 되어 있습니다.

```python
# main.py
from sub import *
print(first.first())
print(second.second())

# sub/__init__.py

# sub/first.py
def first():
  return "***** first!!! *****"
  
# sub/second.py
def second():
  return "***** second!!! *****"
```

<br>

이때 `main.py` 파일을 실행해보면(`python main.py`) 예상과는 다르게 에러가 출력됩니다. `first` 모듈을 찾을 수가 없다고 하네요. 

```python
Traceback (most recent call last):
  File "main.py", line 2, in <module>
    print(first.first())
NameError: name 'first' is not defined
```

<br>

어떻게 된 일이죠? 분명히 패키지의 모든 것을 import 했는데(`*`) 왜 `first` 모듈을 찾을 수 없는 걸까요. 비밀은 `__init__.py` 파일에 있습니다. `__init__.py`에는 `__all__` 이라는 리스트를 선언할 수 있습니다. 그리고 이 `__all__` 변수에 담긴 모듈만 임포트됩니다. 

예를 들어 `first` 모듈만 임포트 되도록 하려면 다음과 같이 `__init__.py` 모듈을 수정하면 됩니다.

```python
# sub/__init__.py
__all__ = ["first"]
```

<br>

그리고 나서 `main.py`를 다시 실행하면(`python main.py`) 다음과 같이 `first` 모듈에 있는 함수에는 접근할 수 있지만 `second` 모듈에 있는 함수에는 접근 할 수 없다는 것을 알 수 있습니다.

```python
***** first!! *****
Traceback (most recent call last):
  File "main.py", line 3, in <module>
    print(second.second())
NameError: name 'second' is not defined
```

<br>

### 패키지의 네임스페이스에 객체를 할당

오픈 소스 패키지의 구조를 이해하기 위해서는 이 원리를 이해하는 것이 가장 중요합니다. 다음과 같은 프로젝트가 있다고 가정해 봅시다.

```
.
├── main.py
└── sub/
    └── __init__.py
```

<br>

그리고 각 모듈은 다음과 같이 작성 되어 있습니다.

```python
# main.py
import sub
print(sub.sub_value)

# sub/__init__.py
sub_value = "sub_value"
```

<br>

`main.py` 파일을 실행(`python main.py`) 하면 어떻게 될까요? "sub_value"라는 값을 정상적으로 출력할 것입니다. 이게 어떻게 가능할까요? `main.py` 파일을 실행하면 내부적으로 다음의 과정을 거칠 것입니다.

1. `main.py`에서 `sub` 패키지를 임포트 한다
2. `sub` 패키지를 초기화 하면서 `__init__.py`을 실행한다.
3. `__init__.py` 내에 선언된 객체들은 모두 `sub` 패키지의 네임스페이스에 소속된다.

<br>

이제 pandas 패키지를 들여다보고 판다스에서 `__init__.py`를 어떻게 활용하는지를 살펴보겠습니다.

<br>

## pandas에서 \_\_init\_\_과 네임스페이스를 활용하는 방법

---

pandas에서는 `__init__.py`과 네임스페이스의 개념을 활용하여 자주 사용하는 메서드를 쉽게 접근 하도록 설계되어 있습니다. 서론에서 살펴본 것처럼 아무래도 `pd.DataFrame()`로 접근하는 것이 `pd.core.frame.DataFrame()`로 접근 하는 것 보다 훨씬 편하죠. 정말 자주 사용되는 객체이니까요.

<br>

pandas 패키지의 구조에서 `DafaFrame()`과 관련된 핵심적인 부분만 따로 보면 다음과 같습니다.

```
.
├── __init__.py
├── core
│   ├── __init__.py
│   ├── api.py
│   ├── frame.py
│   └── ...
└── ...
```

<br>

각 파일의 내용은 다음과 같습니다.

```python
# ./__init__.py
from pandas.core.api import DataFrame

# ./core/__init__.py


# ./core/api.py
from pandas.core.frame import DataFrame

# ./core/frame.py
class DataFrame(NDFrame):
  pass

```

<br>

이를 바탕으로 우리가 `pd.Dataframe()` 이라는 메소드를 사용하기 위한 과정을 따져보면 다음과 같을 것입니다.

1. `import pandas as pd`
2. `pandas/__init__.py` 파일 실행 
   1. `pandas/core` 패키지의  `api` 모듈 로드
   2. `api` 모듈은 `pandas/core` 패키지의 `frame` 모듈에서 `DataFrame` 클래스(객체) 로드
   3. `__init__.py`는 `pandas` 패키지에서 초기화 된 것이므로 `DataFrame` 클래스는 `pandas` 패키지의 네임스페이스에 소속
3. `pd.DataFrame()` 메소드 사용 가능

<br>

## 마무리

지금까지 파이썬 패키지를 계층화 할때 중요한 역할을 하는 `__init__.py` 파일의 역할을 알아 보았습니다. 다시 한번 정리하면...

- 디렉터리를 패키지로 인식하게 함
- `from <package_name> import *` 구문의 작동 방식 결정
- 패키지의 네임스페이스에 객체를 할당

이러한 역할 이해하면 앞으로 오픈 소스 패키지의 구조를 이해하는데 많은 도움이 될 것이라고 믿습니다. 긴글 읽어 주셔서 감사합니다!

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