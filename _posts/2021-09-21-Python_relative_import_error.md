---
title: 파이썬 상대 경로 임포트시 에러
author: Gukwon Koo
categories: [Python, Debug]
tags: [python, relative-import, debug]
pin: false
math: false
comments: false
date: 2021-09-21 16:49:00 +0900
---

최근에 딥러닝 코드를 리팩토링 하면서 프로젝트의 폴더 구조를 변경하는 작업을 진행하였습니다. 이 과정에서 모듈을 임포트 할 때 상대 경로를 활용하였는데요. 직관과 다르게 에러가 발생하였습니다. 파이썬에서 모듈 임포트 방법을 정리하면서 문제를 해결한 사례를 공유 합니다.

# 프로젝트 구조

---

먼저 문제가 발생한 프로젝트 구조를 재현해 보겠습니다.

## 폴더 구조

```
/example/                 
│
├── network/         
│   └── model.py
├── utils/           
│   └── serving.py
└── train.py
```



## network/model.py

```python
from ..utils.serving import serve_func

class Model:
  def train(self):
    return "Train Model"
```



## utils/serving.py

```python
def serve_func():
    return "This is servce func"
```



## train.py

```python
from network.model import Model

model = Model()
model.train()
```



# 문제 상황

---

위와 같은 프로젝트 구조에서 `train.py`를 실행시키면 다음과 같은 에러가 발생합니다.

<br>

***Attempted relative import beyond top-level package***

<br>

일단 에러의 내용을 해석하기 전에 저의 사고 과정을 보겠습니다.

1. `train.py`를 실행하게 되면 `network/model.py`에 있는 클래스를 가져옵니다. 
2. 이때 `network/model.py`에 적힌 코드를 차례대로 실행합니다. 
3. `network/model.py`의 첫줄에 상대 경로를 활용해서 `utils/serving.py` 모듈에 있는 함수를 불러오는 군요.
   - `from ..utils.serving import serve_func`
4. `network/model.py`와 `utils/serving.py`는 sibling 관계입니다.
5. 따라서 `network/model.py` 기준으로 부모 패키지로 한 단계 거슬러 올라가면 sibling 모듈인 `utils/serving.py`를 찾을 수 있을 것 같군요.
6. 그런데 에러 발생!!

<br>

애석하게도 직관에 반하여 에러가 발생하고 말았습니다. 에러 내용은 상대 경로 임포트를 활용할 때 파이썬 인터프리터가 인지 하고 있는 부모 패키지의 level을 넘어서 더 상위 부모 패키지를 찾으려고 해서 발생 한 것입니다. 자세한 내용을 차차 알아 봅시다.

# 문제 원인

---

이러한 에러가 발생한 이유는 파이썬에서 절대경로/상대경로 모듈 임포트의 동작 원리를 정확히 파악하지 못했기 때문입니다. 파이썬에서는 절대 경로 모듈 임포트와 상대 경로 모듈 임포트의 동작 방식에 큰 차이가 있습니다. 그 차이를 알아보겠습니다.

## 절대 경로 임포트

절대 경로로 모듈을 임포트 할 때는 [sys.path](https://docs.python.org/3/library/sys.html#sys.path)에 등록된 경로에서 모듈을 찾습니다. sys.path에는 기본적으로 여러가지 경로가 리스트 형식으로 등록되어 있는데요. 중요한 점은 **sys.path의 첫번째 경로는 항상 실행된 파이썬 스크립트가 속한 폴더의 절대 경로라는 것입니다.**

<br>

> path[0] is the directory containing the script that was used to invoke the Python interpreter.

<br>

예를들어, `train.py`에서 `print(sys.path)` 명령어를 실행하면 다음과 같은 결과를 얻을 수 있습니다.

```python
["/example", ...]
```

`train.py` 스크립트가 속한 폴더의 절대 경로가 존재함을 알 수 있습니다.

<br>

## 상대 경로 임포트

상대 경로로 모듈을 임포트 할 때는 `__name__` 변수에 담긴 값을 통해 패키지의 계층 구조를 확인 합니다. 모두 잘 아시다시피 파이썬 스크립트 파일(`.py`)은 직접 실행할 수도 있고, 모듈로서 다른 파이썬 스크립트에서 실행될 수도 있습니다.

`__name__`은 에 담긴 값은 파이썬 스크립트가 어떻게 실행되느냐에 따라 달라집니다. 예를 들어서 파이썬 스크립트가 직접 실행되면 `__name__`에는 `__main__`이라는 값이 할당됩니다. 한편 다른 모듈에서 임포트되어 실행되면 *`package.module`*과 같이 **모듈이 속한 패키지의 계층 구조를 알려주는 값이 할당** 됩니다.

예를 들어 `network/model.py`에 `print(__name__)`를 넣어서 `train.py`를 실행시켜 보면, `network/model.py`은 다음의 결과를 리턴합니다.

```python
'network.model'
```

정리해보면, `network/model.py`는 `train.py`에서 모듈로서 임포트 되었는데요. 이 때 파이썬은 `__name__`에 담긴 값을 통해 `model.py`의 부모 패키지가 `network`임을 알아 낼 수 있는 것입니다.

마지막으로 **상대 경로 임포트는 파이썬 스크립트가 모듈로 실행되었을 때만 활용할 수 있다**는 사실을 알 수 있습니다. 왜냐하면 직접 실행 되었을 때는 `__name__`에  `__main__`이라는 스트링이 할당되는데, 저 값을 통해서는 모듈이 속한 패키지의 계층 구조를 전혀 알 수 없기 때문입니다.

<br>

## 원인 파악

자, 우리는 이제 모듈이 절대 경로로 임포트 될 때와 상대 경로로 임포트 될 때의 차이를 명확히 알았습니다. 절대 경로 임포트는 `sys.path`에 할당된 경로에서 모듈을 찾는 다는 점이 중요했습니다. 한편 상대 경로 임포트는 `__name__`에 담긴 값을 통해 모듈이 속한 패키지의 계층 구조를 알 수 있어야 한다는 점이 중요했습니다.

다시 에러 내용을 살펴봅시다.

<br>

***Attempted relative import beyond top-level package***

<br>

모듈을 상대 경로로 임포트 했는데, 파이썬 인터프리터가 인지하고 있는 계층 구조를 넘어서는 top-level로 거슬러 올라갔다는 에러 내용입니다. 

에러를 일으킨 직접적인 코드는 `network/model.py`의 `from ..utils.serving import serve_func`입니다. `network/model.py`가 모듈로서 실행될 때 `__name__`에는 `network/model`이라는 값이 담겨 있습니다. 그런데 저 코드는 `network`라는 패키지보다 더 상위의 패키지를 찾으려고 시도합니다. 그러나 파이썬 인터프리터는 `network`보다 더 상위의 패키지가 무엇인지를 `__name__` 에 담긴 값으로는 전혀 알 수가 없습니다. 따라서 에러가 발생한 것입니다. 이제 해결책을 알아볼까요?



# 해결 방안

---

해결 방법은 매우 다양한데요. 저는 개인적으로 절대 경로 임포트를 활용하는 방법이 가장 깔끔하다고 생각하였습니다. 이 방법은 파이썬 스크립트에 다른 코드를 덕지덕지 붙일 필요가 없습니다. 또한 가독성 측면에서도 가장 우수하다고 판단했습니다. `network/model.py`의 내용을 다음과 같이 간단하게 수정하면 됩니다.

여기서

```python
from ..utils.serving import serve_func
```

이렇게 말이죠.

```python
from utils.serving import serve_func
```

<br>

왜 정상 실행되는지 알아야겠죠? 계속 말씀 드린 것처럼 절대 경로 임포트시에는 `sys.path`에 등록된 경로에서 모듈을 찾습니다. `train.py` 실행하면 `sys.path`에 `train.py`가 속한 폴더의 절대경로(`/example/`)가 등록되어 있습니다. 따라서 파이썬 인터프리터는`/example/` 경로에서 `utils/serving.py`를 충분히 찾을 수 있기 때문에 에러가 나지 않고 정상 실행 되는 것입니다.

# Reference

---

- [sys.path, PYTHONPATH: 파이썬 파일 탐색 경로](https://www.bangseongbeom.com/sys-path-pythonpath.html)
- [Relative imports for the billionth time](https://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time)
- [python import from sibling folder](https://stackoverflow.com/questions/48690887/python-import-from-sibling-folder)
- [파이썬 자습서 6.4 패키지](https://docs.python.org/ko/3/tutorial/modules.html#packages)
- [[Python\] Python 3의 상대경로 import 문제 피해가기](https://blog.potados.com/dev/python3-import/)
- [Relative imports in Python 3](https://stackoverflow.com/questions/16981921/relative-imports-in-python-3)
- [PEP 328 -- Imports: Multi-Line and Absolute/Relative](https://www.python.org/dev/peps/pep-0328/#guido-s-decision)