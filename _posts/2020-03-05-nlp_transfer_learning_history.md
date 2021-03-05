---
title: NLP Transfer Learning History
author: Gukwon Koo
categories: [ML, NLP]
tags: [nlp, transfer learning]
pin: false
math: true
comments: true
---

> 김성현 연구원님의 T아카데미 토크ON세미나 '딥러닝 기반의 자연어 언어 모델 BERT' 라는 세미나를 듣고 알게된 정보와 구글링을 통해 알게된 정보를 종합하여 관련 내용을 정리해 보고자 합니다.

## 1 &nbsp; From word embedding To pretrained language models

### 1.1 &nbsp; Traditional context-free representation

#### 1.1.1 &nbsp; One-hot encoding

- 특징: 단어를 수치화하여 표현하기 위해서 단어 사전 크기의 차원을 가지는 벡터를 만들고 단어의 인덱스에 해당하는 원소만 1, 나머지는 0으로 하여 표현
- 단점: 단어의 의미(meaning)를 인코딩 할 수 없음, 단어끼리의 유사성을 계산할 수 없음, 매우 sparse한 벡터 표현으로 연산 자원의 낭비를 초래함

#### 1.1.2 &nbsp; TF-IDF representation - 통계 기반 기법

- 특징: 통계적 접근 방법으로서 TF(term frequency)와 IDF(inverse document frequency) 값을 이용하여 단어의 document 내에서의 중요도를 표현할 수 있는(수치화 할 수 있는) 방법
- 단점: 단어의 의미(meaning)을 인코딩 할 수 없음, one-hot encoding 보다는 덜 하지만 여전히 sparse한 벡터 표현으로 연산 자원의 낭비를 초래함

<br>

#### 1.2 &nbsp; Distributed similarity based representation: Word embeddings

#### 1.2.1 &nbsp; word2vec - 추론 기반 기법

- 특징: 어떤 단어의 의미는 주변 단어들에 의해 결정된다는 분포가설(distributed hypothesis)에 기초하여 은닉층이 1개인 간단한 인공신경망을 통해 context 단어를 통해 target 단어를 예측 하거나(CBOW), target 단어를 이용하여 context 단어를 예측(skip-gram) 방식을 통해 확률 분포를 모델링함. 그 결과로 얻어진 은닉층 가중치 벡터를 단어의 representation으로 활용함
- 장점: 단어의 의미(meaning)를 인코딩 할 수 있고, 단어 사이의 유사성도 추론하는 작업을 할 수 있음
- 단점: OOV(out of vocabulary) 단어들을 처리할 수 없음. 동음이의어처럼 '문맥(context)'에 뜻이 달라질 수 있는 상황을 고려하지 못함. subword information이 무시됨('서울', '서울시' 두 단어를 서로 독립된 두 단어로 취급함). 단어의 DF가 낮을 경우(corpus에서 등장 빈도 자체가 적을 경우) 유사한 단어를 찾지 못함

<br>

#### 1.2.2 &nbsp; Glove(Global vector) - 추론 기반 + 통계 기반 기법

- 특징: word2vec과 같은 방법은 window size내의 맥락들만 고려 할 수 있고 corpus의 전역적인 특성을 고려하지 못한다는 단점이 존재. 따라서 학습시 corpus 전체의 통계 정보(동시 발생 행렬, co-occurence matrix)를 손실함수에 도입하여 이와 같은 단점을 해소하고자 함
- 장점: ?
- 단점: 여전히 OOV 단어를 처리할 수 없음. 단어의 문맥에 따른 의미 변화를 반영할 수 없음.

<br>

#### 1.2.3 &nbsp; Fasttext

- 특징: facebook research에서 공개함. word2vec과 유사하지만, 그 과정에서 단어를 n-gram으로 나누어서 학습을 진행한다는 점에서 차이점이 존재함.
- 장점: OOV에 대응할 수 있음(테스트 단어가 학습 시의 사전에 존재하면 해당 단어의 벡터를 그대로 리턴하지만, OOV일 경우 입력 단어의 n-gram 벡터들의 합을 리턴함). OOV에 대응할 수 있다는 것은 학습시 사용되지 않았지만 오탈자가 없는 '정상적인 단어'에 대해서도 대응을 할 수 있다는 것 외에도 '오탈자'에 대해서도 대응을 할 수 있다는 것을 의미함. subword information을 고려할 수 있기 때문에 DF가 낮은 단어도 word2vec에 비해 유사한 단어를 잘 찾을 수 있음
- 단점: 여전히 단어의 문맥에 따른 의미의 변화를 고려할 수 없음

<br>

## 2 &nbsp; contextualized representations

context-free representation 방법론은 공통적으로 한 단어의 의미가 항상 고정(stable)하다고 보는 큰 한계점이 있었다. 다시말해 동음이의어인 단어를 여러가지 의미로 보는 것이 아니라 단어의 '형태'가 같으면 같은 의미로 파악하는 것이다. 이것을 개선하기 위해서 언어모델(language model)에 기반한 contextualize representation 방법론이 등장하게 되었다.

NLP task의 데이터셋의 크기는 높은 성능을 얻기 위한 재료의 관점에서 볼 때, 그 사이즈가 부족한 것이 현실인데 이 때문에 NLP task에 사용할 neural network를 깊게 디자인 하기 힘들다(작은 데이터셋에 깊은 신경망을 쓰면 overfit이 발생하고 일반화 성능을 얻기 힘듦). 반면에 vision task의 경우 대규모의 학습 데이터셋인 ImageNet이 공개 되어 있다. 따라서 자신의 실제 task에 적용시키기 전에 pre-training을 통해 일반적인 이미지 feature를 추출는 방법을 모델로 하여금 학습하도록 하고, 실제 task에서는 소규모의 데이터셋으로 fine-tuning을 통해 높은 성능을 끌어낼 수 있었다.

NLP task에서도 소규모 데이터셋을 이용하면서도 높은 성능을 이끌어 내기 위해 pre-training 방법론이 제안되어져 왔다. 이는 언어 모델(language model)을 통해서 실현이 가능하다.

NLP task를 진행하는 전통적인 방법은 word embedding 기술을 활용하여 학습을 수행할 모델의 input값을 초기화 시킨 후 모델의 학습을 진행하는 것이었다면, 최근의 NLP task의 흐름은 pre-trained language model(문장 내에서 단어들의 관계성을 high-level 수준으로 이미 알고 있는 모델)을 활용하여 마지막 레이어만 특정 task에 맞게 수정하여 사용하는 방식으로 바뀌었다.

### 2.1 &nbsp; ELMo(Embeddings from Language Models)

문장에 등장하는 단어는 그 문맥에 따라 다른 의미를 지니고 있는 경우가 많다. 예를 들어 '사과'는 '사과가 맛있다.'라는 문장에서는 '과일'을 의미하며, '그녀에서 나의 잘못을 사과하였다.'라는 문장에서는 '자기의 잘못을 인정하고 용서를 빎'이라는 의미이다. 이처럼 자연어는 형태는 같지만 다양한 의미를 지니고 있다. 다시 말해서 해당 단어가 어떤 **문맥** 에서 사용되었는지에 따라 그 의미가 다르다는 것이다. 그러나 앞서 언급한 것처럼 그간 word prepresentaion 방법은 어떤 문맥에서 단어가 상관되었는지에 관계없이 **고정된 하나의 벡터**만을 가질 수 있었다. 

ELMo는 그간의 word representation이 문맥을 반영하지 못한다는 한계점을 지적하며, 문맥에 따라 representation을 다르게 하여 context를 반영한 representation 방법을 제안한다. ELMo는 NLP의 전이학습의 시작점을 제공한 모델로서, bi-directional LSTM 모델을 통해 다음에 올 토큰을 예측하는 언어 모델(language model)이다.

<br>

### 2.2 &nbsp; ULMFiT(Universal Language Model Fine-tuning)

<br>

### 2.3 &nbsp; Transformer

NMT(neural machine translation) 문제를 해결하기 위해 고안된 모델로서 기존의 NMT에서 주로 사용되던 LSTM이 가지는 병렬연산 불가 문제를 해결하였다. seq2seq 모델과 마찬가지로 encoder와 decoder 구조로 이루어져 있지만, RNN 계열의 cell을 사용하지 않고 이를 transformer로 대체하였다. 특히 self-attention 구조를 제안한 것이 특징이다. 기존의 seq2seq with attention 모델의 경우에는 source sentence를 target sentence로 변환하는 과정에서 target word와 source word 관계에 대해서만 설명할 수 있었고(디코더의 출력이 인코더의 어느 부분에 주목한 결과인지 알 수 있음). 그러나 source word 간의 관계에 대해서를 설명할 수 없는 단점이 있었다. 예를 들어 "나는 배가 고파서 사과를 먹었는데, 그것은 상해 있었다."라는 문장에 대해서 '사과'와 '그것'의 관계를 주목할 수 없었다는 것이다. transformer는 self-attention 구조를 통해서 이를 해결하였다. 이후에 등장하는 GPT, BERT는 NMT 문제에 적용된 transfomer를 어떻게 downstream task에 적용 할 수 있을지를 고민하여 탄생한 모델이다. 

<br>

### 2.4 &nbsp; GPT(Generative Pre-Training Transformer)

transfomer의 decoder 부분만 활용하였다.

<br>

### 2.5 &nbsp; BERT(Bidirectional Encoder Representation from Transformer)

지금까지 transformer 기반의 모델은 ELMo가 제안하였던 bi-directional 방법을 고려하지 못하였다. BERT는 transformer의 encoder만을 활용한 모델인데, 인풋 토큰의 일부를 가리는 mask 토큰을 삽입하여 모든 문맥을 고려한(mask 토큰의 앞과 뒤) mask language model이다. 이를 통해 transformer와 ELMo의 장점을 모두 취할 수 있었다.

<br>

## 참고자료

- 김성현, T아카데미 토크ON 세미나 - 딥러닝 기반의 자연어 언어 모델 BERT
- [From Word Embeddings to Pretrained Language Models — A New Age in NLP](<https://towardsdatascience.com/from-word-embeddings-to-pretrained-language-models-a-new-age-in-nlp-part-1-7ed0c7f3dfc5>)
- [The Illustrated BERT, ELMo, and co. (How NLP Cracked Transfer Learning)](