---
title: 미분 공식 증명
author: Gukwon Koo
categories: [Math, Calculus]
tags: [derivative]
pin: false
math: true
comments: true
---
> 미분 공식을 이해하고 내면화하기 위해 대표적인 미분 공식 증명을 정리해 보겠습니다.


## 1 &nbsp;  $$f(x)=x^{n}$$일때, $$f'(x)=nx^{n-1}$$

$$
\begin{aligned}
f'(x) 
&= \lim_{h \to 0} \cfrac {f(x+h) - f(x)} {h} \\
&= \lim_{h \to 0} \cfrac {(x+h)^n - x^n} {h} \\
&= \lim_{h \to 0} \cfrac {(x+h-x)((x+h)^{n-1}x^{0} + (x+h)^{n-2}x^{1} + ... + (x+h)^{0}x^{n-1})} {h} \\
&= \lim_{h \to 0} {((x+h)^{n-1}x^{0} + (x+h)^{n-2}x^{1} + ... + (x+h)^{0}x^{n-1})} \\
&= x^{n-1}x^{0} + x^{n-2}x^{1} + ...+ x^{0}x^{n-1} \\
&= x^{n-1} + x^{n-1} + ... +x^{n-1} \\
&= nx^{n-1}
\end{aligned}
$$

참고) $$a^n - b^n = (a-b)(a^{n-1}b^{0} + a^{n-2}b^{1} +  ... + a^{0}b^{n-1})$$


## 2 &nbsp; 자연상수 $$e$$의 정의
$$e=\lim_{n \to\infty}\bigg(1+\cfrac{1}{n}\bigg)^n=\lim_{x\to0}\big(1+x\big)^{1/n}$$

참고) 마지막항은 $$x = \cfrac{1}{n}$$을 이용한 것


## 3 &nbsp; $$\lim_{x\to0}\cfrac{e^x-1}{x}=1$$

$$
\begin{aligned}
\lim_{x\to0}\cfrac{e^x-1}{x}
&= \lim_{t\to0}\cfrac{t}{ln(1+t)} \Leftarrow e^x-1=t \\
&= \lim_{t\to0}\cfrac{1}{\cfrac{ln(1+t)}{t}} \\
&= \lim_{t\to0}\cfrac{1}{\frac{1}{t} ln(1+t)} \\
&= \lim_{t\to0}\cfrac{1}{ln(1+t)^{\frac{1}{t}}} \\
&= \cfrac{1}{ln(\lim_{t\to0}(1+t)^{\frac{1}{t}})} \\
&= \cfrac{1}{lne} \Leftarrow \lim_{x\to0}\big(1+x\big)^{1/n} \\
&= 1
\end{aligned}
$$

## 4 &nbsp; 지수함수  $$f(x) =e^x$$일 때, $$f'(x)=e^x$$

$$
\begin{aligned}
f'(x) 
&= \lim_{h \to 0} \cfrac {f(x+h) - f(x)} {h} \\
&= \lim_{h \to 0} \cfrac {e^{x+h} - e^{x}} {h} \\
&= \lim_{h \to 0} \cfrac {e^{x}(e^{h}-1)} {h} \\
&= e^{x} \lim_{h \to 0} \cfrac {e^{h}-1} {h} \\
&= e^{x} \Leftarrow \lim_{x\to0} \cfrac{e^x-1}{x}=1 \\
\end{aligned}
$$

## 4 &nbsp; 지수함수  $$f(x) =a^x$$일 때, $$f'(x)=e^{xlna}\cdot lna$$

다음 수식을 먼저 정의 하겠습니다.

$$
\begin{aligned}
a^x 
&= a^{log_{e}e^{x}} \\
&= (e^{x})^{log_{e}a} \Leftarrow a^{log_{c}b} = b^{log_{c}a} \\
&= e^{xlog_{e}a} \\
&= e^{xlna} \\
\end{aligned}
$$

위의 수식을 이용하여 증명을 해보겠습니다.

$$
\begin{aligned}
f'(x) 
&= \cfrac {da^x} {dx} \\
&= \cfrac {d} {dx} e^{xlna} \\
&= \cfrac {d} {dx} e^t \Leftarrow t=xlna \\
&= \cfrac {de^t} {dt} \cfrac {dt} {dx} \\
&= e^t \cfrac {dt} {dx} \\
&= e^t \cdot lna \\
&= e^{xlna} \cdot lna \\
\end{aligned}
$$

## 5 &nbsp; 로그함수  $$f(x) =lnx$$ 일 때, $$f'(x)=\cfrac{1}{x}$$

$$
\begin{aligned}
f'(x) 
&= \lim_{h \to 0} \cfrac {f(x+h) - f(x)} {h} \\
&= \lim_{h \to 0} \cfrac {ln(x+h) - lnx} {h} \\
&= \lim_{h \to 0} \cfrac {1} {h} \cdot {ln \cfrac {x+h} {x}} \\
&= \lim_{h \to 0} ln\bigg(\cfrac{x+h}{x}\bigg)^{\frac{1}{h}} \\
&= \lim_{h \to 0} ln\bigg(\cfrac{x+h}{x}\bigg)^{\frac{x}{h} \cdot \frac{1}{x}} \\
&= \lim_{h \to 0} \cfrac{1}{x} \cdot ln \bigg(\cfrac{x+h}{x}\bigg)^{\frac{x}{h}} \\
&= \lim_{h \to 0} \cfrac{1}{x} \cdot ln \bigg(1+\cfrac{h}{x}\bigg)^{\frac{x}{h}} \\
&= \cfrac{1}{x} \cdot ln \lim_{h \to 0}  \bigg(1+\cfrac{h}{x}\bigg)^{\frac{x}{h}} \\
&= \cfrac{1}{x} \cdot ln \lim_{h \to 0}  \bigg(1+\cfrac{h}{x}\bigg)^{\frac{x}{h}} \\
&= \cfrac{1}{x} \cdot lne \Leftarrow e=\lim_{x\to0}(1+x)^{1/x} \\
&= \cfrac {1} {x} 
\end{aligned}
$$