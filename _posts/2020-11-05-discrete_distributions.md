---
title: 이산형 확률 분포 정리
author: Gukwon Koo
categories: [Math, Statistics]
tags: [statistics, distribution]
pin: false
math: true
comments: true
date: 2020-11-05 20:53:00 +0900
---

이산형 확률 분포에 대해서 정리해 보았습니다. 본 내용은 하버드 확률론 기초 강의(Statistics 110)를 참고하여 정리했음을 밝힙니다.

연속형 확률 분포에 대한 글은 [이곳]()을 참고해주세요.

<br>

## 0 &nbsp; 확률 변수(Random Variable)와 확률 분포(Distribution)

---

확률 변수란 표본 공간($$S$$)에서 발생 가능한 outcome을 **실수 공간**에 맵핑 시키는 **함수**를 의미합니다. 확률 변수는 **함수**이기 다음과 같이 표현 할 수 있습니다.



$$
X:S \rightarrow \mathbb{R}
$$



확률 분포는 확률 변수와 깊은 관련성으로 인해 헷갈리기 쉬운 개념입니다. 확률 변수와 확률 분포의 공통점은 함수라는 점입니다. 그러나 함수의 역할이 다르다는 점에서 차이가 있는데, **확률 분포**란 확률 변수가 **특정한 값을 가질 확률**을 나타내는 **함수**를 의미합니다.[^1] 확률 분포는 확률 변수가 어떤 종류의 값을 가지느냐에 따라서 크게 이산 확률 분포와 연속 확률 분포로 구분됩니다. 이산 확률 분포는 확률 질량 함수(PMF, Probability Mass Function)로 표현 가능 하며, 연속 확률 분포는 확률 밀도 함수(PDF, Probability Density Function)으로 표현 가능 합니다.

<br>

## 1 &nbsp; 이산 확률 변수(Discrete Random Variable) 분포

---

### 1.1 &nbsp; 이항 분포(Binomial Distribution)

#### 1.1.1 정의

**독립적인 베르누이 시행**을 $$n$$번 했을 때 **성공 횟수 $$k$$**에 대한 분포입니다.
$$
X \sim Bin(n,p)
$$

#### 1.1.2 PMF

이항 분포의 PDF는 아래와 같습니다.

$$
P(X=k)= {n \choose k} p^{k}(1-p)^{n-k}
$$

> 이항분포에 왜 조합(combination)이 있을까요? 독립적인 동전 던지기 시행을 5번 했을 때, 2번 성공(=앞면이 나옴) 했을 때를 가정해봅시다. 첫번째, 세번째에서 동전의 앞면이 나왔을 때와 두번째 다섯번째에서 동전이 앞면이 나오는 경우는 모두 같은 확률 $$p^{2}(1-p)^{3}$$입니다. 그러나 두 경우 모두 $$X=2$$ 사건이므로 이러한 경우를 모두 고려하여 확률 분포를 고안해야 합니다. 다시말해, 5개 중 2개를 순서와 상관없이 뽑는 경우를 모두 고려해야하며, 각각의 사건은 서로 배반사건(disjoint, 동시에 발생할 수 없음)입니다.

#### 1.1.3 PMF 검증

먼저 이항 정리(binomial theorem)에 대해서 간단하게 알아보겠습니다. 이항 정리란 \\((p+q)\\)의 거듭 제곱의 전개식을 말합니다[^2]. 참고로 먼저 이항(二項)은 두 개의 항이라는 뜻이며, 항을 옮긴다는 이항(移項))의 뜻이 아님을 주의해주세요.


$$
\sum_{k=0}^{n} {n \choose k}p^{k}q^{n-k} = (p+q)^n
$$


이항 정리를 알아본 이유는 이항 정리를 활용하면 PMF를 검증할 수 있기 때문입니다. 아래 식을 살펴 보시면 이항 분포를 따르는 확률 변수 $$X$$에 대응되는 모든 확률의 합이 $$1$$이 된다는 것을 쉽게 알 수 있습니다. 위에서 언급한 이항 정리 식이 어떻게 쓰였는지 살펴 보시길 바랍니다.

$$
\begin{aligned}
P(S) 
&= P(X=0) + P(X=1) + \cdots + P(X=n) \\[1em]
&= \sum_{k=0}^{n}p^{k}(1-p)^{n-k} = (p+(1-p))^n \\[1em]
&= 1^n = 1
\end{aligned}
$$



#### 1.1.4 기댓값

향후 추가



####  1.1.5 분산

이항 분포의 분산은 지시 확률 변수(indicator random variable)을 활용하여 구할 수 있습니다. 먼저, 지시 확률 변수란 어떤 사건(event)가 발생 했을 때는 1, 발생하지 않았을 때는 0을 출력하는 함수입니다.


$$
I = \begin{cases}
   1 &\text{if } \space \text{event  occured} \\
   0 &\text{if } \space \text{otherwise}
\end{cases}
$$



이항 분포는 독립적인 베르누이 시행에서의 성공 횟수에 대한 분포임을 언급 했습니다. 다시 말해서 이항 분포의 확률 변수는 베르누이 시행의 지시 확률 변수의 합으로 표현할 수 있습니다.



$$
\text{Let} \space X \sim Bin(n,p), \\[1em]
X = I_1 + \cdots + I_n, \text{where} \space I_j \space \text{are} \space \text{i.i.d.} \space Bern(p)
$$



먼저 베르누이 분포의 분산 $$Var(I_j)$$를 구해보겠습니다.



$$
\begin{aligned}
Var(I_j)
&= E(I_{j}^2) - E(I_{j})^2 \\[1em]
&= \sum_{x \in\{0,1\}}x^{2}P(X=x) - p^2 \\[1em]
&= p-p^2 \\[1em]
&= p(1-p)
\end{aligned}
$$



이항 분포의 확률 변수는 베르누이 지시 확률 변수의 합이므로 이항 분포의 분산은 다음과 같이 도출됩니다.



$$
\begin{align}
  Var(X)
  &= Var(I_{1} + \cdots + I_{n}) \\[1em]
  &= Var(I_{1}) + \cdots + Var(I_{n}) \Leftarrow \text{by independence} \\[1em]
  &= p(1-p) + \cdots + p(1-p) \\[1em]
  &= np(1-p)
\end{align}
$$



<br>

### 1.2 &nbsp; 기하 분포(Geometric Distribution)

#### 1.2.1 정의

**독립적인 베르누이 시행**에서 **첫 성공**까지의 **실패 횟수**에 대한 분포입니다. 참고로 기하 분포와 비슷한 의미를 지니는 연속형 확률 분포로 지수 분포(exponential distribution)이 있습니다.


$$
X \sim Geom(p)
$$



#### 1.2.2 PMF



$$
P(X=k) = q^kp
$$



#### 1.2.3 PMF 검증



$$
\sum_{k=0}^{\infty}q^kp = p\sum_{k=0}^{\infty}q^k = \cfrac{p}{1-q} = \cfrac{1-q}{1-q} = 1
$$



#### 1.2.4 기댓값

향후 추가



#### 1.2.5분산

향후 추가

<br>

### 1.3 &nbsp; 음이항 분포(Negative Binomial Distribution)

#### 1.3.1 정의

**독립적인 베르누이 시행**에서 $$r$$번 성공하기 까지의 실패 횟수에 대한 분포입니다. 참고로 음이항 분포와 비슷한 의미를 지니는 연속형 확률 분포로는 감마 분포(gamma distribution)를 들 수 있습니다.



$$
X \sim NB(r, p)
$$

#### 1.3.2 PMF

향후 추가



#### 1.3.3 PMF 검증

향후 추가



#### 1.3.4 기댓값

향후 추가



#### 1.3.5 분산

향후 추가



<br>

### 1.4 &nbsp; 초기하 분포(Hypergeometric Distribution)

#### 1.4.1 정의

크기가 $$N$$인 모집단에서 $$n$$개의 표본을 **비복원 추출** 했을 때 **성공 횟수** $$k$$에 대한 분포입니다. 예를 들어 파란 공이 담긴 주머니에 8개의 공이 있고 빨간 공이 담긴 주머니에 2개의 공이 있다고 가정해봅시다. 이때 모집단의 크기는 10인데, 여기서 비복원 추출로 2개를 뽑았을 때, 파란 공이 뽑힌 개수는 초기하 분포를 따릅니다. 같은 행위를 복원 추출로 하였을 때는 이항 분포가 됩니다.



$$
X \sim HGeom(N, n, p)
$$



#### 1.4.2 PMF


$$
P(X=k) = \cfrac { {w \choose k} {b \choose {n-k}} } { {N \choose n} }
$$



#### 1.4.3 PMF 검증


$$
\begin{align}
  \sum_{k=0}^{n} P(X=k) 
  &= \sum_{k=0}^{n} \cfrac {\binom wk \binom b{n-k}} {N \choose n} \\[1em] 
  &= \cfrac {1} {N \choose n} \sum_{k=0}^{n}{w \choose k} {b \choose n-k} \\[1em]
  &= \cfrac {N \choose n} {N \choose n} = 1
\end{align}
$$



참고로 3번째 term에서 4번째 term으로의 변화는 Vandermonde 항등식에 의해서 가능합니다.



$$
{w+b \choose k} = \sum_{k=0}^{n} {w \choose k} {b \choose n-k}
$$

#### 1.4.4 기댓값

향후 정리

#### 1.4.5 분산

향후 정리

<br>

### 1.5 &nbsp; 포아송 분포(Poisson Distribution)

#### 1.5.1 정의

**단위 시간/공간** 동안의 **사건 발생 횟수**에 대한 이산 확률 분포를 말합니다. 모수인 $$\lambda$$는 단위 시간 동안의 평균 사건 발생 횟수를 뜻합니다. 여기서 '평균'이라는 단어를 쓴 이유는, 포아송 분포의 기댓값(평균)이 $$\lambda$$이기 때문입니다.


$$
P(X=k) \sim Pois(\lambda)
$$



#### 1.5.2 PMF



$$
P(X=k) = \cfrac{\lambda^ke^{-\lambda}}{k!}
$$



#### 1.5.3 PMF 검증

먼저 우리는 $$e^\lambda$$의 talyor series(expansion)을 아래와 같이 나타낼 수 있다는 것을 알고 있습니다.



$$
e^{\lambda} = \sum_{k=0}^{\infty}\frac{\lambda^k}{k!}
$$



이와 같은 사실을 활용해 PMF를 검증해보겠습니다.



$$
\sum_{k=0}^{\infty} \cfrac{\lambda^k e^{-\lambda}}{k!} = e^{-\lambda} \sum_{k=0}^{\infty} \cfrac{\lambda^k}{k!} = e^{-\lambda}e^{\lambda}=1
$$
​	

#### 1.5.4 기댓값


$$
\begin{aligned}
E(X) 
&= e^{-\lambda} \sum_{k=0}^{\infty} k \cfrac{\lambda^k}{k!} \\[1em]
&= e^{-\lambda} \sum_{k=1}^{\infty} \cfrac{\lambda^k}{(k-1)!} \\[1em]
&= \lambda e^{-\lambda} \sum_{k=1}^{\infty} \cfrac{\lambda^{k-1}}{(k-1)!} \\[1em]
&= \lambda e^{-\lambda} e^{\lambda} \\[1em]
&= \lambda
\end{aligned}
$$

#### 1.5.5 분산

향후 추가



#### 1.5.6 이항 분포와의 관계

먼저 이항 분포를 '시간'의 맥락에서 생각해 보겠습니다. 음식점에 방문하는 손님의 수를 이항 분포로 모델링 하는데, 하루동안 음식점에 한명의 고객이 방문할 확률을 0.3이라고 해보죠. 그렇다면 10일간 4명의 고객이 방문할 확률은 $$P(X=4)={10 \choose 4}0.3^40.7^6$$으로 계산할 수 있습니다. 그러나 이러한 계산은 직관적으로 보았을 때 어딘가 부족해 보입니다.

위의 예시를 이항 분포로 모델링할 경우에는 하루동안 음식점에 방문하는 손님이 1명이라는 가정이 내재되어있습니다. 그런데 이러한 가정은 너무 비현실적이죠. 하루동안 여러명이 손님이 방문할 수 있다는 정보를 반영하여 모델링을 할 수 없을까요? 시간을 더욱 작은 단위로 쪼개면 가능할 것 같습니다. 예를들어 24시간이라는 1시간 단위로 잘게 쪼개서 한 시간 단위로 방문자 수를 모델링하면 결국 하루 동안의 방문자가 여러명이 될 수도 있다는 정보를 반영할 수 있게 됩니다.

이항 분포의 시행을 최대한 잘게 쪼갠댜는 말은 결국 시행을 무한번하게 되는 것과 동치가 됩니다. 이러한 맥락에서 포아송 분포가 등장하게 됩니다. 포아송 분포는 단위 시간당 평균 사건 발생 횟수를 나타내는 $$\lambda$$를 모수로 가지고 있습니다. 그리고 이항 분포의 기대값은 $$np$$입니다. 평균 사건 발생 횟수를 의미하죠. 따라서 $$\lambda=np$$로 셋팅 해놓고 $$n$$을 무한대로 보내면 $$p$$는 0으로 수렴하는 방식으로 포아송 분포를 도출할 수 있을 것 같습니다.


$$
\begin{aligned}
\lim_{n \rightarrow\infty \\ p \rightarrow0}P(X=k) 
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} {n \choose k}p^k(1-p)^{n-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} {n \choose k} \bigg(\cfrac{\lambda}{n}\bigg)^k\bigg(1-\cfrac{\lambda}{n}\bigg)^{n-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} {n \choose k} \bigg(\cfrac{\lambda}{n}\bigg)^k\bigg(1-\cfrac{\lambda}{n}\bigg)^{n}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{n!}{(n-k)!k!} \bigg(\cfrac{\lambda}{n}\bigg)^k\bigg(1-\cfrac{\lambda}{n}\bigg)^{n}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{n!}{(n-k)!n^k} \cfrac{\lambda^k}{k!}\bigg(1-\cfrac{\lambda}{n}\bigg)^{n}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{n(n-1)\cdots(n-k+1)}{n^k} \cfrac{\lambda^k}{k!}\bigg(1-\cfrac{\lambda}{n}\bigg)^{n}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{n}{n} \cfrac{n-1}{n} \cdots \cfrac{n-k+1}{n} \cfrac{\lambda^k}{k!}\bigg(1-\cfrac{\lambda}{n}\bigg)^{n}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{\lambda^k}{k!}\bigg(1-\cfrac{\lambda}{n}\bigg)^{n}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{\lambda^k}{k!}e^{-\lambda}\bigg(1-\cfrac{\lambda}{n}\bigg)^{-k} \\
&= \lim_{n \rightarrow\infty \\ p \rightarrow0} \cfrac{\lambda^k}{k!}e^{-\lambda} \\
&= \cfrac{\lambda^k}{k!}e^{-\lambda}
\end{aligned}
$$

>  참고로 지수 함수에 대한 다음의 식이 활용 되었습니다.
> $$
> e^\lambda = \lim_{n \rightarrow \infty}\bigg(1+ \cfrac{\lambda}{n} \bigg)^n \\[1em]
> e^{-\lambda} = \lim_{n \rightarrow \infty}\bigg(1 - \cfrac{\lambda}{n} \bigg)^n
> $$
> 



### 1.6 &nbsp; 다항 분포(Multinomial Distribution)

#### 정의

$$k(\ge 3)$$개의 상태(outcome)을 가질 수 있는 독립적인 experiment를 $$n$$번 시행 했을 때, 각 상태의 발생 횟수에 대한 이산 확률 분포입니다. 이항 분포를 상태가 $$k(\ge 3)$$개 발생할 경우로 일반화 하는 것이라고 생각하면 이해하기 수월합니다. 여기서 $$X_k$$는 $$k$$번째 상태가 발생한 횟수를 나타내며, $$p_k$$는 $$k$$번째 상태가 발생할 확률을 나타냅니다.

$$
\vec{X} \sim Mult_k(N, \vec{p})
$$

$$
\vec{X}=(X_1, \cdots, X_k), \space \vec{p}=(p_1, \cdots, p_k)
$$



#### PMF


$$
\text{if} \space \sum_k n_k=n \space \text{and} \space 0 \space \text{otherwise}\\
P(X_1=n_1, \cdots, X_k=n_k)=\cfrac{n!}{n_1!, \cdots, n_k!}\space p_{1}^{n_1} \cdots p_{k}^{n_k}
$$





#### PMF 검증

향후 추가

#### 기댓값

향후 추가

#### 분산

향후 추가

<br>

## 참고자료

---

[^1]: [위키피디아: 확률 분포](https://ko.wikipedia.org/wiki/%ED%99%95%EB%A5%A0_%EB%B6%84%ED%8F%AC)
[^2]: [위키피디아: 이항 정리](https://ko.wikipedia.org/wiki/%EC%9D%B4%ED%95%AD_%EC%A0%95%EB%A6%AC)



