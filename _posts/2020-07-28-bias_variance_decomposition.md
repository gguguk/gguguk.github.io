---
title: Bias Variance Decomposition
author: Gukwon Koo
categories: [Math, Statistics]
tags: [bias, variance, decomposition]
pin: false
math: true
comments: true
---
> 머신러닝의 학습의 판단 기준이 되는 error를 bias, variance, irreducible error의 관점에서 분해해보고 좋은 품질의 데이터가 필요한 이유와 bias 및 variance의 trade-off가 발생할 수밖에 없는 이유를 알아보겠습니다.

## 1 &nbsp; Prerequsite to start with

- 통계학에서의 추정은 모집단을 설명하는 알 수 없는 미지의 **모수**를 추정하고자 하는 것입니다.
- 그러나 우리는 모집단의 분포와 그 분포를 형성시키는 모수를 알 수 없습니다.
- 우리가 얻을 수 있는 데이터는 특정 분포를 따르는 모집단에서 데이터를 샘플링 한 것으로 생각해야 합니다.
- 따라서 전통적인 통계학 기법이든, 통계학에 기반을 둔 머신러닝 기법이든 추정을 하기 위해 활용하는 데이터는 모두 **표본**입니다.

<br>

## 2 &nbsp; Problem setting

- 우리는 궁극적으로 모집단의 데이터에 적합시킨(fit) regressor를 추정하고자 합니다.

$$
Y=f(X)+\epsilon
$$

- $$\epsilon$$(random gaussian noise)은 평균 0, 분산 $$\sigma^2$$인 가우시안 분포

$$
E(\epsilon) = 0
$$

$$
Var(\epsilon)=\sigma^2
$$

- 모집단의 분포에서 크기(size) T만큼의 표본을 무한번 복원 추출을 하여 표본 $$X^{(i)}$$, $$Y^{(i)}$$를 뽑습니다.

$$
\begin{aligned}
X_{i} &= (x^{(i)}_{1}, \space x^{(i)}_{2}, \space ..., \space x^{(i)}_{T})\\
Y_{i} &= (y^{(i)}_{1}, \space y^{(i)}_{2}, \space ..., \space y^{(i)}_{T})
\end{aligned}
$$

- 각 표본으로 각자의 데이터를 통해 regressor를 fitting 하여 class of models인 estimator를 구합니다.

$$
estimator = \hat{f}(X|\theta)
$$

- estimator의 각 model은 자신들이 **적합된 표본 데이터에 따라 각기 다른 $$\theta$$값을 가지게 됩니다.** 따라서 이후에 살펴볼 bias 수식에 기대값이 있게 됩니다.
- **이후의 수식은 training시 활용한 표본 데이터가 아니라 모집단 분포에서 새로 추출한 test data에 대한 수식임을 반드시 명심합시다!!**(train data와 test data는 결국 같은 분포에 샘플링 된 것임)

<br>

## 3 &nbsp; Proof of  Bias-Variance Decomposition

먼저, 수식의 단순화를 위해 estimator를 $$\hat{f}(X|\theta))$$에서 $$\theta$$를 삭제한 $$\hat{f}(X)$$로 표기하겠습니다.

$$ \begin{aligned}
MSE(\hat{f}(X)) 
&= E[(Y-\hat{f}(X))^2] \qquad (1)\\
&= E[(f(X) + \epsilon - \hat{f}(X))^2] \qquad (2) \\
&= E[(f(X) - \hat{f}(X)) + \epsilon)^2] \qquad (3)\\
&= E[(f(X)-\hat{f}(X))^2 + \epsilon^2 + 2(f(X)-\hat{f}(X))\epsilon] \qquad (4)\\
&= E[(f(X)-\hat{f}(X))^2] + E[\epsilon^2] + E[2(f(X)-\hat{f}(X))\epsilon] \qquad (5)\\
&= E[(f(X)-\hat{f}(X))^2] + E[\epsilon^2] + E[2(f(X)-\hat{f}(X))]E[\epsilon] \qquad (6)\\
&= E[(f(X)-\hat{f}(X))^2] + E[\epsilon^2] \qquad (7) \\
\end{aligned} $$

- $$(2)$$는 $$Y=f(X)+\epsilon$$으로 도출됩니다. 
- $$(3), (4)$$는 square expansion인 $$(a+b)^2 = a^2+b^2+2ab$$를 통해 도출됩니다.
- $$(5)$$는 기대값의 linear property를 이용한 것입니다. 
- $$(6)$$은 확률변수 $$\epsilon$$와 $$\hat{f}$$의 독립성을 이용한 것입니다. 두 확률변수가 독립일 때, 두 확률변수의 곱의 기대값은 각각의 기대값의 곱으로 분리될 수 있다는 사실을 떠올린다면 쉽게 도출할 수 있습니다. 다음으로 위 단락에서 도출한 마지막 항을 이어서 전개해 보겠습니다.

$$
\begin{aligned}
MSE(\hat{f}(X)) 
&= E[(f(X)−E[\hat{f}(X)]+E[\hat{f}(X)]− \hat{f}(X))^2] + E[\epsilon^2] \qquad (8) \\
&= E[(f(X)−E[\hat{f}(X)])+(E[\hat{f}(X)]− \hat{f}(X)))^2] + E[\epsilon^2] \qquad (9) \\
&= E[(f(X)−E[\hat{f}(X)])^2+(E[\hat{f}(X)]− \hat{f}(X))^2 + 2(f(X)−E[\hat{f}(X)])(E[\hat{f}(X)]− \hat{f}(X))] + E[\epsilon^2] \qquad (10) \\
&= E[(f(X)−E[\hat{f}(X)])^2]+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + E[2(f(X)−E[\hat{f}(X)])(E[\hat{f}(X)]− \hat{f}(X))] + E[\epsilon^2] \qquad (11) \\
&= (f(X)−E[\hat{f}(X)])^2+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + E[2(f(X)−E[\hat{f}(X)])(E[\hat{f}(X)]− \hat{f}(X))] + E[\epsilon^2] \qquad (12) \\
&= (f(X)−E[\hat{f}(X)])^2+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + 2(f(X)−E[\hat{f}(X)])(E[E[\hat{f}(X)]− \hat{f}(X)]) + E[\epsilon^2] \qquad (13) \\
&= (f(X)−E[\hat{f}(X)])^2+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + 2(f(X)−E[\hat{f}(X)])(E[\hat{f}(X)]− E[\hat{f}(X)]) + E[\epsilon^2] \qquad (14) \\
&= (f(X)−E[\hat{f}(X)])^2+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + E[\epsilon^2] \qquad (15) \\
&= (f(X)−E[\hat{f}(X)])^2+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + E[(\epsilon − 0)^2] \qquad (16) \\
&= (f(X)−E[\hat{f}(X)])^2+E[(E[\hat{f}(X)]− \hat{f}(X))^2] + E[(\epsilon − E[\epsilon])^2] \qquad (17) \\
&= Bias[\hat{f}(X)]^2 + Var[\hat{f}(X)] + \sigma^{2}_{\epsilon} \qquad (18) 
\end{aligned}
$$

- $$(8)$$은 식의 편리한 유도를 위해 $$E[\hat{f}(X)]$$를 빼고 더한 것입니다. 원래의 식과 완벽하게 동일합니다. 
- $$(9), (10)$$은 또 다시 square expansion을 활용하였습니다.
- $$(11)$$은 기대값의 linear property를 활용한 것입니다. 
- $$(12)$$는 $$E[c] = c$$($$c$$ is not r.v but constant)를 활용한 것입니다. 모수인 $$f(X)$$는 정해진 상수이며, 확률변수 $$\hat{f}(X)$$의 기대값인 $$E[\hat{f}(X)]$$는 **상수**입니다. $$\hat{f}(X)$$의 값은 $$X$$의 realization(observation)에 따라서 그 값이 변하는 확률변수이지만, 해당 확률변수의 기대값은 고정된 상수임을 반드시 기억해야하겠습니다.
- $$(13)$$은 $$(12)$$와 같은 이유로 기대값 formula에서 상수항을 밖으로 꺼낸 것입니다.
- $$(14)$$는 $$E[E[X]] = E[X]$$ 성질과 linear 성질을 활용하였습니다. 그 결과 3번째 항의 경우 값이 $$0$$이 됩니다.
- $$(15), (16), (17)$$은 $$E[\epsilon] = 0$$을 이용해 유도하였습니다. 또한 분산의 공식에 의해 마지막 항을 $$\sigma^{2}_{\epsilon}$$으로 치환할 수 있습니다.

<br>

**결과적으로 추정식의 오차는 $$Bias^2$$, $$Variance$$, $$irreducible \space error$$로 decomposition되는 것을 확인할 수 있습니다.** irreducible error는 절대로 줄일 수 없는 내재적인 오류를 뜻합니다.

<br>

### 2.1 &nbsp; Meaning of Bias
decomposition을 통해 이끌어 낸 bias의 의미

### 2.2 &nbsp; Meaning of Variance
decomposition을 통해 이끌어낸 variance의 의미

<br>

## 4 &nbsp; Bias-Variance Trade-off

코드를 통해서 시각화

## Reference
- [The Bias-Variance Tradeoff](https://towardsdatascience.com/the-bias-variance-tradeoff-8818f41e39e9)
- [The Bias-Variance trade-off : Explanation and Demo](https://towardsdatascience.com/the-bias-variance-trade-off-explanation-and-demo-8f462f8d6326)
- [계량경제학 입문](http://www.sigmapress.co.kr/shop/shop_image/g28592_1405384604.pdf)
- [Variance Reduction and Ensemble Methods](https://personal.utdallas.edu/~nrr150130/cs7301/2016fa/lects/Lecture_10_Ensemble.pdf)
- [MS&E 226: Fundamentals of Data Science
Lecture 6: Bias and variance](https://web.stanford.edu/class/msande226/lecture6_prediction.pdf)
- [위키피디아-편향-분산 트레이드오프](https://ko.wikipedia.org/wiki/%ED%8E%B8%ED%96%A5-%EB%B6%84%EC%82%B0_%ED%8A%B8%EB%A0%88%EC%9D%B4%EB%93%9C%EC%98%A4%ED%94%84)
- [[머신 러닝] 편향-분산 트레이드오프 (Bias-Variance Tradeoff)](https://untitledtblog.tistory.com/143)
- [Estimating Parameters from Simple Random Samples](https://www.stat.berkeley.edu/~stark/SticiGui/Text/estimation.htm)
- [Why is an estimator considered a random variable?](https://stats.stackexchange.com/questions/317541/why-is-an-estimator-considered-a-random-variable)
- [Bias-Variance Analysis: Theory and Practice - CS229](http://cs229.stanford.edu/summer2019/BiasVarianceAnalysis.pdf)