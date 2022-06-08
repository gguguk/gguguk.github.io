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

# Vector Quantization이란?

- vector 정보를 압축 또는 인코딩하는 방법. 2차원 벡터라고 생각하면 좀 이해하기 쉬울 듯
- N개의 벡터를 대표적인 K개의 벡터(K<N)로 맵핑하는 과정을 벡터 양자화라고 함
- 쿼리 벡터 -> codebook에서 가장 가까운 codeword 찾고 -> codeword와
- 양자화 하는 과정에서 필연적으로 loss가 발생. 원래의 벡터를 거리가 가까운 다른 벡터로 대처 하기 때문에.
- 따라서 그 손실을 최소화하는 양자화 함수 $$q(x)$$를 찾는 것이 중요함

# Terminology

- index
  - A database for vectors
- quantization
  - 
- codebook
  - quantized 벡터들의 집합. 이 집합의 원소의 개수를 codebook의 size라고 함
- codeword
  - codebook 집합에 존재하는 quantized 벡터들을 지칭
- MIPS(Maximum Inner Product Search)
  - popular paradigm for solving large scale classiﬁcation and retrieval tasks.
  - Exhaustively computing the exact inner product between q and n datapoints is often expensive and sometimes infeasible.
  - Several techniques have been proposed in the literature based on hashing, graph search, or **quantization** to solve the approximate maximum inner product search problem eﬃciently, and the quantization based techniques have shown strong performance.

# Methodology

- 여러가지 기초적인 방법론

# 파이썬 구현

- brute-force와의 비교
- FAISS의 VQ?

# Reference

---

- [What is the difference between K-Means and Voronoi?](https://www.quora.com/What-is-the-difference-between-K-Means-and-Voronoi)
- [딥러닝 Quantization[시리즈 1/3] – vector quantization 알아보기](https://datacrew.tech/vector-quantization/)
