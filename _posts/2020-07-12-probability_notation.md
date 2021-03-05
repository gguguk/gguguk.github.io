---
title: Probability Notation
author: Gukwon Koo
categories: [Math, Statistics]
tags: [probability]
pin: false
math: true
comments: true
---

> 확률과 관련된 공부를 하면서, semicolon(;), commam(,), vertical line(\|)이 포함된 probability notation을 많이 보았습니다. 그러나 그 의미가 혼용되어 사용되는 것을 발견하고 그것을 정리하기 위해 포스팅을 작성합니다.

<br/>

## 1 &nbsp; $$P(x\vert\theta)$$를 바라보는 2가지 관점
$$P(x\vert\theta)$$는 2가지 관점에서 사용되는 것을 목격할 수 있습니다. $$x$$를 변수, $$\theta$$를 상수로 바라보는 관점과 $$x$$를 상수, $$
\theta$$를 변수로 바라보는 관점이 그것입니다. 두 가지 관점은 어느 부분을 더욱 **강조**할 것인가와 관련되어 있습니다.


### 1.1 &nbsp; $$P(x\vert\theta)$$는 $$x$$에 대한 함수($$\theta$$는 상수)

$$\theta$$는 RV가 아니라 상수로 취급되면, $$P(x\vert\theta)$$는 모델의 파라미터 $$\theta$$가 주었졌을 때의 $$x$$의 확률입니다. $$\theta$$가 상수이므로(확률변수가 아님) 조건부 확률이라고 부르지 않습니다. 또한 $$P(x\vert\theta)$$는 $$P(x;\theta)$$ 또는 $$P_{\theta}(x)$$로도 표현됩니다.

만일 $$\theta$$가 RV이면, $$P(x\vert\theta)$$는 조건부 확률이며, $$\cfrac {P(X, \theta)} {P(\theta)}$$로 정의됩니다.

### 1.2 &nbsp; $$P(x\vert\theta)$$는 $$\theta$$에 대한 함수($$x$$는 상수)

이 경우는 $$P(x\vert\theta)$$를 최대로 하는 $$\theta$$를 찾는 MLE에서의 likelihood를 의미합니다. 이는 $$L(\theta\vert x)$$로 표현할 수 있습니다. 따라서 likelihood는 $$\theta$$에 값을 할당하여 얻을 수 있는 특정 데이터 $$x$$에 대한 확률인 $$P(x\vert\theta)$$을 표기하는 약어(shorthand)에 불과합니다. 따라서 이러한 관점에서의 $$P(x\vert\theta)$$는 objective 함수로 주로 사용됩니다.

<br/>

## 2 &nbsp; likelihood

## Reference
- [What is the difference between “likelihood” and “probability”?](https://stats.stackexchange.com/questions/2641/what-is-the-difference-between-likelihood-and-probability)