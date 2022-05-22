---
title: vector search 시리즈 1 - 서론
author: Gukwon Koo
categories: [CS, Vector Search]
tags: [vector search, quantization]
pin: false
math: true
comments: true
date: 2022-05-22 16:21:00 +0900
---

# Vector Search란?

- similarity search라고도 불림
- 벡터들 사이의 유사도를 계산해서 가장 가까운 벡터를 찾는 행위를 말함

# 왜 필요?

- vector representation은 단순한 의미를 넘어서 semantic 의미를 반영하고 있음
- 이들 사이의 유사도를 계산하면, 다양한 ML 분야에서 응용이 가능함
- 추천 시스템, 이미지 유사도 비교 등등

# Vector Search 그냥 하면 되는 거 아니야?

- 유사한 벡터를 효율적으로 찾을 수 있는 방안이 필요함
- 200만개에 대해서 유사도를 계산 해본다고 해보자... 하나의 n개의 벡터 중에서 query vector q와 가장 유사한 k개의 벡터를 찾으려면... 완전 탐색 방법으로 0.1초가 걸린다고 해보자. 
- 0.1초는 100ms로 어떻게 보면 빠르다고 생각할 수도 있음
- 그러나 이러한 연산을 n개의 벡터에 적용하면... 소요시간은 연산 소요시간은 기하급수적으로 늘어남

# Vecotor search의 연구 방향

- 크게 두가지 방향으로 요약 할 수 있음
- 비교할 대상을 줄이는 연구 방향
  - partitioning, clustering, ...
  - tree
  - LSH
  - graph
  - ...
- 유사도 계산을 효율적으로 하는 방향
  - quantization
    - 메모리 효율적 활용
    - Remember that the magic of Product Quantizers is that you only have to calculate a relatively small table of partial distances between the query vector chunks and the codes in the codebooks–the rest is look-ups and summations.