---
title: Taylor Series
author: Gukwon Koo
categories: [Math, Calculus]
tags: [taylor series]
pin: false
math: true
comments: true
---
## 1 &nbsp; Taylor Series
$$x=a$$에서 무한번 미분 가능한 함수 $$f(x)$$를 근사 다항식으로 표현한 것을 **테일러 급수**(taylor series, taylor expansion)라고 합니다.

$$
\begin{aligned}
f(x)
&= \cfrac{f(a)(x-1)^0}{0!} + \cfrac{f^{(1)}(a)(x-a)^1}{1!} + \cfrac{f^{(2)}(a)(x-a)^{2}}{2!} + ... \\
&= \sum^{\infty}_{n=0} \cfrac {f^{(n)}(a)}{n!}{(x-a)}^{n}
\end{aligned}
$$

이를 다르게 표현하면, $$f(x)$$를 $$x=a$$에서 동일한 미분계수(derivative)를 가지는 다항함수로 근사 시키는 것이라고 할 수 있겠습니다.

## 2 &nbsp; 자연상수 $$e$$

테일러 급수를 통해 자연상수 $$e$$를 다항함수로 표현 할 수 있습니다. 자연상수 $$e$$의 정의는 다음과 같습니다.

$$
e=\lim\limits_{n\rightarrow\infty}{\Big(1+\cfrac{1}{n}\Big)}^{n} =\lim\limits_{n\rightarrow0}{\Big(1+n\Big)^{1/n}}
$$

그러나 n이 무한대로 커지면 연산에 필요한 cost가 가중되게 됩니다. 다른 형태의 식으로 2.718281828... 을 근사할 수 없을까요? 테일러 급수를 이용해 봅시다!

$$
\begin{aligned}
f(x)
&= e^{x} \\ 
&= \sum_{n=0}^{\infty}\cfrac{e^a}{n!}(x-a)^n
\end{aligned}
$$

위 식은 $$e^x$$를 무한번 미분해도 $$e^x$$임을 고려하면 쉽게 도출 할 수 있습니다.

위 식에서 $$a=0$$이라면,

$$
\begin{aligned}
f(x)
&= e^{x} \\ 
&= \sum_{n=0}^{\infty}\cfrac{1}{n!}x^n \\
&= 1+\cfrac{1}{1!}x+\cfrac{1}{2!}x^2+\cfrac{1}{3!}x^3+...
\end{aligned}
$$

$$x=1$$을 대입하면,

$$
\begin{aligned}
f(1)
&= e \\ 
&= \sum_{n=0}^{\infty}\cfrac{1}{n!} \\
&= 1+\cfrac{1}{1!}+\cfrac{1}{2!}+\cfrac{1}{3!}+...
\end{aligned}
$$

$$x=-1$$을 대입하면,

$$
\begin{aligned}
f(-1)
&= e^{-1} \\ 
&= \sum_{n=0}^{\infty}\cfrac{(-1)^{n}}{n!} \\
&= 1-\cfrac{1}{1!}+\cfrac{1}{2!}-\cfrac{1}{3!}+...
\end{aligned}
$$


## 3 &nbsp; 매클로린 급수
매클로린 급수(maclaurin series)란 talyor seires에서 $$a=0$$인 series를 뜻합니다. 위에서 $$e^x$$를 테일러 시리즈로 근사할 때 $$a=0$$으로 하였는데, 이는 매클로린 급수를 의미한다고 하겠습니다.