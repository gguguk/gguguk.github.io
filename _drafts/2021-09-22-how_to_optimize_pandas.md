---
title: 판다스 데이터 프레임 속도 최적화
author: Gukwon Koo
categories: [Python, Pandas]
tags: [python, pandas, apply]
pin: false
math: false
comments: false
date: 2021-09-22 15:28:00 +0900
---

최근에 전처리 코드를 리팩토링하면서 나름 큰 dataframe을 다룰 일이 있었습니다. 이 때 습관적으로 `apply()` 함수를 활용해서 데이터를 처리하였습니다. 그런데 문득 궁금해졌습니다. 

<br>

> pandas의 `apply()` 함수 진짜 빠른 것 맞아? 더 빠른 방법은 없나?

<br>

`apply()` 함수가 for loop를 도는 것에 비해 속도 상의 이득이 있다고 언뜻 들어서 습관적으로 활용하긴 했는데... 진짜 빠를까요? 🤔 혹시 더 빠르게 대용량의 데이터를 처리하는 방법이 있을까요? 이와 관련해서 조사한 내용을 공유 합니다.

<br>

## 판다스 자료 구조.

---

본격적으로 코드 최적화 하기에 앞서 판다스에서 활용하는 자료 구조에 대해 간략하게 알아봅시다. 코드 최적화를 위한 기초 지식으로서 알아둘 필요가 있습니다.

판다스에서 다루는 대표적인 자료 구조로는 **시리즈(Series)**와 **데이터프레임(DataFrame)**이 있습니다.

- **시리즈(Series)**
  - label(*index*)이 달려 있는 1차원 array입니다. 
  - **numpy ndarray**에 판다스의 편의적인 기능을 추가하여 wrapping한 것입니다.
- **데이터프레임(DataFrame)**
  - label(*index*, *column name*)이 달려 있는 2차원의 matrix입니다. 
  - **여러 개의 판다스 시리즈로 구성**되어 있습니다.

<br>

## 판다스에서 많은 데이터를 처리하는 방법들.

---

판다스에서는

<br>

## 누가 누가 빠를까?

---

속도 최적화를 위해서 다양하게 고려해야 할 것이 많겠지만, 위의 예시에서 우리는 중요한 두 가지 포인트를 발견할 수 있습니다.

- 연산 비용이 큰 부분을 최적화 하자.
- 반복 실행되는 횟수를 줄이자.

<br>

## 가장 효율적인 시나리오

- 숫자를 다룰 때?
  - 무조건 vectorization. 특히 series가 아니라 numpy의 vectorization 활용
- 숫자를 다루지 않을 때?

<br>

## 언제 apply를 쓸까?

- groupby 연산과 함께

<br>

## Pandas Series Vectorization vs Numpy ndarray Vectorization

- 가장 큰 차이는 판다스 Series는 label을 기반으로 데이터를 자동으로 정렬한 후에 벡터 연산[^1]을 한다는 점입니다. 

  >  A key difference between Series and ndarray is that operations between Series automatically align the data based on label.

- 이 말을 코드 최적화 관점에서 생각해보면 판다스 Series의 벡터 연산은 numpy ndarray 벡터 연산에 비해 추가적인 작업이 수반된다는 것입니다.

- 따라서 numpy ndarray 벡터 연산이 판다스 series 벡터 연산에 비해 조금이라도 더 빠른 것이라고 할 수 있습니다. overhead가 작습니다.

<br>

## Reference

---

- [A Beginner’s Guide to Optimizing Pandas Code for Speed](https://engineering.upside.com/a-beginners-guide-to-optimizing-pandas-code-for-speed-c09ef2c6a4d6)
- [Sofia Heisler No More Sad Pandas Optimizing Pandas Code for Speed and Efficiency PyCon 2017](https://www.youtube.com/watch?v=HN5d490_KKk)
- [When should I (not) want to use pandas apply() in my code?](https://stackoverflow.com/questions/54432583/when-should-i-not-want-to-use-pandas-apply-in-my-code)
- [Pandas User Guide Document](https://pandas.pydata.org/pandas-docs/stable/user_guide/index.html#user-guide)
  - [Intro to data structures](https://pandas.pydata.org/pandas-docs/stable/user_guide/dsintro.html#intro-to-data-structures)
  - [Vectorized operations and label alignment with Series](https://pandas.pydata.org/pandas-docs/stable/user_guide/dsintro.html#vectorized-operations-and-label-alignment-with-series)
- [Nuts and Bolts of NumPy Optimization Part 1: Understanding Vectorization and Broadcasting](https://blog.paperspace.com/numpy-optimization-vectorization-and-broadcasting/)

<br>

## Footnote

---

[^1]: numpy 벡터 연산도 본질적으로는 loop를 돕니다. 그러나 numpy 벡터 연산이 파이썬 loop에 비해 빠른 이유는 미리 컴파일 되고 최적화된 C 코드에 loop를 위임하기 때문입니다. numpy의 함수들은 C로 작성된 코드의 wrapper일 뿐입니다. 따라서 numpy는 loop 연산을 파이썬 보다 훨씬 효율적인 C 레벨에서 수행할 수 있습니다([참고1](https://blog.paperspace.com/numpy-optimization-vectorization-and-broadcasting/), [참고2](https://numpy.org/doc/stable/glossary.html#term-vectorization)). 