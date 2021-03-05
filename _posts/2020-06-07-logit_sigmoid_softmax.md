---
title: Ligit, Sigmoid, Softmax의 관계
author: Gukwon Koo
categories: [ML, Basic]
tags: [logit, sigmoid, softmax]
pin: false
math: true
comments: true
---

> 왜 NN의 출력층에 sigmoid, softmax 함수를 사용할까요? 이는 출력층의 값을 '확률'로서 표현하기 위한 필연적 결과입니다. 본 글에서는 logit, sigmoid, softmax의 관계에 대해서 정리해보았습니다.

## 1. Basic 
앞으로 내용을 전개할 때 중요하게 사용되는 bayes theorem(베이즈 정리) 및 law of total probability(전확률 법칙)에 대해 간단히 정리하고 넘어가겠습니다.

### Bayes Theorem
불확실성 하에서 의사결정문제를 수학적으로 다룰 때 사용하는 것으로 두 확률 변수의 사전 확률과 사후 확률 사이의 관계를 나타내는 수식입니다. 공식은 아래와 같습니다.

$$
\begin{aligned}
P(Y|X) &= {\cfrac {P(X \cap Y)} {P(X)}} \\
P(X|Y) &= {\cfrac {P(Y \cap X)} {P(Y)}} \\
P(Y \cap X) &= P(X \cap Y) = P(X|Y)P(Y) = P(Y|X)P(X)\\
\therefore P(Y|X) &= {\cfrac {P(X \vert Y)P(Y)} {P(X)}} 
\end{aligned}
$$

- $$P(Y \vert X)$$: 사후확률(*posterior probability*)
- $$P(X \vert Y)$$: 가능도(*likelihood*)
- $$P(Y)$$: 확률변수 Y의 사전확률(*prior probability*)
- $$P(X)$$: 확률변수 X의 사전확률(*prior probability*)

### Law of total probability
표본 공간 $$S$$를 $$n$$개의 영역으로 나누었을 때, 표본 공간 $$S$$의 사건 $$A$$의 확률은 다음과 같이 구할 수 있습니다.

$$
P(A) = P(A \cap B_{1}) + P(A \cap B_{2})+ ... + P(A \cap B_{n}) 
$$

<br>

## 2. Logit & Sigmoid
logit은 log odds를 뜻합니다. 이를 이해하기 위해서는 odds에 대한 개념 학습이 선행되어야 하겠습니다. odds란 도박에서 얻을(pay off) 확률과 잃을 확률(stake)의 비율을 뜻합니다. 다시 말해 **두 확률의 비**를 뜻한다고 생각하면 되겠습니다. 도박이 아니라 NN task에서 흔히 다루는 이진 분류 문제에서 생각해봅시다.

- $$Classes: C_{1}, C_{2}$$
- $$Probabiliy \space of \space C_{1} \space given\space X: y=P(C_{1} \vert X)$$
- $$Probabiliy \space of \space C_{2} \space given\space X: 1-y=P(C_{2} \vert X)$$

우리는 하고자 하는 것은 $$X$$가 어떤 클래스일지 맞추는 간단한 문제를 해결하는 것입니다. 이 경우 우리는 간단하게 $$P(C_{1} \vert X) = y$$ 와 $$P(C_{2} \vert X) = 1-y$$ 중 큰 값을 $X$의 클래스로 예측하면 됩니다.

**그런데, 앞서 언급한 두 가지 수식을 하나의 수식으로 합쳐서 이와 같은 결정 문제를 표현 할 수 없을까요?** 확률의 비인 odds를 통해 해결할수 있습니다. 아래와 같이 말이죠.

$$
\begin{aligned}
odds &= \cfrac{y}{1-y} = \cfrac{P(C_{1}|X)}{1-P(C_{1}|X)} \\\\

Choose &= 
\begin{cases}
   C_{1} &\text{if } \cfrac{y}{1-y}>1 \\
   C_{2} &\text{if } \cfrac{y}{1-y}<1
\end{cases}
\end{aligned}
$$

한편 NN의 출력층의 값의 범위를 살펴봅시다. 출력층의 값을 $z$라고 하고, 단순화 시켜서 표현한다면 아래와 같습니다.

$$
z = \theta_{0}+\theta_{1}x_{1} + ... \\\\
$$

이때 $$z$$ 값의 범위는,

$$
-\infty < z < \infty
$$

NN 학습에서 주로 사용되는 손실함수는 크로스 엔트로피입니다. $P_{data}$는 데이터의 분포이며, $$P_{model}$$은 모델의 예측 분포라고 할 때, 크로스 엔트로피는 다음과 같이 나타낼 수 있습니다.

$$
H(P,Q)=−\sum_{x}P_{data}(x)logP_{model}(x)
$$

**여기서 중요한 점은 $$P_{data}(x)$$와 $$P_{model}(x)$$가 *확률*이라는 것입니다.** 그렇다면 $$z$$를 어떻게 ***확률***로 개념으로 변환시킬 수 있을까요? 결론적으로 말하면, ***$$sigmoid$$ 함수***를 통해 $$-\infty$$ ~ $$+\infty$$의 값을 $$0$$~$$1$$ 범위로 변환 할 수 있습니다. $$sigmoid$$ 함수는 $$logit$$ 함수에서 도출할 수 있습니다.

$$
\begin{aligned}
logits 
&= log(odds)\\
&= log\bigg(\cfrac{y}{1-y}\bigg) \\
&= log \bigg(\cfrac{P(C_{1}|X)}{1-P(C_{1}|X)} \bigg) \\
\end{aligned}
$$

$$
\therefore -\infty < logits < +\infty
$$

$$logits$$과 $$z$$의 범위가 같으니, 두 값을 동치로 놓고, 식을 전개해봅시다.

$$
\begin{aligned}
z 
&= log \bigg( \cfrac {y} {1-y} \bigg) ,\space e^{z} = \bigg( \cfrac {y} {1-y} \bigg) \\
\end{aligned}
$$

$$
\begin{aligned}
y 
&= \cfrac {e^z} {1+e^z} \\
&= \cfrac {(e^z)/e^z} {(1+e^z)/e^z} \\
&= \cfrac {1} {1+e^{-z}}
\end{aligned}
$$

위 식을 볼때, `logit`과 `sigmoid`는 역함수 관계임을 알 수 있으며, 2가지 중요한 사실을 도출할 수 있습니다.
- `logits`은 $$0$$~$$1$$의 범위를 $$-\infty$$ ~ $$+\infty$$로 변환
- `sigmoid`는 $$-\infty$$ ~ $$+\infty$$를 $$0$$~$$1$$로 변환

> 두 함수가 다음과 같은 관계가 있을 때, 두 함수 $$f$$와 $$g$$를 역함수 관계라고 합니다. 
> $$ f(x) = y \\ g(y) = x $$

<br>

## 3. Softmax
softmax는 sigmoid 함수를 클래스가 3개 이상일 때로 일반화 하면 유도할 수 있습니다. simoid를 일반화하면 softmax이며, softmax의 특수한 경우가 simoid 함수입니다. 유도 과정에 간단한 산수적인 테크닉이 필요합니다.

이진 분류일 때, odds는 아래와 같이 정의됩니다.

$$
\cfrac{P(C_{1}|X)}{P(C_{2}|X)} = e^{z}
$$

클래스의 개수가 $K$일때로 일반화 하면 위 식은 아래의 형태로 새로 정의할 수 있습니다.

$$
\tag{1}\cfrac{P(C_{i}|X)}{P(C_{k}|X)} = e^{z_{i}}
$$

양변을 $$i=1$$ 부터 $$K-1$$까지 더해보겠습니다.

$$
\sum_{i=1}^{K-1} \cfrac{P(C_{i}|X)}{P(C_{k}|X)} = \sum_{i=1}^{K-1} e^{z_{i}}
$$

위 식의 좌항의 분모는 $$\sum$$ 연산과 관련이 없기 때문에, 아래처럼 도출됩니다.

$$
\cfrac{P(C_{1}|X)+P(C_{2}|X)+...+P(C_{K-1}|X)}{P(C_{k}|X)} = \sum_{i=1}^{K-1} e^{z_{i}}
$$

확률의 합은 1이므로, 좌항의 분자를 변형하여 식을 아래와 같이 표현할 수 있습니다.

$$
\cfrac{1-P(C_{K} \vert X)}{P(C_{k} \vert X)} = \sum_{i=1}^{K-1} e^{z_{i}}
$$

$$P(C_{K} \vert X)$$에 대해서 정리하면,

$$
\tag{2}P(C_{K} \vert X)= \cfrac {1} {1+\sum_{i=1}^{K-1} e^{z_{i}}}
$$

$$(1)$$ 식은 아래와 같이 쓸수 있습니다.

$$
\tag{3} P(C_{i}|X) = e^{z_{i}}P(C_{k}|X)
$$

$$(2)$$식을 $$(3)$$식에 대입 합니다.

$$
P(C_{i}|X) = \cfrac {e^{z_{i}}} {1+\sum_{i=1}^{K-1} e^{z_{i}}}
$$

$$(1)$$에 $$i$$대신 $$k$$를 대입하면, $$e^{z_{k}} = 1$$ 이므로, **최종적으로 우리가 익히 아는 softmax 함수를 도출 할 수 있습니다.**

$$
\begin{aligned}
P(C_{i}|X) 
&= \cfrac {e^{z_{i}}} {e^{z_{i}}+\sum_{i=1}^{K-1} e^{z_{i}}} \\
&= \cfrac {e^{z_{i}}} {\sum_{i=1}^{K} e^{z_{i}}}
\end{aligned}
$$

## 참고자료
- [한 페이지 머신러닝](https://opentutorials.org/module/3653/22995)
- [Sigmoid, Logit and Softmax](https://chacha95.github.io/2019-04-04-logit/)

