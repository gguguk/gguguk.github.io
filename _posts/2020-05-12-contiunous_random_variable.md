---
title: Continuous Random Variable
author: Gukwon Koo
categories: [Math, Statistics]
tags: [continuous, random variable]
pin: false
math: true
comments: true
---

> 이상화 교수님의 확률 및 통계 5강 '이산 확률 변수와 연속 확률 변수' 강의를 듣고 간단하게 내용을 정리해보도록 하겠습니다.

## 1 &nbsp; 왜 연속 확률 변수는 특정 실수값의 확률을 정의할 수 없을까?

0과 1사이의 '모든' 실수값들 중에서 '0.5'라는 숫자를 뽑을 확률을 정의해봅시다. 0과 1사이에는 무수히 많은 숫자들이 있을 것이고. 그 중에서 0.5라는 숫자를 뽑을 확률을 실질적으로는 '0'이라고 생각할 수 있습니다.

$$
\cfrac{1}{\infty} = 0
$$

<br>

## 2 &nbsp; 그렇다면 연속 확률 변수는 '확률'을 어떻게 정의해야 할까?

'특정한' 값이 아니라 '아주 작은 구간'에서 정의할 수 있다. $$F(x)$$에 아주 작은 값을 더한 값을 구한 후($$F(x+\Delta x)$$) , 이를 단위 구간으로 나누어 봅시다.


$$
\begin{aligned}
\lim_{\Delta \to 0}\cfrac{P(x<X<x+\Delta x)}{\Delta x} &= \lim_{\Delta \to 0} \cfrac{F(x+\Delta x )-F(x)}{\Delta x}\\\\
&=F'(x)
\end{aligned}
$$


이를 통해 다음과 같은 사실을 알 수 있습니다.

- 연속 확률 변수에서 특정한 값의 '확률'을 정의할 수는 없지만 '밀도(density)'는 정의할 수 있습니다(밀도는 단위 길이당 특정 확률값을 표현한 것).
- 밀도는 누적 분포 함수를 '미분' 하면 구할 수 있습니다. 즉 아래와 같은 식이 성립합니다.
  
$$
\begin{aligned}
F'(x) &= f(x)\\
F(x) &= \int f(x)dx
\end{aligned}
$$



- $$f(x)$$는 확률 밀도 함수(probability density function, PDF)라고 합니다.

확률 밀도 함수(PDF)가 되기 위해서는 다음의 두 조건을 반드시 만족해야 합니다.

- $$0\leq f(x)$$, (그러나 $$f(x) \leq1$$일 필요는 없다. 확률이 아니라 밀도이기 때문).
-  $$\int_{-\infty}^{\infty} f(x) =1$$.

<br/>

## 3 &nbsp; 마지막으로 헷갈릴만한 개념들을 정리 해봅시다.

- *PMF*의 경우, 특정 확률 변수에 상응하는 값이 '확률'입니다.
- *그러나 PDF*의 경우, 특정 확률 변수에 상응하는 값이 '확률'이 아니라 '밀도'입니다.
- 누적 분포 함수(*CDF*)는 특정 확률 변수에 해당하는 값이 '확률'입니다. 이는 *PMF*의 *CDF*나 *PDF*의 *CDF* 모두에 공통적으로 해당됩니다.