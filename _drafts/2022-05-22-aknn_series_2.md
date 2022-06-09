---
title: AkNN 시리즈 2 - vector quantization
author: Gukwon Koo
categories: [CS, AkNN]
tags: [aknn, vector search, quantization]
pin: false
math: true
comments: true
date: 2022-05-22 16:21:00 +0900
---

이번 글은  AkNN 시리즈의 두번 째 글입니다. [저번 글](https://gguguk.github.io/posts/aknn_series_1/)에서는 vector search란 무엇이고 실무에서 vector search를 활용하기 어려운 이유에 대해서 언급했습니다. 그리고 효율적인 vector search를 위한 방법론으로 AkNN을 알고리즘이 있다는 것을 말했습니다. 특히 AkNN 알고리즘은 다양한 방향으로 연구되어 왔는데, 그 중에서 가장 주목 받고 있는 벡터 양자화(vector quantization)에 집중해서 시리즈 글을 작성할 것이라고 언급하고 글을 마무리 했었네요.

따라서 이번 글에서는 벡터 양자화가 무엇인지, 그리고 이쪽 분야에서 자주 등장하는 낯선 용어들을 정리하면서 시작하겠습니다. 그리고 벡터 양자화를 직관적으로 이해를 해보고, 실제 파이썬 코드로 구현하여 완전 탐색(brute force)에 비해서 속도 측면에서는 얼마나 빠르고, 정확도는 얼마나 손해를 볼지도 같이 확인 하는 시간을 가져보려고 합니다.

<br>

# Terminology

----

- index
  - A database for vectors
- codebook
  - quantized 벡터들의 집합. 이 집합의 원소의 개수를 codebook의 size라고 함
- codeword
  - codebook 집합에 존재하는 quantized 벡터들을 지칭
- MIPS(Maximum Inner Product Search)
  - popular paradigm for solving large scale classiﬁcation and retrieval tasks.
  - Exhaustively computing the exact inner product between q and n datapoints is often expensive and sometimes infeasible.
  - Several techniques have been proposed in the literature based on hashing, graph search, or **quantization** to solve the approximate maximum inner product search problem eﬃciently, and the quantization based techniques have shown strong performance.

<br>

# Vector Quantization이란?

---

- vector 정보를 압축 또는 인코딩하는 방법. 2차원 벡터라고 생각하면 좀 이해하기 쉬울 듯
- N개의 벡터를 대표적인 K개의 벡터(K<N)로 맵핑하는 과정을 벡터 양자화라고 함
- k-dimensional vector(in $$R^k$$)를 유한한 벡터의 집합 $$Y=\{y_1, y_2, ..., y_n\}$$으로 맵핑하는 방법론을 통칭하는 말임.
  - $$y_i$$를 codeword라고 하고, 집합 $$Y$$를 codebook이라고 함. 그리고 집합 Y의 크기를 codebook의 크기라고 함

- 이는 voronoi cell을 찾는 과정으로 이해할 수 있음
  - 각 voronoli cell의 

- 양자화 하는 과정에서 필연적으로 loss가 발생. 원래의 벡터를 거리가 가까운 다른 벡터로 대처 하기 때문에.
  - 이러한 loss를 distortion이라고도 표현함
  - 가장 간단하게는 MSE로 vector quantization을 평가 할 수 있음

- 실제 사용은?쿼리 벡터 -> codebook에서 가장 가까운 codeword 찾고 -> codeword와
  - 

- 따라서 그 손실을 최소화하는 양자화 함수 $$q(x)$$를 찾는 것이 중요함
- 얼핏보면 차원 축소 알고리즘과 비슷해 보일 수 있음. 그러나 엄연히 다른 알고리즘
  - 차원 축소는 k 차원의 벡터들을 d(d<k) 차원의 벡터로 맵핑하고 나면, 새로 사상된 벡터들끼리 바로 대수적 연산이 가능
  - 그러나 벡터 양자화는 codeword를 찾은 후에 사상된 codeword 벡터를 사용하지 않고 index: codeword로 이루어진 일종의 look-up table의 형태로 가지고 있음. 따라서 인덱스끼리의 대수적 연산은 의미가 없음


<br>

# Vector quantization이 좋은 이유?

---

- float를 integer 형으로 바꿔서 메모리의 효율성 증대
- 압축하면 왜 좋지? 메모리의 band-width와 관련이 있나?

<br>

# Codebook을 만드는 방법

---

- 여기까지 봤으면 vector quantization이 무엇이고, 왜 검색 효율성을 높여주는지 이해했을 것. 즉 검색 속도가 왜 빠른지는 이해가 됨
- 그런데 조금만 생각해보면 vector quantization은 결국에 원래의 벡터를 다른 벡터로 대표해서 표현하기 때문에 정보의 손실이 필연적으로 발생하게 됨. 즉 작동 원리상 정확도의 손실이 무조건 발생함. 
- 그렇다면 최대한 손실(distortion)이 적은, 학습에 사용했던 여러 벡터를 잘~ 대표할 수 있는 codebook을 만드는 것이 중요할 것
- 그 중 suboptimal 솔루션이지만, 가장 간단한 방법인 LBG 소개. 이는 사실 k-means와 비슷.
  - L2(유클리디안 거리) 사용하여 centroid를 찾음


<br>

# 손실을 평가하는 방법?

---

- 이것도 정말 다양한 방법이 있고, 
- 그 중 가장 간단한 방법이 MSE 소개
- 검색 정확도는 보통 recall로 리포트

<br>

# 단점은 없을까?

---

- 이런 이런 단점이 있다.
- 그래서 product quantization으로 가자. 이거는 쉽다. 벡터를 split해서 각각에 대해서 VQ를 하면된다. 다음 글에서 살펴보자.

<br>

# 파이썬 구현

---

- brute-force search vs VQ(via k-means) 검색 속도 및 정확도 비교
- FAISS의 VQ?

<br>

# Reference

---

- [What is the difference between K-Means and Voronoi?](https://www.quora.com/What-is-the-difference-between-K-Means-and-Voronoi)
- [딥러닝 Quantization[시리즈 1/3] – vector quantization 알아보기](https://datacrew.tech/vector-quantization/)
