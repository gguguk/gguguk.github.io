---
title: 고유값(eigen value)과 고유벡터(eigen vector)
author: Gukwon Koo
categories: [Math, Linear algebra]
tags: [eigenvalue, eigenvector]
pin: false
math: true
comments: true
date: 2021-01-05 22:20:00 +0900
---

![](https://upload.wikimedia.org/wikipedia/commons/0/06/Eigenvectors.gif)_방향은 변하지 않고 크기만 변하는 벡터를 찾아라!_

<br>

---

**두줄 요약 !!**

- 선형 변환을 했을 때 크기는 변하나 방향은 변하지 않는 벡터를 고유벡터라고 하고 변하는 크기는 고유값이라 함

- 행렬 분해 기법(EVD, SVD)과 PCA 등 여러 응용 사례의 가장 기초가 되는 개념

---

<br>

선형대수학 개념 중 고유값(eigen value)와 고유벡터(eigen vector)는 머신러닝과 아주 밀접한 관련이 있습니다. 머신러닝을 공부해보신 분들은 한번쯤은 꼭 들어보셨을 PCA, SVD 및 행렬 분해를 활용한 추천 시스템 등의 근간이 되는 개념입니다. 그러나 그 의미를 직관적으로 이해하기는 쉽지 않습니다. 이번 글에서는 직관을 기르기 위해 고유값과 고유벡터의 수식적 의미와 기하학적 의미를 살펴보고 간단하게 예시를 들어 직접 계산 해보겠습니다. 그리고 고유값과 고유벡터를 활용하는 응용 분야에 대해서 간단하게 짚고 넘어가겠습니다. 사실 이번 글은 응용 분야를 차례대로 정리하기 위한 시작글입니다. 관련 내용 계속 포스팅 하겠습니다.

<br>

# 1 &nbsp; 고유값(eigen value)과 고유벡터(eigen vector)란?

---

## 기본적 의미

[행렬을 선형 변환](https://angeloyeo.github.io/2019/07/15/Matrix_as_Linear_Transformation.html)[^6]이라고 한다면 **선형 변환 정방행렬 $$A$$를 가했을 때 크기(scale)만 변하고 방향은 변하지 않는 영벡터가 아닌 벡터를 고유벡터(eigen vector) $$\vec{v} $$ 라고 하며 이때의 크기를 고유값(eigen value) $$\lambda$$**라고 합니다. 좀더 정확하게 말하면 고유값과 고유벡터는 행렬에 대응되는 개념입니다. 따라서 고유값 $$\lambda$$는 행렬 $$A$$의 고유값 $$\lambda$$이며 고유벡터 $$\vec{v}$$는 행렬 $$A$$의 고유값 $$\lambda$$에 대한 고유벡터 $$\vec{v}$$라고 표현하는 것이 더욱 정확하겠습니다. 말로 풀어쓴 것을 수식으로 정리하면 매우 간단합니다.

<br>

$$
A\vec{v} = \lambda \vec{v}
$$

$$
\left[
    \begin{matrix} 
        a_{11} & \cdots & a_{1n} \\
        \vdots & \ddots & \vdots \\
        a_{n1} & \cdots & a_{nn}
    \end{matrix} 
\right]
\left[
	\begin{matrix}
		v_{1} \\
		\vdots \\
		v_{n}
	\end{matrix}
\right] = 
\lambda
\left[
	\begin{matrix}
		v_{1} \\
		\vdots \\
		v_{n}
	\end{matrix}
\right]
$$

<br>

위 수식의 우항을 좌항으로 넘겨서 수식을 정리해보면 다음과 같이 나타낼 수 있습니다. 이 수식을 **특성 방정식(chracteristic equation)**이라고 하며 고유값과 고유벡터를 구하는 것은 특성 방정식의 해를 찾는 것입니다. 이 방정식을 고유값 $$\lambda$$에 대해서 풀고, 다시 그 값을 대입하여 고유벡터 $$\vec{v}$$를 찾을 수 있습니다.

<br>

$$ (A-\lambda I)\vec{x} = 0 $$ 

<br>

이 방정식의 $$\lambda$$ 해를 어떻게 찾을 수 있을까요? 고유 벡터의 정의를 말씀드렸을 때 **고유벡터는 영벡터가 아닌 벡터이다** 라고 말씀 드렸죠. 그 조건을 활용해 보자구요. 고유벡터가 영벡터가 아니기 위해서는 $$(A-\lambda I)$$ 행렬이 **역행렬[^4]이 존재하지 않아야 합니다.**  만일 $$(A-\lambda I)$$이 역행렬을 가지게 된다면 좌항과 우항에 $$(A-\lambda I)^{-1}$$을 곱해서 $$\vec{x}=0$$이 되기 때문입니다. 이는 고유벡터의 정의에 위배되는 것이죠. 따라서 특성 방정식의 $$\lambda$$ 에 대한 해를 구하기 위해서는 **행렬식[^5]의 값이 0** 이어야 함을 알 수 있습니다.

<br>

$$ det(A-\lambda I)=0 $$  

<br>

마지막으로 중요하게 짚고 넘어가야 할 점이 있습니다.

- 고유값과 고유벡터는 $$n \times n$$ **정방 행렬 $$A$$**에 대해서만 구할 수 있습니다.
- 고유값은 행렬에 대해 유한한데 비해, **고유벡터를 유일한 값이 아닙니다.** 다시 말해 특정 고유값은 여러개의 고유벡터를 가질 수 있습니다(이 개념은 섹션2를 읽어보시면 이해가 쉽게 되실 겁니다.)



<br>

## 기하학적 의미

행렬은 선형변환이라는 사실을 알고 있다면 특정 행렬 $$A$$와 특정 벡터 $$\vec{v}$$가 있을 때, 그 둘을 곱하는 것은 벡터 $$\vec{v}$$를 새로운 기저 벡터로 표현하는 것이라는 사실을 이해하실 수 있을 것입니다. 따라서 대부분의 경우에는 기존 벡터 $$\vec{v}$$와 선형 변환 후의 벡터 $$A\vec{v}$$의 크기나 방향이 모두 달라질 가능성이 높습니다. 그러나 선형 변환후에도 크기는 달라질 수 있지만 방향이 변하지 않는 벡터들도 존재합니다. 그러한 벡터들을 고유벡터라고 하죠.  아래 이미지를 살펴봅시다. 두 이미지는 원래의 이미지가 옆으로 기울어진 이미지로 변화하는 선형 변환을 보여 주고 있습니다.

<br>

![](https://upload.wikimedia.org/wikipedia/commons/3/3c/Mona_Lisa_eigenvector_grid.png){:width="650"}

<br>

선형 변환전에는 빨간색 벡터와 파란색 벡터가 각각 $$\vec{v}=\left[\begin{matrix} 0 \\ 5 \end{matrix} \right]$$, $$\vec{k}=\left[\begin{matrix} 5 \\ 0 \end{matrix} \right]$$를 나타냅니다. 그러나 선형 변환후에는 빨간색 벡터와 파란색 벡터가 각각 $$\vec{v}=\left[\begin{matrix} 1.2 \\ 5 \end{matrix} \right]$$, $$\vec{k}=\left[\begin{matrix} 5 \\ 0 \end{matrix} \right]$$입니다. 빨간색 벡터 $$\vec{v}$$는 크기와 방향이 모두 변화한 반면에 파란색 벡터 $$\vec{k}$$는 크기만 1만큼 변화하였고 방향은 변화하지 않았습니다. 따라서 선형 변환 후에도 크기는 변하지만 방향은 변하지 않는 벡터 $$\vec{k}$$는 해당 선형 변환의 고유벡터입니다. 크기는 1만큼 변했으므로 고유값 $$\lambda$$는 1이라고 할 수 있겠네요. 

<br>

# 2 &nbsp; 직접 고유값과 고유벡터를 계산해보자

---

선형 변환 행렬 $$A=\left[\begin{matrix} 2 & 1 \\ 1 & 2 \end{matrix} \right]$$일 때, 고유값 $$\lambda$$와 고유벡터 $$\vec{v}$$를 찾아보겠습니다. 먼저 섹션 1에서 서두에 언급한 고유값과 고유벡터의 정의를 수식으로 다시 적어보겠습니다.

<br>

$$ A\vec{x} = \lambda \vec{x} $$

<br>

또한 위에서 언급하였듯이 $$\vec{x}$$가 non-trivial solution이기 위해서는 다음의 식을 만족해야 합니다.

<br>

$$ det(A-\lambda I) = 0 $$

<br>

위 수식에 실제 값을 대입하면,

<br>

$$ \begin{align}
det(A-\lambda I)
&= det\bigg(\left[\begin{matrix} 2 & 1 \\ 1 & 2 \end{matrix} \right] - \lambda \left[\begin{matrix} 1 & 0 \\ 0 & 1 \end{matrix} \right] \bigg)  \\[1em]
&= det\bigg(\left[\begin{matrix} 2-\lambda & 1 \\ 1 & 2-\lambda \end{matrix} \right] \bigg) \\[1em]
&= \lambda^2 -4 \lambda +3 \\[1em]
&= (\lambda-3)(\lambda-1) \\[1em]
&= 0
\end{align} $$

<br>

따라서 행렬 $$A$$의 고유값은 $$\lambda = 3$$, $$\lambda=1$$임을 얻어내었습니다.  이제 이 값을 이용하여 고유 벡터를 찾아보죠. 먼저 $$\lambda=1$$ 일때의 고유벡터를 찾아볼게요. $$A\vec{x} = \lambda \vec{x}$$에 $$\lambda=1$$를 대입하면,

<br>

$$ \left[\begin{matrix} 2 & 1 \\ 1 & 2 \end{matrix} \right] \left[\begin{matrix} v_1 \\ v_2 \end{matrix} \right] = 1 \left[\begin{matrix} v_1 \\ v_2 \end{matrix} \right] \tag{1} $$

<br>

위 수식은 아래의 연립방정식을 계산 하는 것입니다.

<br>

$$ \begin{align}
2v_1 + v_2 &= v_1 \\
v_1 + 2v_2 &= v_2
\end{align} $$

<br>

따라서 얻을 수 있는 솔루션은

<br>

$$ v_1 = -v_2 $$

<br>

따라서 고유값이 $$\lambda=1$$일때의 고유 벡터는 $$\left[\begin{matrix} 1 \\ -1 \end{matrix} \right]$$, $$\left[\begin{matrix} 0.8 \\ -0.8 \end{matrix} \right]$$ 등의 여러 값을 가질 수 있습니다. 



이쯤에서 애매함을 느끼실 것 같습니다. '고유값을 구하는 것까지는 딱 떨어져서 이해 했는데, 고유벡터는 여러개의 값을 가질 수 있는건가?' 네 그렇습니다. 섹션 1에서도 잠깐 언급했지만 어떤 행렬의 고유값은 유한하게 딱 정해져 있지만 고유값에 대응되는 고유벡터는 다수 존재할 수 있습니다. 이는 실제로 여러가지 값을 대입 해보면 이해가 됩니다.  

이처럼 고유벡터는 여러개가 될 수 있으므로 일반적으로는 고유벡터를 단위벡터[^3]로 표기 한다고 합니다. 그렇다면 본 예시에서는 고유값 $$\lambda=1$$에 대응되는 고유벡터를 $$\left[\begin{matrix} 1 / \sqrt{2}\\ -1 / \sqrt{2} \end{matrix} \right]$$로 나타낼 수 있습니다.

같은 방식으로 고유값 $$\lambda=3$$ 일때의 고유 벡터는 $$\left[\begin{matrix} 1 / \sqrt{2}\\ 1 / \sqrt{2} \end{matrix} \right]$$입니다.

<br>



# 3&nbsp; 응용

---

## 고유값 분해(eigen decompostion)

정방행렬을 대각화[^1]하는 방법입니다. 그러나 모든 정방 행렬이 고유값 분해가 가능한 것은 아닙니다. $$n \times n$$ 정방 행렬 $$A$$를 고유값 분해하기 위해서는 행렬 $$A$$가 $$n$$개의 선형 독립(linearly independent)[^2]인 고유벡터를 가져야 합니다. 자세한 내용은 추후에 포스팅 하도록 하겠습니다.

<br>

## PCA(주성분 분석)

앞서 기하학적 의미에서 살펴 보았듯이 고유벡터는 변화의 '축' 이라고도 생각할 수 있겠습니다. 마치 지구가 자전하는 3차원 선형 변환에서 자전축은 고정된 것처럼 말이죠. 고유벡터를 변화의 축이라는 개념을 활용한 기법이 바로 PCA입니다. PCA는 데이터가 가진 정보를 최대한 보존하면서 변수를 추출하는 기법입니다. 데이터가 가진 정보를 최대한 보존하면서 차원을 줄인다는 의미는 데이터를 공분산 행렬의 고유벡터에 사영(projection) 한다는 의미인데요. 자세한 내용은 [이 곳](https://gguguk.github.io/posts/PCA/)을 참고해주세요.

<br>

## SVD(특이값 분해)

SVD는 대각화를 통해 행렬을 분해하는 기법입니다. 고유값 분해는 정방 행렬에만 대각화를 할 수 있으나 SVD는 정방행렬이 아니어도 대각화가 가능하다는 장점이 있습니다. 그러나 단순히 행렬을 '분해'하는 것보다는 [분해한 행렬에서 정보가 큰 순서대로 일정 개수의 벡터를 추려서 근사적으로 기존 행렬로 복원하는 식으로 많이 사용되는 기법](https://angeloyeo.github.io/2019/08/01/SVD.html)입니다. SVD의 수식을 살펴 보면, 결과적으로 특정 행렬의 고유값과 고유벡터를 구하는 문제로 귀결됩니다. 자세한 내용은 추후에 따로 포스팅 하도록 하겠습니다.

<br>

# 참고자료

---

- [고유값과 고유벡터 (eigenvalue & eigenvector), 다크 프로그래머](https://darkpgmr.tistory.com/105)
- [고윳값과 고유벡터, 공돌이의 수학정리노트](https://angeloyeo.github.io/2019/07/17/eigen_vector.html)
- [고윳값 분해(eigen-value decomposition), 공돌이의 수학정리노트](https://angeloyeo.github.io/2020/11/19/eigen_decomposition.html)
- [고유값(eigenvalue)과 고유벡터(eigenvector)의 정의, R Friend)](https://rfriend.tistory.com/181?category=606751)
- [행렬식(determinant), ratsgo's blog](https://ratsgo.github.io/linear%20algebra/2017/05/21/determinants/)
- [행렬식의 기하학적 의미, 공돌이의 수학정리노트](https://angeloyeo.github.io/2019/08/06/determinant.html)
- [SVD와 PCA, 그리고 잠재의미분석(LSA), ratsgo's blog](https://ratsgo.github.io/from%20frequency%20to%20semantics/2017/04/06/pcasvdlsa/)

<br>

# 각주

[^1]:행렬에 조작을 가하여 대각 성분을 제외한 모든 값을 0으로 만드는 것을 말합니다.
[^2]: 어느 한 벡터를 다른 벡터들의 선형 결합(상수배의 합)으로 표현할 수 없으면 이를 선형 독립이라고 합니다.
[^3]: 길이가 1인 벡터를 뜻합니다. $$ \|v\|=1 $$
[^4]: 원래 행렬과 곱했을 때 단위 행렬이(=항등 행렬) 되는 행렬을 뜻합니다. $$ AA^{-1} = I $$
[^5]: 행렬의 원소들을 특정 수식에 따라 계산하여 나온 결과값. 특별한 정의는 없으나 그 결과값이 행렬의 특성을 결정 짓는다는 점에서 중요하게 다뤄집니다. 특히 행렬식을 통해서 [역행렬의 존재 여부를 판정](https://angeloyeo.github.io/2019/08/06/determinant.html)할 수 있습니다.

[^6]: [좌표 변환과 선형 변환은 한끗 차이입니다.](https://blog.naver.com/je1206/220818602286) 어떤 행렬에 벡터를 곱했을 때 산출되는 값은 '점이 이동한 것'으로 볼 수도 있고 '좌표 축을 변환' 한 것으로도 볼 수 있기 때문입니다.