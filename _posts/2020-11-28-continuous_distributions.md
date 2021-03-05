---
title: 연속형 확률 분포 정리
author: Gukwon Koo
categories: [Math, Statistics]
tags: [statistics, distribution]
pin: false
math: true
comments: true
date: 2020-11-28 16:26:00 +0900
---

하버드 확률론 기초 강의(Statistics 110)을 수강 후, 교수님께서 강조하신 여러가지 분포에 대해 개념정리 할 필요성을 느꼈습니다. 특히 이커머스 도메인에서는 통계에 기반한 분석 기법들이 많이 활용되는데(LTV, MAB 등), 실무에서 주어진 태스크를 빠르고 정확히 수행하기 위해 미리 관련 개념 정리를 해두려고 합니다. 이번 포스트는 대표적인 연속형 확률 분포(정규분포, 지수분포, 베타분포, 감마분포)의 정의, PDF, 기대값, 분산에 대해 정리해보았습니다. 그리고 보너스로 지수 분포, 감마분포, 포아송 분포를 한꺼번에 다루는 포아송 과정(poisson process)에 대해서도 미약하지만 정리해보았습니다.

본 내용은 하버드 확률론 기초 강의(Statistics 110)를 참고하여 정리했음을 밝힙니다. 이산형 확률 분포에 대한 글은 [이곳](https://gguguk.github.io/posts/discrete_distributions/)을 참고해주세요.

<br>

## 0 &nbsp; 확률 변수(Random Variable)와 확률 분포(Distribution)

---

확률 변수란 표본 공간($$S$$)에서 발생 가능한 outcome을 **실수 공간**에 맵핑 시키는 **함수**를 의미합니다. 확률 변수는 **함수**이기 다음과 같이 표현 할 수 있습니다.



$$
X:S \rightarrow \mathbb{R}
$$



확률 분포는 확률 변수와 깊은 관련성으로 인해 헷갈리기 쉬운 개념입니다. 확률 변수와 확률 분포의 공통점은 함수라는 점입니다. 그러나 함수의 역할이 다르다는 점에서 차이가 있는데, **확률 분포**란 확률 변수가 **특정한 값을 가질 확률**을 나타내는 **함수**를 의미합니다.[^1] 확률 분포는 확률 변수가 어떤 종류의 값을 가지느냐에 따라서 크게 이산 확률 분포와 연속 확률 분포로 구분됩니다. 이산 확률 분포는 확률 질량 함수(PMF, Probability Mass Function)로 표현 가능 하며, 연속 확률 분포는 확률 밀도 함수(PDF, Probability Density Function)으로 표현 가능 합니다.

<br>

## 1 &nbsp; 연속 확률 변수(Continuous Random Variable) 분포

---

### 1.1 &nbsp; 균등 분포(Uniform Distribution)

#### 1.1.1 정의

일정 구간의 확률 밀도가 모두 동일한 연속 확률 분포입니다. 모수인 $$a$$와 $$b$$는 각각 구간의 최솟값과 최댓값을 나타냅니다.



$$
U \sim Unif(a, b)
$$



#### 1.1.2 PMF


$$
f_{U}(u) = 
\begin{cases}
   c\big(=\cfrac{1}{b-a}\big) & a \le x \le b \\
   0 &\text{otherwhise}
\end{cases}
$$



#### 1.1.3 PMF 검증



$$
\int_{a}^{b}cdu = c(b-a) = 1
$$



#### 1.1.4 기댓값



$$
\begin{aligned}
E(U) 
&= \int_{a}^{b}uf_U(u)du \\[1em]
&= \int_{a}^{b}\cfrac{u}{b-a}du \\[1em]
&= \cfrac{b^2-a^2}{2(b-a)} \\[1em]
&= \cfrac{(b+a)(b-a)}{2(b-a)} \\[1em]
&= \cfrac{a+b}{2}
\end{aligned}
$$

#### 1.1.5 분산



$$
\begin{aligned}
Var(U) 
&= E(U^2) - E(U)^2 \\[1em]
&= \int_{a}^{b}u^2f_{U}(u)du - \bigg(\cfrac{a+b}{2}\bigg)^2 \\[1em]
&= \int_{a}^{b}u^2\cfrac{1}{b-a}du - \bigg(\cfrac{a+b}{2}\bigg)^2 \\[1em]
&= \cfrac{b^3-a^3}{3(b-a)} - \cfrac{(b+a)^2}{4} \\[1em]
&= \cfrac{4(a-b)(a^2+ab+b^2) - (b+a)^2}{12} \\[1em]
&= \cfrac{(b-a)^2}{12}\\[1em]
\end{aligned}
$$

<br>

### 1.2 &nbsp; 표준 정규 분포(Standard Normal/Gaussian Distribution)

#### 1.2.1 정의

평균이 0(zero average)이고 분산이 1(unit variance)인 연속 확률 분포입니다.


$$
X \sim Normal(0, 1)
$$



#### 1.2.2 PDF

표준 정규 분포의 PMF를 유도하기 위해서는 가우스 적분[^1], 이중 적분, 극 좌표계, 삼각함수 등의 사전 지식이 필요함을 언급해 둡니다. 표준 정규 분포의   PDF는 다음과 같이 나타낼 수 있습니다. 여기서 \\(\cfrac{1}{\sqrt{2\pi}}\\\)는 해당 식의 적분 값을 1로 만들기 위한 정규화 상수(normalizing constant) 입니다. 적분 값을 1로 만들어야 하는 이유는 확률의 합이 반드시 1이 되어야 하기 때문입니다.



$$
f_Z(z) = \cfrac{1}{\sqrt{2\pi}}e^{-z^2/2}
$$



정규화 상수를 미지수로 둔 후에 \\(\sqrt{2\pi}\\)가 되도록 유도해 보겠습니다.



$$
\begin{aligned}
I 
&= \int_{-\infty}^{\infty}e^{-z^2/2}dz \space \text{일 때,} \\[1em]
I^2 
&= \int_{-\infty}^{\infty}e^{-x^2/2}dx \int_{-\infty}^{\infty}e^{-y^2/2}dy \\[1em]
&= \int_{-\infty}^{\infty}\int_{-\infty}^{\infty}e^{-(x^2+y^2)/2}dxdy \space \Leftarrow \text{이중 적분 꼴로 치환} \\[1em]
&= \int_{\theta=0}^{2\pi}\int_{r=0}^{\infty}e^{-r^2/2}rdrd\theta \space \Leftarrow \text{좌표계를 극좌표계로 변환} \\[1em]
&= - \int_{\theta=0}^{2\pi}\int_{u=0}^{-\infty}e^udud\theta \space \Leftarrow -\cfrac{r^2}{2}=u \space \text{and} \space rdr=-du \\[1em]
&= - \int_{\theta=0}^{2\pi}\Big[e^u\Big]_{0}^{-\infty}d\theta \\[1em]
&= \int_{\theta=0}^{2\pi}d\theta \\[1em]
&= 2\pi \\[1em]
\therefore \space I &= \sqrt{2\pi} \space (\because I > 0)
\end{aligned}
$$



#### 1.2.3 기댓값

표준 정규 분포의 기댓값은 기함수(홀함수, odd function)의 성질을 이용하여 쉽게 도출 가능합니다. 기함수에 대한 내용은 [이 곳]()을 참고해주세요. 간단히 말하자면 기함수는 원점 대칭이기 때문에 $$\int_{-a}^{a}f(x)dx=0$$의 성질을 가지고 있습니다. 어떤 함수가 기함수라면, $$-f(x) = f(-x)$$를 만족합니다. 기댓값이 0인 이유는 표준 정규 분포의 기댓값 수식이 기함수의 성질 $$-f(x) = f(-x)$$를 만족하기 때문입니다.



$$
E(z) = \cfrac{1}{\sqrt{2\pi}}\int_{-\infty}^{\infty}ze^{-z^2/2}dz = 0
$$



#### 1.2.4 분산



$$
\begin{aligned}
Var(Z) 
&= E(Z^2) - E(Z)^2 \\[1em]
&= E(Z^2) - 0^2 \\[1em]
&= E(Z^2) \\[1em]
&= \cfrac{1}{\sqrt{2\pi}}\int_{-\infty}^{\infty} z^2e^{-z^2/2}dz \\[1em]
&= \cfrac{2}{\sqrt{2\pi}}\int_{0}^{\infty} z^2e^{-z^2/2}dz \space \Leftarrow \space \text{by even function's property}\\[1em] 
&= \cfrac{2}{\sqrt{2\pi}}\int_{0}^{\infty} zze^{-z^2/2}dz \space \Leftarrow \space \text{by parts}
\end{aligned}
$$

<br>

### 1.3 &nbsp; 정규 분포(Normal/Gaussian Distribution)

#### 1.3.1 정의

표준 정규 분포를 평균이 $$\mu$$, 분산이 $$\sigma^2$$이 되도록 location과 scale을 변화시킨 분포입니다.



$$
X \sim Normal(\mu, \sigma^2)
$$



참고로, $$X = \mu + \sigma z$$를 통해 표준 정규 분포에서 정규 분포로 변환할 수 있습니다. $$z=\cfrac{X-\mu}{\sigma}$$를 표준화(standardization)라고 합니다.



#### 1.3.2 PDF



$$
F_X(x) = P(X\le x) = P\bigg(\cfrac{X-\mu}{\sigma} \le \cfrac{x-\mu}{\sigma}\bigg) = \Phi(z) \space \text{일 때,} \\
$$


$$
\begin{aligned}
f_X(x) 
&= \Phi'(z) \\[1em]
&= \cfrac{\partial \Phi(z)}{\partial z} \cfrac{\partial z}{\partial x} \\[1em]
&= \text{표준 정규 분포의 pdf} \space \cdot \space \cfrac{1}{\sigma} \\[1em]
&= \cfrac{1}{\sigma\sqrt{2\pi}}e^{-\bigg(\cfrac{x-\mu}{\sigma}\bigg)^2/2}
\end{aligned}
$$



#### 1.3.3 기댓값



$$
E(X) = E(\mu + \sigma z) = \mu + \sigma \cdot0 = \mu
$$

#### 1.3.4 분산



$$
Var(X) = Var(\mu + \sigma z) = Var(\sigma z) = \sigma^2 Var(z) = \sigma^2
$$

<br>

### 1.4 &nbsp; 지수 분포(Exponential Distribution)

#### 1.4.1 정의

다음 사건아 발생하기까지 **대기 시간**에 대한 연속형 확률 분포입니다. 또는 포아송 분포와 연결 지어서 설명할 수 있는데, 포아송 분포에서 한 사건이 발생한 후 다음 사건이 발생하기까지의 **대기 시간**이라고도 해석합니다. 모수인 $$\lambda$$는 포아송 분포의 모수와 같은 의미로 생각하면 되겠습니다(단위 시간당 평균 사건 발생 횟수)



$$
X \sim E\text{x}po(\lambda)
$$



#### 1.4.2 PDF



$$
f_X(x) = \lambda e^{-\lambda x}
$$



#### 1.4.3 기댓값


$$
\begin{aligned}
\int_{0}^{\infty}x\lambda e^{-\lambda x} 
&= \Big[x(-e^{-\lambda x})\Big]_{0}^{\infty} + \int_{0}^{\infty}e^{-\lambda x} \space \Leftarrow \text{by parts} \\[1em] 
&= \int_{0}^{\infty}e^{-\lambda x} \\[1em] 
&= \Big[-\cfrac{1}{\lambda}e^{-\lambda x}\Big]_{0}^{\infty} \\[1em]
&= \cfrac{1}{\lambda}
\end{aligned}
$$

#### 1.4.4 분산



$$
\begin{align}
  Var(X) 
  &= E(X^2) - E(X)^2 \\[1em]
  &= \cfrac{2}{\lambda^2} - \bigg(\cfrac{1}{\lambda}\bigg)^2 \Leftarrow \space E(X^2) = \cfrac{2}{\lambda^2} \\[1em] 
  & = \cfrac{2-1}{\lambda^2} \\[1em] 
  &= \cfrac{1}{\lambda^2}
\end{align}
$$

<br>

### 1.5 &nbsp; 베타 분포(Beta Distribution)

#### 1.5.1 정의

확률 변수가 0과 1사이에서 정의되는 연속형 확률 분포입니다. 모수 $$a$$, $$b$$에의해 분포의 모양이 유연하게 변화할 수 있는 특성이 있습니다. 확률 변수의 범위가 0과 1사이라는 특성 때문에 0과 1사이에서 정의되는 parameter를 가지는 사전 확률 분포로 자주 활용됩니다.



$$
X \sim Beta(a. b)
$$


#### 1.5.2 PDF

베타 분포의 PDF는 다음과 같습니다. $$x$$의 범위가 0과 1사이에만 정의 됩니다.



$$
f_{X}(x) = 
  \begin{cases}
  \cfrac {1} {\beta(a, b)}x^{a-1}(1-x)^{b-1} &, \space 0 \lt x \lt1 \\[1em]
  0 &, \space \text{otherwhise}
  \end{cases}
$$



참고로 $$\beta(a,b)$$은 정규화 상수(normalizing constant)로서 적분시 값이 1이 되도록 보장하기 위해 필요한 값이며 *beta integral*을 통해서 도출됩니다.



$$
\beta(a,b) = \int_{0}^{1}x^{a-1}(1-x)^{b-1}dx
$$



정규화 상수의 값이 어떻게 도출되는지 궁금지 않으신가요? 정규화 상수는 감마 분포와의 관계를 통해서 도출이 가능합니다. 다음을 가정을 통해 정규화 상수를 유도해 봅시다.


$$
X \sim Gamma(a, 1)
$$

$$
Y \sim Gamma(b, 1)
$$

$$
T = X+Y
$$

$$
W = \cfrac{X}{X+Y}
$$

$$
\text{Suppose X, Y are independent}
$$



다음으로 $$T$$와 $$W$$의 결합 확률 분포 $$f_{T,W}(t,w)$$를 계산 해보겠습니다. 이때 확률 변수의 변환(transformation of random variable) 개념이 사용되었습니다. 확률변수 변환에 대한 자세한 내용은 [이 곳](https://analysisbugs.tistory.com/27)[^2]을 참조해 주시길 바랍니다.



$$
\begin{aligned}
  
  f_{T,W}(t,w) 
  &= f_{X,Y}(x,y)\Bigg\vert det\bigg(\cfrac{\partial(x, y)}{\partial(t, w)}\bigg)\Bigg\vert \\[1em]
  &= f_{X}(x)f_{Y}(y)\Bigg\vert det\bigg(\cfrac{\partial(x, y)}{\partial(t, w)}\bigg)\Bigg\vert \Leftarrow \space \text{independence of X and Y} \\[1em] 
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}x^{a}e^{-x}y^{b}e^{-y}\cfrac{1}{xy}\Bigg\vert det\bigg(\cfrac{\partial(x, y)}{\partial(t, w)}\bigg)\Bigg\vert \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}x^{a}e^{-x}y^{b}e^{-y}\cfrac{1}{xy}\Bigg\vert det\bigg(
  	\begin{bmatrix}
  	\partial x/\partial t & \partial x/\partial w \\
  	\partial y/\partial t & \partial y/\partial w \\
  	\end{bmatrix}
  \bigg)\Bigg\vert \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}x^{a}e^{-x}y^{b}e^{-y}\cfrac{1}{xy}\Bigg\vert det\bigg(
    \begin{bmatrix}
    w & t \\
    1-w & -t \\
    \end{bmatrix}
  \bigg)\Bigg\vert \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}x^{a}e^{-x}y^{b}e^{-y}\cfrac{1}{xy}\Bigg\vert -wt-t+tw \Bigg\vert \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}x^{a}e^{-x}y^{b}e^{-y}\cfrac{1}{xy}t \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}x^{a-1}e^{-x}y^{b-1}e^{-y}t \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}(tw)^{a-1}e^{-x}(t(1-w))^{b-1}e^{-y}t \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}w^{a-1}(1-w)^{b-1}e^{-(x+y)}t^{a+b-1} \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}w^{a-1}(1-w)^{b-1}e^{-t}t^{a+b}\cfrac{1}{t} \\[1em]
  &= \cfrac{1}{\Gamma(a) \Gamma(b)}w^{a-1}(1-w)^{b-1}\Gamma(a+b) \cfrac{1}{\Gamma(a+b)} e^{-t}t^{a+b}\cfrac{1}{t} \\[1em]
  &= \cfrac{\Gamma(a+b) }{\Gamma(a) \Gamma(b)}w^{a-1}(1-w)^{b-1}\cfrac{1}{\Gamma(a+b)} e^{-t}t^{a+b}\cfrac{1}{t} \\[1em]
  \end{aligned}
$$



위와 같이  결합 확률 분포 $$f_{T,W}(t,w)$$를 도출할 수 있습니다. 이제 거의 다 왔습니다!!  결론적으로 말하면, 주변 확률 분포인 $$f_{W}(w)$$를 유도하면 베타 분포의 정규화 상수와 감마 분포와의 관계성을 도출할 수 있습니다.


$$
\begin{aligned}
  f_{W}(w) 
  &= \int_{-\infty}^{\infty} f_{T,W}(t, w)dt \\[1em]
  &= \int_{-\infty}^{\infty} \cfrac{\Gamma(a+b) }{\Gamma(a) \Gamma(b)}w^{a-1}(1-w)^{b-1}\cfrac{1}{\Gamma(a+b)} e^{-t}t^{a+b}\cfrac{1}{t} \\[1em]
  &= \cfrac{\Gamma(a+b) }{\Gamma(a) \Gamma(b)} w^{a-1}(1-w)^{b-1}
  \end{aligned}
$$


> 참고로, 두번째 항에서 세번째 항으로 변화한 것은 $$\cfrac{1}{\Gamma(a+b)} e^{-t}t^{a+b}\cfrac{1}{t}$$ 가 $$Gamma(a+b, \space 1)$$의 **PDF**이기 때문입니다. 확률 분포 전체의 합은 1이 된다는 것을 상기해봅시다.

따라서 베타 분포의 정규화 상수 $$\cfrac {1} {\beta(a, b)}$$는 $$\cfrac{\Gamma(a+b) }{\Gamma(a) \Gamma(b)}$$이며, 베타 분포의 정규화 상수는 감마 분포의 비율과 연관되어 있다는 것을 유도할 수 있습니다.



#### 1.5.3 기댓값

감마 함수의 재귀 공식 $$\Gamma(a+1) = a \Gamma(a)$$, $$\Gamma(a) = \cfrac{\Gamma(a+1)}{a}$$을 이용하면 베타 분포의 기대값을 계산할 수 있습니다.



$$
\begin{align}
E(T) 
  &= \int_{0}^{1}t \cdot \cfrac{\Gamma(a+b)}{\Gamma(a) \Gamma(b)} t^{a-1}(1-t)^{b-1}dx \tag{1}\\[1em]
  &= \int_{0}^{1} \cfrac{\Gamma(a+b)}{\Gamma(a) \Gamma(b)} t^{a}(1-t)^{b-1}dx \tag{2} \\[1em]
  &= \int_{0}^{1} \cfrac{\cfrac{\Gamma(a+b+1)}{a+b}}{\cfrac{\Gamma(a+1)\Gamma(b)}{a}}t^{a}(1-t)^{b-1}dx \tag{3} \\[1em]
  &= \cfrac{a}{a+b} \int_{0}^{1} \cfrac{\Gamma(a+1+b)}{\Gamma(a+1)\Gamma(b)}t^a(1-t)^{b-1}dx \tag{4} \\[1em]
  &= \cfrac{a}{a+b} \tag{5}
\end{align}
$$



특히 식 $$(4)$$의 적분 부분은 $$Beta(a+1, \space b)$$의  PDF임을 눈치 채셨다면, 당연히 확률 분포의 적분값은 1이 됨을 알 수 있습니다.



#### 1.5.4 분산

분산도 기댓값과 같은 방식으로 감마 함수의 재귀 공식을 활용하여 도출할 수 있습니다. $$Var(X) = E(X^2) - E(X)^2$$ 공식을 활용하기 위해 $$E(X^2)$$을 먼저 구해보겠습니다.



$$
\begin{align}
	E(T^2) 
	&= \int_{0}^{\infty} t^2 \cfrac{\Gamma(a+b)}{\Gamma(a) \Gamma(b)} t^{a-1} (1-t)^{b-1}dt \tag{1} \\[1em]
	&= \cfrac{a(a+1)}{(a+b)(a+b+1)} \int_{0}^{\infty} \cfrac {\Gamma(a+b+2)} {\Gamma(a+2) \Gamma(b)} t^{a+1} (1-t)^{b-1} dt \tag{2} \\[1em]
	&= \cfrac{a(a+1)}{(a+b)(a+b+1)} \tag{3}
\end{align}
$$



식 $$(2)$$의 적분 부분은 $$Beta(a+2, \space b)$$의 PDF입니다. 확률 분포이므로 적분 값이 1임을 알 수 있습니다. 최종적으로 분산을 구해보겠습니다.



$$
\begin{align}
	Var(T)
	&= \cfrac {a(a+1)} {(a+b)(a+b+1)} - \cfrac{aa}{(a+b)(a+b)} \\[1em]
	&= \cfrac {a} {a+b} \cdot \cfrac{(a+1)(a+b) - a(a+b+1)}{(a+b)(a+b+1)} \\[1em]
	&= \cfrac {ab} {(a+b+1)(a+b)^2}
\end{align}
$$

<br>

### 1.6 &nbsp; 감마 분포(Gamma Distribution)

#### 1.6.1 정의

$$a$$번째 사건이 발생하기까지 대기시간에 대한 연속형 확률 분포를 나타냅니다. 참고로 감마 분포, 지수 분포, 포아송 분포는 포아송 과정(poisson process)에서 서로 연관이 되어 있는데, 관련 내용은 appendix를 참고해주세요.



$$
Y \sim Gamma(a, \lambda)
$$

#### 1.6.2 PDF

감마 분포는 *감마 함수*와 *표준 감마 분포* $$Gamma(a, 1)$$의 PDF를 구하면 손쉽게 도출 가능합니다. 먼저 감마 함수를 정의해 보겠습니다. 감마 함수는 다음과 같습니다.



$$
\begin{aligned}
\Gamma(a) 
&= \int_{0}^{\infty}x^{a-1}e^{-x}dx \\[1em]
&= \int_{0}^{\infty}x^{a}e^{-x}\cfrac{1}{x}dx

\end{aligned}
$$



이제 감마 함수의 양변을 $$\Gamma(a)$$로 나누어 봅시다. 이를 통해 우리는 $$\cfrac{1}{\Gamma(a)} x^{a-1} e^{-x}$$를 0부터 무한대까지 더했을 때의 값이 $$1$$ 사실을 유도할 수 있습니다. 모든 가능한 값들을 다 더해서 **1**...? 확률 분포의 적분값이 1이라는 사실을 통해,  우리는 $$\cfrac{1}{\Gamma(a)} x^{a-1} e^{-x}$$가 확률 분포임을 알 수 있습니다. 다시 말해 감마 함수에 약간의 조작을 가해서 표준 감마 분포 $$Gamma(a, 1)$$의 PDF($$x>0$$)가 도출 되었습니다. 



$$
\begin{aligned}
\cfrac{1}{\Gamma(a)} \Gamma(a) 
&= \int_{0}^{\infty}\cfrac{1}{\Gamma(a)} x^{a-1} e^{-x}dx \\[1em]
1 &= \int_{0}^{\infty}\cfrac{1}{\Gamma(a)} x^{a-1} e^{-x}dx \\[1em] 
\end{aligned}
$$



이제 표준 감마 분포를 일반화 하여 감마 분포를 도출해 봅시다. $$X \sim Gamma(a, 1)$$, $$Y=\cfrac{X}{\lambda}$$ 일때, $$Y$$의 PDF를 찾아봅시다.



$$
\begin{aligned}
f_{Y}(y) 
&= f_{X}(x)\cfrac{dx}{dy} \\[1em]
&= \cfrac{1}{\Gamma(a)}(\lambda y)^{a-1}e^{-\lambda y} \lambda \space \Leftarrow \space \because \space x=\lambda y \space \text{and} \space \lambda = \cfrac{dx}{dy} \\[1em]
&= \cfrac{\lambda^{a}}{\Gamma(a)}y^{a-1}e^{-\lambda y}
\end{aligned}
$$



#### 1.6.3 기댓값

감마 분포의 기댓값은 감마 함수의 재귀 공식 $$\Gamma(a+1) = a \Gamma(a)$$을 활용하면 쉽게 구할 수 있습니다. 먼저 $$X \sim Gamma(a, 1)$$일때, $$nth$$ moment를 구해보도록 하죠.


$$
\begin{aligned}
E(X^n) 
&= \cfrac{1}{\Gamma(a)}\int_{0}^{\infty} x^{n}\cdot x^{a} e^{-x} \cfrac{1}{x}dx \\[1em]
&= \cfrac{1}{\Gamma(a)}\int_{0}^{\infty} x^{a+n} e^{-x} \cfrac{1}{x} dx \\[1em]
&= \cfrac{\Gamma(a+n)}{\Gamma(a)} \space \Leftarrow \space \Gamma(a+n) = \int_{0}^{\infty} x^{a+n} e^{-x} \cfrac{1}{x} dx
\end{aligned}
$$



결국 확률 변수 $$X$$의 $$nth$$ moment는 감마 함수로 표현할 수 있게됨을 알 수 있습니다. 이제 기대값을 구해보죠.


$$
\begin{aligned}
E(X)
&= \cfrac{\Gamma(a+1)}{\Gamma(a)} \\[1em]
&= \cfrac{a\Gamma(a)}{\Gamma(a)} \\[1em]
&= a
\end{aligned}
$$


표준 감마 분포의 기대값은 $$a$$임을 알 수 있습니다. $$Y = \cfrac{1}{\lambda}$$를 이용하여 일반 감마 분포의 기대값을 도출해 봅시다.


$$
\begin{aligned}
E(Y) 
&= E\bigg(\cfrac{X}{\lambda}\bigg) \\
&= \cfrac{1}{\lambda}E(X) \\
&= \cfrac{a}{\lambda}
\end{aligned}
$$

#### 1.6.4 분산

분산도 역시 $$X \sim Gamma(a, 1)$$ 일때를 기준으로 $$X$$의 $$nth$$ moment를 구하여 도출할 수 있습니다. 아참, $$Var(X) = E(X^2) - E(X)^2$$임을 상기해주세요.


$$
\begin{aligned}
Var(X)
&= E(X^2) - E(X)^2 \\[1em]
&= \cfrac{\Gamma((a+1) +1)}{\Gamma(a)} - \bigg( \cfrac{\Gamma(a+1)}{\Gamma(a)} \bigg)^{2} \\[1em]
&= \cfrac{(a+1)\Gamma(a+1)}{\Gamma(a)} - \bigg( \cfrac{a\Gamma(a)}{\Gamma(a)} \bigg)^{2} \\[1em]
&= \cfrac{(a+1)a\Gamma(a)}{\Gamma(a)} - a^{2} \\[1em]
&= a^{2} +a - a^{2} \\[1em] 
&= a
\end{aligned}
$$



이상 표준 감마 분포의 분산을 도출 했습니다. 표준 감마 분포의 기대값과 마찬가지로 $$a$$임을 알 수 있습니다. 그럼 마지막으로 감마 분포의 분산을 계산해 보겠습니다.


$$
\begin{aligned}
Var(Y) 
&= E(Y^2) - E(Y)^2 \\[1em]
&= \cfrac{1}{\lambda^2}E(X^2) - \bigg(\cfrac{1}{\lambda}E(X) \bigg)^2 \\[1em]
&= \cfrac{a^2+a}{\lambda^2} - \bigg(\cfrac{a}{\lambda}\bigg)^2 \\[1em]
&= \cfrac{a}{\lambda^2}
\end{aligned}
$$

<br>

## 2 &nbsp; 부록: 포아송 과정(Poisson Process)[^3][^4]

---

지수 분포, 감마 분포, 포아송 분포가 관련되어 있는데 포아송 과정에 대해서 알아보겠습니다. 사실 포아송 과정을 이해하기 위해서는 먼저 확률 과정(stochastic process) 및 셈 과정(counting process)에 대한 개념이 있어야 합니다. 단어가 정말 어려운데... 하나하나 차근차근 살펴보죠. 먼저 과정(process)란 단어가 굉장히 어렵게 느껴집니다. 통계에서 활용되는 **과정**이라는 개념은 **시간**과 관련 있는 개념이라고 생각하시면 조금 쉬울 것 같습니다.



### 2.1 &nbsp; 확률 과정(stochastic process)

과정은 시간과 관련된 개념이라는 것은 알겠는데... 그래서 확률 과정(stochastic process)란 무슨 뜻일까요? 확률 과정이란 **순차적**으로 관측되는 **확률 변수**들의 **집합**을 의미합니다. 다시 말해 어느 시점 $$t$$에서의 값이 상수가 아니라 어떤 값을 확률적으로 가지는 확률 변수라는 의미입니다. 예를 들어 60분 동안 도착하는 이메일의 개수와 같이 연속 구간(보통 시간)에서 발생하는 불규칙적인 사건을 모델링 하는데 활용됩니다. 여기서 60분 동안 도착하는 이메일의 개수는 2개, 3개와 같이 고정된 상수 값이 아니라 어떤 분포를 따르는 확률 변수 입니다.



### 2.2 &nbsp; 셈 과정(counting process)

그렇다면 셈 과정(counting process)란 무엇일까요? 셈 과정이란 $$t$$시간 동안 발생하는 **사건의 수**를 나타내는 확률 과정을 뜻합니다. 위에서 언급한 60분 동안 도착하는 이메일 수, 1분 동안 은행에 방문하는 고객 수와 같은 것들이 셈 과정의 대표적인 예시라고 하겠습니다. 셈 과정은 확률 과정이므로 확률 변수들의 **집합** 입니다. 따라서 셈 과정은 아래와 같이 나타낼 수 있습니다.



$$
\{N(t), \space t \ge0\} \\[1em]
$$

$$
N(t) \space \text{represents the number of events that have occured through time} \space t
$$



### 2.3 &nbsp; 포아송 과정(poisson process)

마지막으로 포아송 과정(poisson process)는 무엇일까요? 포아송 과정은 아래의 조건을 따르는 셈 과정을 뜻합니다. 요악하자면, $$t$$ 시점까지의 사건 발생 횟수가 포아송 분포를 따르는 셈과정이라고 할 수 있겠습니다.



(1) $$N(0)=0$$

(2) 모든 $$t$$에 대해 $$N(t) \sim \text{Pois}(\lambda), P(N(t)=n)=\cfrac{(\lambda t)^{n}e^{-\lambda t}}{n!}$$

(3) (독립 증분) $$0 \le q \le r \le s \le t$$인 $$q,r,s,t$$에 대해 $$N(t)-N(s)$$와 $$N(r)-N(q)$$는 독립인 확률 변수

(4) (정상 증분) 모든 $$s, t \ge 0$$에 대해 $$N(t+s)-N(s)$$는 $$N(t)$$와 같은 분포

독립 증분(independent increment)이란 서로 다른 시간 구간에서 발생하는 사건의 수가 서로 연관이 없다는 성질을 말하며, 정상 증분(stationary increment)란 두 시점 사이에 발생하는 사건의 수가 두 시점간의 시간의 폭에만 의존하는 성질을 의미합니다.

<br>

## 참고자료

---

[^1]: [공돌이의 수학 정리 노트: 가우스 적분](https://angeloyeo.github.io/2020/01/17/Gaussian_Integral.html)
[^2]: [분석 벌레의 공부방: 확률변수들의 함수](https://analysisbugs.tistory.com/27)
[^3]: [Seoncheol Park: 포아송과정(Poisson process)](https://seoncheolpark.github.io/book/_book/9-5-poisson-process.html)
[^4]: [종혁의 저장소: [확률 과정] 푸아송 과정](https://mons1220.tistory.com/67)



