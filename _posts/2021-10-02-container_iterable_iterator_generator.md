---
title: 파이썬 이터러블(iterable), 이터레이터(iterator), 제너레이터(generator)의 차이
author: Gukwon Koo
categories: [Python, Basic]
tags: [python, iterable, iterator, generator, container, sequence]
pin: false
math: false
comments: false
date: 2021-10-02 19:03:00 +0900
---

본 글은 ['우리를 위한 프로그래밍 : 파이썬 중급'](https://www.inflearn.com/course/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%8C%8C%EC%9D%B4%EC%8D%AC-%EC%A4%91%EA%B8%89-%EC%9D%B8%ED%94%84%EB%9F%B0-%EC%98%A4%EB%A6%AC%EC%A7%80%EB%84%90/dashboard)를 수강 후(내돈내산!!💰) 헷갈렸던 `iterable`, `iterator`, `generator`의 차이를 정리합니다. 파이썬을 중급 이상 수준에서 활용하시는 분들은 자주 들어보셨을 만한 개념인데요. 

파이썬 활용 수준을 업그레이드 하기 위해서 반드시 짚고 넘어가야 하는 개념이지 않나 싶습니다. 그래서 이번 기회에 매우 헷갈리는 세 친구들의 차이점을 정리해보겠습니다!! 하나하나에 대해 자세히 다루진 않고 차이점 위주로 정리합니다.

본 글에서는 아래의 주제를 다룹니다.

- container(컨테이너)
- sequence(시퀀스)
- iterable(이터러블)
- iterator(이터레이터)
- generator(제너레이터)

<br>

## Container(컨테이너)

---

**컨테이너(container)란 임의의 객체를 담을 수 있는 파이썬 객체**를 말합니다. 아시다시피 파이썬의 모든 것은 객체이죠. `int`, `float`, `str`, 심지어 `function`도 객체입니다. 이러한 객체 중에서 다른 객체들을 담을 수 있는 객체, 즉 컨테이너는 다음 같은 것들이 있습니다.

- list
- tuple
- str
- set
- dictionary
- ...

좀 더 practical 하게는 membership 테스트를 수행할 수 있는 객체라고 말하기도 합니다. membership 테스트란, `x in y`[^1]와 같은 operation을 수행하는 것을 말합니다. 말이 좀 어려울 수 있는데, 아래 예시를 보시면 쉽게 이해가 가능합니다.

```python
print('a' in ['a', 'b', 'c'])  # True
print('a' in ('a', 'b', 'c'))  # True
print('a' in {'a', 'b', 'c'})  # True
print('a' in {'a': 1, 'b': 2, 'c': 3})  # True
print('a' in 'abc')  # True
```

<br>

## Sequence(시퀀스)

---

**시퀀스(sequence)란 다음과 같은 특징을 가지는 컨테이너(container)입니다.**

- 원소의 순서가 유지됨
- 정수 인덱싱을 통해 원소(element)를 꺼내올 수 있음
- 길이가 있음

시퀀스의 예시는 다음과 같습니다.

- list
- tuple
- str
- ...

헷갈릴만한 점은 모든 컨테이너가 시퀀스가 되지는 않는다는 것입니다. 예를 들어서 `set`은 대표적인 컨테이너지만 원소의 순서가 보장되지 않고, 정수 인덱싱을 지원하지 않습니다.

<br>

## Iterable(이터러블)

---

**iterable은 원소를 한번에 하나씩 반환할 수 있는 객체입니다.** 모든 sequence와 일부 non-sequence가 해당됩니다.

- Sequence
  - list
  - tuple
  - str
- Non-sequence
  - dictionary

일반적으로 iteralble에는 `__iter__`[^2]와 `__getitem__`[^3] 매직 메소드가 구현되어 있습니다. iterable과 가장 헷갈리는 개념이 iterator인데요. iterable은 말 그대로 원소를 **반환할 수 있는** 또는 **반환할 능력**을 가지고 있는 객체를 뜻합니다. 이 말은 iterable이 실제로 원소를 반환하는 행위를 할 수 있다는 것을 의미하지는 않죠. 미묘하지만 iterator와의 큰 차이점입니다. 

그렇다면 iterable이라고 해서 반드시 iterator라고 할 수 있을까요 🤔?? 답은 '아니오' 입니다 🙅‍♀️. 가장 대표적인 iterable로 `list`가 있는데, 여러분들께서 잘 아시다시피 `list` 자체는 원소를 반환하지 못합니다.

```python
li = [1, 2, 3]
print(next(li))  # TypeError: 'list' object is not an iterator
```

오류 내용을 보면 `list`는 iterator가 아니라고 합니다. 따라서 `list`에 담긴 원소를 바로 꺼내오지는 못하는 것이죠.  iterable 이라고 해서 iterator라고 할 수 없다는 사실을 확인 할 수 있습니다.

이런 의문이 생기실 수도 있을 것 같습니다. for loop에 넣으면 바로 원소를 꺼내올 수 있지 않나요 🙋‍♀️? for loop에 어떤 객체를 태우면 이를 [내부적으로 iterator로 변환해서 사용](https://www.geeksforgeeks.org/python-difference-iterable-iterator/)합니다. 결국 원소를 하나씩 꺼내오려면 iterable이 아니라 iterator가 필요한 것이죠.

<br>

## Itertator(이터레이터)

---

**iterator란 순차적으로 원소를 하나씩 꺼내올 수 있는 객체**를 뜻합니다. 실제로 원소를 꺼내올 때는 `next()` 메소드를 이용해서 원소를 하나씩 꺼내옵니다. 만약 더이상 꺼내올 데이터가 없다면 `StopIteration` 에러를 발생시킵니다. iterable 컨테이너는 `__iter__()` 매직 메소드를 통해 iterator가 될 수 있습니다. iterator가 되었을 때 비로소 원소를 하나씩 꺼내오는 행위를 할 수 있게 됩니다.

iterable 객체인 `list`는 그 자체로는 원소를 하나씩 꺼내오지 못하는데요. iterator로 변환하면 가능합니다.

```python
li = [1, 2, 3]
iterator = iter(li)

print(type(iterator))  # <type 'list_iterator'>
print(next(iterator))  # 1
```

파이썬의 모든 것은 객체입니다. 그리고 그 객체에 매직메서드를 정의하면 파이썬의 규칙에 따라 잘 작동하는 커스텀 객체를 구현할 수 있는데요. iterator를 클래스를 통해 직접 만들 수 있습니다. `__iter__()`[^4] 메서드와 `__next__()` 메서드만 구현되어 있으면 돼요.

다음은 `start` 부터 `stop-1`까지 숫자를 1씩 증가시키면서 제곱수를 리턴하는 iterator의 예시입니다.

```python
class Squares:
    def __init__(self, start, stop):
        self.start = start
        self.stop = stop
    def __iter__(self):
        return self
    def __next__(self):
        if self.start >= self.stop:
            raise StopIteration
        current = self.start * self.start
        self.start += 1
        return current

iterator = Squares(2, 5)

print(next(iterator))  # 4
print(next(iterator))  # 9
print(next(iterator))  # 16
print(next(iterator))  # StopIteration
```

간단한 연산을 하는 것인데, 코드가 생각보다 길죠? 좀 더 간단하게 할 수 없을까요?? 미리 정답을 알려드리자면 🤫 generator를 활용하면 됩니다.

<br>

## Generator(제너레이터)

---

**generator는 iterator의 특성을 모두 가지고 있으면서, `yield` statement로 작성된 객체**입니다. 따라서 **generator는 특수한 형태의 iterator** 라고 할 수 있습니다. generator와 iterator 모두 container에 담긴 원소를 하나씩 꺼내올 수 있는 객체라는 점에서는 공통점을 가지고 있습니다. 그러나 generator가 iterator와 구분되는 다음과 같은 특성이 있습니다.

- 함수 내에 `yield` statement이 존재합니다.
- 호출 되면 `yield` 까지 코드를 실행하고 멈춥니다. 다음 번에 호출될 때는 이전 `yield` 부터 다음 `yield`까지 코드를 실행합니다. 이러한 작업을 `StopIteration` 에러가 발생할 때까지 진행합니다.
- [lazy implementation](https://en.wikipedia.org/wiki/Lazy_initialization)으로서 generator가 **호출**되었을 때 리소스를 할당합니다. **생성** 되었을 때 리소스를 할당하지 않습니다.

generator를 사용하면 iterator 섹션에서 살펴 보았던 제곱수를 리턴하는 함수를 다음과 같이 훨씬 간결하게 만들 수 있습니다.

```python
def squares(start, stop):
    for i in range(start, stop):
        yield i**2

generator = squares(2, 5)

print(type(generator))  # <class 'generator'>
print(next(generator))  # 4
print(next(generator))  # 9
print(next(generator))  # 16
print(next(generator))  # StopIteration
```

<br>

generator는 iterator의 특성을 모두 가지고 있기 때문에 gerator는 iterator라고 할 수 있습니다. 그러나 그 반대는 성립하지 않습니다.

![](/assets/img/post_img/iterator_generator.png){:width="300"}

## Reference

---

- [[Document] Python glossary: sequence](https://docs.python.org/2/glossary.html#term-sequence)
- [[Document] Python glossary: iterable](https://docs.python.org/2/glossary.html#term-iterable)
- [[Document] Python glossary: iterator](https://docs.python.org/2/glossary.html#term-iterator)
- [[Document] 파이썬 데이터 모델: \_\_contains\_\_()](https://docs.python.org/ko/3.7/reference/datamodel.html#object.__contains__)
- [[Document] 파이썬 데이터 모델: \_\_iter\_\_()](https://docs.python.org/ko/3.7/reference/datamodel.html#object.__iter__)
- [[Document] 파이썬 데이터 모델: \_\_getitem\_\_()](https://docs.python.org/ko/3.7/reference/datamodel.html#object.__getitem__)
- [[Blog] 실용 파이썬 프로그래밍: 프로그래밍 유경험자를 위한 강좌 - 시퀀스](https://wikidocs.net/84391)
- [[Blog] python iterable과 iterator의 의미](https://bluese05.tistory.com/55)
- [[Blog] Python - 시퀀스 자료형](https://velog.io/@suasue/Python-시퀀스-자료형)
- [[Blog] Iterables vs. Iterators vs. Generators](https://nvie.com/posts/iterators-vs-generators/)
- [[Blog] Python - Difference between iterable and iterator](https://www.geeksforgeeks.org/python-difference-iterable-iterator/)
- [[Blog] Python Generators: Memory-efficient programming tool](https://medium.com/learning-better-ways-of-interpretting-and-using/python-generators-memory-efficient-programming-tool-41f09077353c)
- [[Stackoverflow] What exactly are “containers” in python? (And what are all the python container types?)](https://stackoverflow.com/questions/11575925/what-exactly-are-containers-in-python-and-what-are-all-the-python-container)
- [[Stackoverflow] Difference between Python's Generators and Iterators](https://stackoverflow.com/questions/2776829/difference-between-pythons-generators-and-iterators)
- [[Stackoverflow] Generator function (yield) much faster then iterator class (next)](https://stackoverflow.com/questions/43729052/generator-function-yield-much-faster-then-iterator-class-next)
- [[Stackoverflow] What does the "yield" keyword do?](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do/31042491#31042491)

<br>

## Footnote

---

[^1]:`x in y` operation은 객체에 `__contains__` 매직 메서드가 구현되어 있기 때문에 가능합니다([참고](https://docs.python.org/ko/3.7/reference/datamodel.html#object.__contains__)).
[^2]:원소를 하나씩 호출할 수 있는 iterator를 반환하는 매직 메서드입니다([참고](https://docs.python.org/ko/3.7/reference/datamodel.html#object.__iter__)).
[^3]: `self[key]`의 값을 구하기 위해 호출되는 매직 메서드입니다([참고](https://docs.python.org/ko/3.7/reference/datamodel.html#object.__getitem__)).
[^4]: 자기 자신을 return 합니다. `for` 나 `in` statement에서 활용됩니다([참고](https://docs.python.org/2/library/stdtypes.html#iterator.__iter__)).

