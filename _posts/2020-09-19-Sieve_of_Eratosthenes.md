---
title: 에라토스테네스의 체
author: Gukwon Koo
categories: [Algorithms, Theory]
tags: [algorithms]
pin: false
math: true
comments: true
---

## 1 &nbsp; 에라토스테네스의 체란?
고대 그리스 수학자 에라토스테네스가 고안한 N까지의 수열에서 소수만을 골라내는 알고리즘입니다. 소수를 대량으로 빠르게 판별할 수 있는 장점이 있습니다.

## 2 &nbsp; 알고리즘 구현 원리
- **Step1.** 먼저 판별하고 싶은 범위의 수열을 초기화 합니다. 예시로 2부터 120까지의 자연수($$N=120$$) 중 소수를 판별하는 과정을 들어보도록 하겠습니다.

$$
arr = [2, 3, 4, 5, ..., 120] 
$$

- **Step2.** 배열의 첫번째 원소인 2를 제외한 2의 배수를 모두 제거합니다
- **Step3.** 배열의 다음 원소인 3을 제외한 3의 배수를 모두 제거합니다.
- **Step4.** 배열의 다음 원소인 4는 2의 배수를 제거하는 과정에서 이미 제거 되었으므로 넘어갑니다.
- **Stpe5.** 배열의 다음 원소인 5를 제외한 5의 배수를 모두 제거합니다.
- **Step6.** 이와 같은 과정을 $$sqrt{(N)} = \sqrt{120}$$ 이하인 자연수까지만 반복하고 배열에 남아 있는 원소를 출력합니다.

## 3 &nbsp; 파이썬 구현
```python
import numpy as np

n = 10000 # 10000까지의 자연수 중 소수를 골라내 보자
mask = [True for _ in range(2, n+1)] # 2부터 n까지의 자연수 배열
_n = int(np.sqrt(len(mask))) # _n까지만 배수 삭제 작업을 진행

for i in range(_n):
    # True는 삭제 되지 않은 숫자
    if mask[i]:
        # 자기 자신을 제외한 배수들의 mask를 False로 바꾸기
        for j in range(i+(i+2), len(mask), i+2):
            mask[j] = False

print([prime+2 for prime, b in enumerate(mask) if b])
```

## Reference
- [에라토스테네스의 체-위키피디아](https://ko.wikipedia.org/wiki/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98_%EC%B2%B4)
- [에라토스테네스의 체-나무위키](https://namu.wiki/w/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98%20%EC%B2%B4)