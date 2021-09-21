---
title: 고유값 분해(Eigenvalue Decompostion)
author: Gukwon Koo
categories: [Math, Linear algebra]
tags: [evd]
pin: false
math: true
comments: true
date: 2021-01-26 22:18:00 +0900
---

![eigenvalue decomposition](https://www.oreilly.com/library/view/mastering-numerical-computing/9781788993357/assets/cb9a2709-fbfc-4fc0-be93-10fcbf54b757.png){:width='600'}_정방행렬을 고유벡터와 고유값의 행렬로 분해할 수 있다?! (Eigenvalue decomposition, OREILLY)_

<br>

### 2줄 요약

---

- 고유값 분해는 $$ n \times n $$ 정방 행렬을 고유벡터로 이루어진 행렬과 고유값으로 이루어진 행렬로 대각화 하는 방법
- 행렬 거듭제곱, SVD 등에서 활용 됨


---

<br>

$$n \times n$$ 정방 행렬을 고유벡터와 고유값으로 대각화 할 수 있는 고유값 분해에 대해 알아보겠습니다. 고유값 분해를 통해서 행렬 거듭제곱꼴이나 행렬식 계산을 단순화할 수 있습니다. 또한 SVD를 이해하기 위한 선수지식으로서도 의미가 있겠습니다. 본 글을 이해하기 위해서 아래의 개념에 대한 이해가 필요합니다.

- 고유값과 고유벡터
- 대칭 행렬(symmetric matrix)
- 직교 행렬(orthogonal matrix)

또한 행렬과 벡터의 표기는 아래를 따릅니다.

- 행렬: $$\boldsymbol{X}$$
- 벡터: $$\boldsymbol{x}$$
- 벡터의 원소: $$x_1, x_2, \cdots, x_i$$

<br>

# 1 &nbsp; 고유값 분해

---

## 1.1 &nbsp; 고유값 분해란?

고유값 분해(eigen value decomposition)란 $$n \times n$$ **정방 행렬**을 자신의 **고유벡터**를 열벡터로 가진 행렬 $$\boldsymbol{V}$$와 **고유값**으로 이루어진 행렬 $$\boldsymbol{\Lambda}$$로 대각화 분해하는 기법입니다.

<br>

$$ \boldsymbol{X} = \boldsymbol{V} \boldsymbol{\Lambda} \boldsymbol{V}^{-1}$$

<br>

## 1.2 &nbsp; 고유값 분해 수식 유도

고유값 분해의 수식 유도는 고유값과 고유벡터의 개념과 행렬의 각 열의 상수를 인수 분해 방법을 알면 쉽게 유도할 수 있습니다. 고유값과 고유벡터는 [이글](https://gguguk.github.io/posts/eigenvalue_eigenvector/)을 참고해주시구요. 상수 인수 분해 방법에 대해서 알아 보겠습니다.

#### 1.2.1 &nbsp; 행렬의 상수의 인수 분해

$$n \times n$$ 정방 행렬 $$\boldsymbol{X}$$가 다음의 $$n$$개의 상수 $$\lambda_{1,\cdots,n}$$과 벡터 $$\boldsymbol{v}_{1,\cdots,n}$$ 로 이루어져 있다고 생각해보죠.

<br>

$$ \boldsymbol{X} = 
\begin{bmatrix} 
	| & | &  & |\\ 
	\lambda_1\boldsymbol{v}_1 & \lambda_2\boldsymbol{v}_2 & \cdots & \lambda_n\boldsymbol{v}_n\\ 
    | & | &  &|\end{bmatrix} $$ 

<br>

이 행렬 $$\boldsymbol{X}$$의 상수 $$\lambda_{1, \cdots, n}$$를 빼내서 행렬의 곱으로 표현하면 아래와 같이 쓸 수 있습니다. 적당한 숫자를 대입해서 계산해보시면 좌항과 우항의 연산 결과가 같다는 것을 알 수 있습니다.

<br>

$$ \boldsymbol{X} = 
\begin{bmatrix} | & | &  & |\\ \lambda_1\boldsymbol{v}_1 & \lambda_2\boldsymbol{v}_2 & \cdots & \lambda_n\boldsymbol{v}_n\\     | & | &  &|
\end{bmatrix}=
\begin{bmatrix} 
	\vert & | &  & |\\
	\boldsymbol{v}_1 & \boldsymbol{v}_2 & \ldots & \boldsymbol{v}_n\\ 
    \vert & \vert &  &\vert
\end{bmatrix}
\begin{bmatrix} 
	\lambda_1 & \ldots  & 0 \\ 
   \vdots & \ddots & \vdots \\
   0 & \ldots    &\lambda_n
\end{bmatrix} $$

<br>

#### 1.2.2 &nbsp; 고유값과 고유벡터를 행렬 notation으로 표시

$$n \times n$$ 정방 행렬 $$\boldsymbol{X}$$는 최대 $$n$$개의 고유값과 고유벡터를 가질 수 있습니다.

<br>

$$\boldsymbol{X} \boldsymbol{v}_1 = \lambda_1 \boldsymbol{v}_1$$

$$\boldsymbol{X} \boldsymbol{v}_2 = \lambda_2 \boldsymbol{v}_2$$

$$\vdots$$

$$\boldsymbol{X} \boldsymbol{v}_n = \lambda_n \boldsymbol{v}_n$$

<br>

여러개의 고유값과 고유벡터를 행렬 하나에 표현해볼까요? 먼저 고유벡터들을 열벡터로 하는 $$n \times n$$ 정방 행렬을 $$\boldsymbol{V}$$라고 정의합시다.

<br>

$$ \boldsymbol{X} \boldsymbol{V} = \boldsymbol{X}
\begin{bmatrix} 
	| & | &  & |\\ 
	\boldsymbol{v}_1 & \boldsymbol{v}_2 & \cdots & \boldsymbol{v}_n \\ 
    | & | &  &|\end{bmatrix} = \begin{bmatrix} 
	| & | &  & |\\ 
	\boldsymbol{X}\boldsymbol{v}_1 & \boldsymbol{X}\boldsymbol{v}_2 & \cdots & \boldsymbol{X}\boldsymbol{v}_n \\ 
    | & | &  &|\end{bmatrix} = \begin{bmatrix} 
	| & | &  & |\\ 
	\lambda_1\boldsymbol{v}_1 & \lambda_2\boldsymbol{v}_2 & \cdots & \lambda_n\boldsymbol{v}_n \\ 
    | & | &  &|\end{bmatrix} $$

<br>

$$\Lambda$$를 고유값들을 대각성분으로 하는 $$n \times n $$ 정방행렬이라고 하면, 인수 분해를 적용하여 $$\boldsymbol{X} \boldsymbol{V}$$를 다음과 같이 나타낼 수 있습니다. 

<br>

$$ \begin{align}\boldsymbol{X} \boldsymbol{V} &= \begin{bmatrix} 
	| & | &  & |\\ 
	\lambda_1\boldsymbol{v}_1 & \lambda_2\boldsymbol{v}_2 & \cdots & \lambda_n\boldsymbol{v}_n \\ 
    | & | &  &|\end{bmatrix} = \begin{bmatrix} 
	| & | &  & |\\ 
	\boldsymbol{v}_1 & \boldsymbol{v}_2 & \cdots & \boldsymbol{v}_n \\ 
    | & | &  &|\end{bmatrix} \begin{bmatrix} 
	\lambda_1 & \ldots  & 0 \\ 
   \vdots & \ddots & \vdots \\
   0 & \ldots    &\lambda_n
\end{bmatrix} \\[1em] &= \boldsymbol{V}\boldsymbol{\Lambda} \end{align}$$

<br>

행렬 $$\boldsymbol{X}$$가 고유값 분해 가능하다면, 다시 말해 아래의 두 조건을 만족한다면 (이와 관련된 조금 더 상세한 내용은 Appendix를 참조해주세요.)

<br>

- 행렬 $$\boldsymbol{X}$$는  $$n \times n$$ **정방 행렬**입니다.
- 행렬 $$\boldsymbol{X}$$는 **$$n$$개의 선형 독립(linearly independent)[^1] 고유벡터**를 가집니다.

<br>

행렬 $$\boldsymbol{X}$$는 최종적으로 다음과 같이 고유벡터와 고유값들로 이루어진 행렬로 분해 가능합니다.

<br>

$$\boldsymbol{X} = \boldsymbol{V}\boldsymbol{\Lambda}\boldsymbol{V}^{-1}$$

<br>

# 2 &nbsp; 고유값 분해를 손으로 해보자

---

행렬 $$\boldsymbol{A}=\left[\begin{matrix} 2 & 1 \\ 1 & 2 \end{matrix} \right]$$를 고유값 분해를 해보겠습니다. 고유값과 고유벡터를 구하는 방법은 [여기](https://gguguk.github.io/posts/eigenvalue_eigenvector/)을 참조해 주시구요. 행렬 $$\boldsymbol{A}$$의 고유값과 고유벡터는 다음과 같습니다.

<br>

$$\lambda_1 =1, \space \boldsymbol{v}_1 = \left[\begin{matrix} 1/\sqrt{2} \\ -1/\sqrt{2} \end{matrix} \right], \space \lambda_2 = 3, \space \boldsymbol{v}_2=\left[\begin{matrix} 1/\sqrt{2} \\ 1/\sqrt{2} \end{matrix} \right]$$

<br>

고유값 분해를 활용하면 행렬 $$\boldsymbol{A}$$를 다음과 같이 분해할 수 있습니다. $$\boldsymbol{V}$$는 $$\boldsymbol{A}$$의 고유벡터를 열벡터로 하는 정방 행렬이고 $$\boldsymbol{\Lambda}$$는 $$\boldsymbol{V}$$의 고유값들을 대각 성분으로하는 대각 행렬입니다.

<br>

$$ \begin{align}
\boldsymbol{A}
&= \boldsymbol{V}\boldsymbol{\Lambda}\boldsymbol{V}^{-1} \\[1em]
&= \begin{bmatrix} 1/\sqrt{2} & 1/\sqrt{2} \\ 1/\sqrt{2} & -1/\sqrt{2} \end{bmatrix} \begin{bmatrix} 3 \space & 0 \\[0.5em] 0 \space & 1\end{bmatrix} \begin{bmatrix} 1/\sqrt{2} & 1/\sqrt{2} \\ 1/\sqrt{2} & -1/\sqrt{2} \end{bmatrix} 
\end{align} $$

<br>

원래 행렬 $$\boldsymbol{A}$$와 정말로 같은 값인지 검증 해볼까요?

<br>

$$ \begin{align}
\boldsymbol{A}
&= \begin{bmatrix} 1/\sqrt{2} & 1/\sqrt{2} \\ 1/\sqrt{2} & -1/\sqrt{2} \end{bmatrix} \begin{bmatrix}  3 \space & 0 \\[0.5em] 0 \space & 1\end{bmatrix} \begin{bmatrix} 1/\sqrt{2} & 1/\sqrt{2} \\ 1/\sqrt{2} & -1/\sqrt{2} \end{bmatrix} \\[1em]
&= \begin{bmatrix} 3/2+1/2 & 3/2-1/2 \\ 3/2-1/2 & 3/2+1/2 \end{bmatrix} \\[1em]
&= \begin{bmatrix} 2 & 1 \\ 1 & 2 \end{bmatrix}
\end{align} $$

<br>

# 3 &nbsp; 고유값 분해의 활용

---

## 3.1 &nbsp; 행렬의 거듭제곱

임의의 행렬 $$ \boldsymbol{X} $$에 대해서 $$ \boldsymbol{X}^2, \boldsymbol{X}^3 $$ 정도는 우리가 행렬 곱셈을 통해서 손으로도 손쉽게 계산 할 수 있습니다. 그렇다면 $$\boldsymbol{X}^{1000}$$은 어떤가요? 물론 계산할 수도 있겠지만 매우 번거롭습니다. 하지만 고유값 분해를 적용하면 행렬의 거듭제곱꼴을 일정한 규칙으로 단순화 시켜줄 수 있습니다! (물론 고유값 분해가 가능한 조건을 만족하는 행렬에 대해서만 할 수 있겠죠?)  먼저 $$\boldsymbol{X}^2$$과 $$\boldsymbol{X}^3$$은 고유값 분해를 통해 아래와 같이 구할 수 있습니다.

<br>

$$ \begin{align}
\boldsymbol{X}^2 
&= \boldsymbol{V \Lambda V}^{-1} \boldsymbol{V \Lambda V}^{-1} = \boldsymbol{V} \boldsymbol{\Lambda}^2 \boldsymbol{V}^{-1} \\
\boldsymbol{X}^3 
&= \boldsymbol{V \Lambda V}^{-1} \boldsymbol{V \Lambda V}^{-1} \boldsymbol{V \Lambda V}^{-1} = \boldsymbol{V} \boldsymbol{\Lambda}^3 \boldsymbol{V}^{-1} \\
\end{align} $$

$$ \vdots $$

<br>

규칙을 찾아보면 고유값으로 이루어진 대각 행렬에만 거듭 제곱 횟수만큼 승수를 늘려가는 것을 알 수 있습니다. 대각행렬의 거듭 제곱은 각 대각 원소에 필요한 만큼 거듭 제곱을 취하면 되기 때문에 전체적인 행렬의 거듭 제곱꼴이 아래와 같이 매우 단순화됩니다.

<br>

$$ \boldsymbol{X}^n = \boldsymbol{V} \boldsymbol{\Lambda}^n \boldsymbol{V}^{-1} $$

<br>

## 3.3 &nbsp; SVD

SVD는 고유값 분해와 마찬가지로 행렬 분해 기법입니다. 다만 고유값 분해가 정방 행렬에 대해서만 가능하다면, SVD는 직사각 행렬에도 적용할 수 있는 아주 큰 장점이 있습니다. $$m \times n$$ 행렬 $$\boldsymbol{A}$$에 대한 특이값 분해는 다음과 같이 나타낼 수 있습니다.

<br>

$$ \boldsymbol{A} = \boldsymbol{U} \boldsymbol{\Sigma} \boldsymbol{V}^{T} $$

<br>

여기서 $$\boldsymbol{U}$$는 $$\boldsymbol{AA}^{T}$$를 고유값 분해해서 얻어진 행렬이며, $$\boldsymbol{V}$$는 $$\boldsymbol{A}^{T}\boldsymbol{A}$$를 고유값 분해하여 얻어진 행렬입니다. SVD에 대한 더 자세한 내용은 향후에 더 자세히 다루고, 이글에 링크를 연결해 두겠습니다.

<br>

# 4 &nbsp; 대칭행렬과 고유값 분해

---

## 4.1 &nbsp; 대칭행렬의 의미와 특성

대칭행렬이란 $$n \times n$$ 정방행렬이면서 $$\boldsymbol{X} = \boldsymbol{X}^T$$를 만족하는 행렬을 뜻합니다. 대표적으로 공분산 행렬이 대칭행렬입니다. 대칭행렬은 아래의 두 가지 특성을 지니고 있습니다.

<br>

- **항상** 고유값 분해가 가능합니다. 
- 고유값 분해시 **직교 행렬(orthogonal matrix)**[^2]로 분해가 된다는 특징도 있습니다.

<br>

직교 행렬은 $$\boldsymbol{X}\boldsymbol{X}^T = \boldsymbol{I}$$(단위행렬)인 행렬을 말하며, 각 행벡터는 행벡터끼리 열벡터들은 열벡터끼리 서로간 **직교** 하면서 **크기가 1**인 특성이 있습니다. 다시 말해 벡터들이 정규 직교(orthonomal) 한다는 것입니다. 대칭행렬은 SVD를 다루면서 좀더 자세하게 언급할 기회가 있을 것 같습니다.

<br>

정리하자면 대칭행렬을 고유값 분해하면 다음과 같은 결과를 얻을 수 있습니다.

<br>

$$ \begin{align} \boldsymbol{X} &= \boldsymbol{V \Lambda V^{-1}}\\ &=\boldsymbol{V \Lambda V}^T \end{align}$$

<br>

# Appendix

---

## 고유값 분해 가능 조건

고유값 분해는 다음의 두 조건을 만족하는 행렬에 대해서만 가능합니다.

<br>

- 행렬 $$\boldsymbol{X}$$는  $$n \times n$$ **정방 행렬**입니다.
- 행렬 $$\boldsymbol{X}$$는 **$$n$$개의 선형 독립(linearly independent)[^1] 고유벡터**를 가집니다.

<br>

고유벡터의 선형 독립과 관련된 조건은 사실 가역 행렬(invertable matrix) 조건과 관련 있습니다. 어떤 행렬 $$\boldsymbol{V}$$의 열벡터들이 선형 독립이라면 행렬 $$\boldsymbol{V}$$는 역행렬을 가질 수 있습니다. 이를 수식으로 표현하면 다음과 같습니다.

<br>

$$ \begin{bmatrix} | & | &  & |\\ \boldsymbol{v}_1 & \boldsymbol{v}_2 & \cdots & \boldsymbol{v}_n\\     | & | &  &|
\end{bmatrix} \begin{bmatrix} c_1 \\ \vdots \\ c_n \end{bmatrix} = \boldsymbol{0} \\[1em] $$

$$\boldsymbol{V}\boldsymbol{c} = \boldsymbol{0}$$

<br>

만일 $$\boldsymbol{v}_{1, \cdots, n}$$가 선형 독립이라면 위 수식을 만족시키는 유일한 해는 $$c_1 = \cdots = c_n = 0$$ 입니다. 따라서 행렬 $$\boldsymbol{V}$$는 가역행렬이라는 사실을 도출할 수 있습니다.

<br>

> 행렬 $$\boldsymbol{V}$$가 가역 행렬이라는 것은 $$\boldsymbol{V}$$의 열벡터들이 선형 독립이라는 것과 동치입니다.

<br>

우리 예시에서 $$\boldsymbol{V}$$는 원 행렬 $$\boldsymbol{X}$$의 고유 벡터를 열벡터로 하는 행렬입니다. 따라서 행렬 $$\boldsymbol{V}$$가 선형 독립이라는 사실은 $$n \times n$$ 행렬 $$\boldsymbol{X}$$이 $$n$$개의 선형 독립 고유벡터를 가진다는 사실을 뜻하게 됩니다.

<br>

$$\boldsymbol{XV}\boldsymbol{V}^{-1} = \boldsymbol{V\Lambda} \boldsymbol{V}^{-1} $$

$$\boldsymbol{X}= \boldsymbol{V\Lambda} \boldsymbol{V}^{-1} $$

<br>

# Reference

---

- [대각화와 고유값 분해](http://elearning.kocw.net/contents4/document/lec/2013/Chungbuk/LeeGeonmyeong1/12.pdf)
- [Proof that columns of an invertible matrix are linearly independent](https://math.stackexchange.com/questions/1925062/proof-that-columns-of-an-invertible-matrix-are-linearly-independent)
- [정방행렬의 대각화(diagonalization): 고유값 분해(eigendecomposition).txt](https://bskyvision.com/127) 
- [선형독립(linearly independent), 선형종속(linearly dependent)](https://rfriend.tistory.com/163)
- [Eigenvalue Decomposition](https://blueblazin.github.io/math/2016/08/18/eigenvalue-decomposition.html)

<br>

# footnote

---

[^1]: $$k$$개의 벡터 $$\boldsymbol{v}_1, \boldsymbol{v}_2, \cdots, \boldsymbol{v}_k$$에 대해서 이들 벡터의 선형 결합인 $$c_1\boldsymbol{v}_1+c_2\boldsymbol{v}_2+\cdots+c_k\boldsymbol{v}_k = 0$$를 만족하는 해가 $$c_1=c_2=\cdots=c_k=0$$가 유일하다면 이를 선형 독립이라고 합니다. 다른 말로 하면 $$k$$개의 벡터 중 임의의 벡터 $$\boldsymbol{v}_i$$를 다른 벡터들의 선형결합으로 표현할 수 없는 것입니다. 대비되는 개념은 선형 의존(linearly dependent)입니다.
[^2]: $$\boldsymbol{XX}^T = \boldsymbol{I}$$, 즉 자신과 대칭인 행렬과 곱하면 단위 행렬이 되는 행렬입니다. 역행렬의 정의에 따라 $$\boldsymbol{X}^{-1} = \boldsymbol{X}^T$$입니다.