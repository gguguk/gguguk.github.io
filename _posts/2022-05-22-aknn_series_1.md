---
title: AkNN 시리즈 1 - 서론
author: Gukwon Koo
categories: [CS, AkNN]
tags: [aknn, vector search, quantization]
pin: false
math: true
comments: true
date: 2022-05-22 16:21:00 +0900
---

일반적으로 추천 알고리즘을 서빙하는 과정을 매우 단순화 하여 표현하면 다음과 같습니다.

1. 데이터를 모아서 추천 알고리즘을 학습 시킵니다. 
2. 추천 알고리즘에 어떤 오브젝트(유저, 아이템 등)의 정보를 인풋하여 각 오브젝트의 벡터값을 찾습니다.
3. **벡터값들의 유사도를 비교하여 가장 가까운 $$k$$개의 오브젝트를 찾고, 이를 추천 결과로 내보냅니다.**

벡터들간의 유사도를 효율적으로 계산하여 최종 추천 결과를 뽑아내는 과정이 중요하다는 것을 알 수 있습니다. 따라서 이번 AkNN 시리즈 글에서는 벡터간의 유사도를 빠르게 비교하는 방안에 대한 연구 방향을 개략적으로 정리합니다. 특히 다음 글부터는 quantization과 관련된 내용을 심화로 다룰 예정입니다. 저도 공부를 하며 정리하고 있기 때문에 잘못된 부분이 있을 수도 있습니다. 지적 부탁드리고, 이 주제에 관심 있으신 분들은 함께 시작해 보시죠! 👋

<br>

# Vector Search란?

---

> vector search는 데이터가 임베딩된 벡터 공간 상에서 질의(query) 벡터와 가장 가까운 $$k$$의 벡터를 찾는 행위를 말합니다.

우리는 살아가면서 정말 많은 '검색' 행위를 합니다. 일반적인 회사원 분들이라면 네이버나 구글에서 맛집을 검색하거나 엑셀 단축키 등을 검색하실 것이구요. 개발자 분들께서는 원하는 데이터를 찾기 위해 사내 데이터 베이스에 수많은 검색을 하실 겁니다. 그리고 대부분의 검색은 그 의도가 매우 명확할 겁니다. 예를 들어서 "최근 한달동안 가장 많이 팔린 제품의 개수는?" 또는 "저번 달에 신규 가입한 유저들 중 이번 달에 잔존한 비율은?" 등을 들 수 있겠네요.

그러나 "에어팟과 가장 유사한 제품 3개를 찾아줘"라는 질의는 어떤가요? 위에서 언급한 질의보다는 모호한 것은 사실입니다. 그러나 우리는 일상 생활에서 이러한 표현을 많이 사용합니다. 그리고 사람은 그 문장이 내포하는 의미를 자연스럽게 해석하고, 이에 대한 해답을 제시 할 수 있습니다. 예를 들어 맥북, 아이패드, 애플워치를 들 수 있겠네요.

그렇다면 어떻게 컴퓨터는 추상적인 질문에 적절한 답을 줄 수 있을까요? 이를 위해서 머신러닝 기술이 활용됩니다. 특히 RNN, CNN, BERT 등 유명한 알고리즘들은 어떻게 데이터를 벡터로 잘 표현할 수 있을까에 대한 뉴럴넷 기반의 대안이라고 할 수 있습니다. 좋은 알고리즘은 데이터의 추상적이고 내재적인 의미를 잘 임베딩 벡터로 표현할 수 있습니다. 

이렇게 벡터 공간에 하나의 점으로 임베딩된 데이터들은 유사한 의미를 가진 데이터일수록 서로 가까운 위치에 존재하게 되고, 이질적인 의미를 가질 수록 서로 멀리 위치하게 됩니다. 특히 데이터를 벡터로 표현하면 여러가지 거리 또는 유사도 함수를 사용하여 벡터들 간의 유사도를 대수적으로 계산할 수 있게 됩니다.

특히 서두에서 언급한 벡터간의 유사도를 비교하여 가장 가까운 벡터를 찾는 행위, 즉 vector search는 추천 시스템을 구성하는 핵심 컴포넌트라고 할 수 있습니다. 좀 더 강하게 말하면, **추천 알고리즘의 성능은 위와 같은 질문에 적절한 벡터를 얼마나 정확하고 빠르게 제공할 수 있느냐에 달려 있다고 할 수 있습니다.**

<br>

# vector search는 생각보다 어렵다

---

vector search는 특정 distance/similarity 함수에 따라서 질의(query) 벡터와 가장 가까운 $$k$$ 개의 벡터를 찾는 행위입니다. 그런데 vector search가 말 처럼 쉽지는 않습니다. 100만 개의 벡터에서 질의 벡터와 가장 가까운 10개의 벡터를 찾는다고 해봅시다. 가장 가까운 벡터를 찾기 위해서는 어쩔 수 없이 질의 벡터와 나머지 100만 개의 벡터와의 distance/similarity를 구해야 합니다. 모든 벡터에 대해 비교를 해봐야 가장 가까운 벡터를 찾을 수 있으니까요. 만일 이 행위가 0.1초가 소요 된다고 해봅시다. 0.1초 자체만 보면 엄청 짧은 시간이죠. 

그러나 100만개 모두에 대해 가장 유사한 10개의 벡터를 찾는다고 해봅시다. 단순히 산술적으로 계산하면 100,000초가 소요됩니다. 이를 시간으로 환산하면 **무려 27시간**입니다. 보통 추천 시스템을 운용할 때는 특정 주기마다 추천으로 나갈 제품 셋을 미리 연산해 놓고 데이터 베이스에 쌓아 놓는 방식을 많이 활용합니다. 만약 배치 주기가 24시간(하루)이라면... 27시간이나 소요되는 연산은 도저히 운용이 불가능합니다. 

vector search를 실무에서 활용할 수 있으려면 어떤 방법론을 고민해보아야 할까요? 검색 속도를 빠르게 할 수 있다면 검색의 정확도를 조금 희생해도 괜찮지 않을까요? 물론 그 정확도의 희생은 용인 가능한 수준이어야 하겠지만요. 이러한 관점에서 탄생한 알고리즘을 통칭 **AkNN**(Approximate k Neareast Neighbor)이라고 합니다.

<br>

#  Approximate K Neareast Neighbor(AkNN)

---

AkNN 알고리즘을 다시 정의해봅시다. AkNN은 정확도는 손해를 보더라도 빠르게 vector search를 할 수 있는 알고리즘을 통칭합니다. 요즘 빅데이터 시대라고 하죠. 제가 몸 담고 있는 현업에서도 제품수가 200만개를 훌쩍 뛰어 넘습니다. 회사가 성숙할수록 데이터는 계속해서 증가할 것으므로 **reasonable 시간 안에서** 유사한 데이터를 발견하는 일은 점점 더 중요해질 것입니다. 

AkNN 알고리즘의 연구 방향은 크게 2가지로 나눌 수 있습니다:

- Vector Encoding
  - Tree
  - LSH(Locality Sensitive Hashing)
  - **Quantization**
- None Exhaustive Search Component: 완전 탐색을 피하기 위한 방법
  - Inverted Files
  - HNSW(Hierarchical Navigable Small World Graphs)

AkNN과 관련된 내용을 모두 다루면 좋으나... 시간의 한계로 그럴 수는 없겠습니다. 따라서 이번 시리즈 글에서는 FAISS나 ScaNN과 같은 유명한 AkNN 오픈 소스 패키지에서 반드시 언급되는 quantization 기법 중 VQ(Vector Quantization)와 PQ(Product Quantization)을 자세히 다루고자합니다. 참고로 quantization 기법은 이렇게 다양합니다. 관심 있으신 분은 아래의 키워드를 참고해서 학습해보세요!

- Vector quantization
- Product quantization
- Binary quantization
- Additive quantization
- Tenary quantization
- Learing quantization

<br>

# Reference

---

- [What is Similarity Search?](https://www.pinecone.io/learn/what-is-similarity-search/)
- [Facebook AI Similarity Search (Faiss): The Missing Manual](https://www.pinecone.io/learn/faiss-tutorial/)
- [Comprehensive Guide To Approximate Nearest Neighbors Algorithms](https://towardsdatascience.com/comprehensive-guide-to-approximate-nearest-neighbors-algorithms-8b94f057d6b6)