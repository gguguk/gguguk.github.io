---
title: Byte Pair Encoding
author: Gukwon Koo
categories: [ML, NLP]
tags: [nlp, byte pair encoding]
pin: false
math: true
comments: true
---

> 최근 NLP에서 tokenizer로 많이 사용되고 있는 BPE에 대해서 간단하게 정리해 보겠습니다. 전체코드는 [이곳](https://github.com/gwkoo82/tutorials/blob/master/nlp/BPE_generate_vocab_tutorial.ipynb)에서 확인해 보실 수 있습니다.

## 1 &nbsp; Backgroud: Subword Segmentation
subword segmentation(단어 분리, 단어 분절)이란, 하나의 단어(혹은 토큰)는 여러 개의 subword의 조합으로 이루어져 있다는 가정하에 subword 단위의 토크나이징을 수행하여 단어를 이해하려는 목적을 가진 전처리 작업을 말합니다. 이어서 설명할 BPE는 subword segmentation 기법 중 하나입니다.

### 1.1 &nbsp; Example
실제로 영어나 한국어는 subword 단위로 의미를 분리할 수 있는 경우가 많습니다. 아래의 예시를 보시면 그 의미를 쉽게 이해할 수 있습니다.

|언어|단어|subword 구성|
|:---|:---|:---|
|영어|concentrate|con(=together) + centr(=center) + ate(=make)|
|한국어|집중|集(모을 집) + 中(가운데 중)|

<br>

## 2 &nbsp; BPE(Byte Pair Encoding)
BPE는 subword segmentation의 대표적인 알고리즘입니다. 'byte'라는 단어에서 알 수 있듯이 원래는 데이터 압축을 위한 알고리즘으로 탄생했지만, 현재는 NLP 분야의 대표적인 토크나이징 방법으로 활용되고 있습니다. 특히 최신 NLP 모델인 BERT 등에서 기본적인 토크나이저로 활용되고 있습니다.

### 2.1 &nbsp; Merits of BPE
- 단어 사전의 OOV 문제 완화: subword segmentation인 BEP의 최대 장점은 `OOV` 문제를 완화할 수 있다는 것입니다. 기존의 vocabulary에 없는 신조어나 오타 등도 subword 단위나 character 단위로 토크나이징 하면 known token의 조합으로 표현가능 합니다. 예를 들어 `athazagoraphobia`처럼 기존 단어 사전에 없던 단어는 `UNK`토큰이 아니라 `['▁ath', 'az', 'agor', 'aphobia']`와 같이 subword 단위로 쪼개서 OOV 문제를 회피할 수 있습니다[(Sennrich et al., 2015)](https://scholar.google.co.kr/scholar?hl=ko&as_sdt=0%2C5&q=Neural+machine+translation+of+rare+words+with+subword+units&btnG=).
  
- 언어에 관계없는 토크나이저 구축 가능: BPE 토크나이저는 각 언어의 특성에 적합한 사전 전처리 과정이 필요하지 않은 universal tokenizer입니다. 그러나 한국어의 경우 형태소 분석 후 BPE를 적용하는 것이 특정 태스크에서는 더 높은 스코어를 유도한다는 연구 결과도 있으므로([Park et al., 2020](https://scholar.google.co.kr/scholar?hl=ko&as_sdt=0%2C5&q=An+Empirical+Study+of+Tokenization+Strategies+for+Various+Korean+NLP+Tasks&btnG=)) 진리인 것은 아닌 것 같습니다.

### 2.2 &nbsp; vocabulary 생성 방법

BPE를 통한 토크나이징은 크게 **1) 사전 구축**, **2) 사전을 통한 토크나이징**으로 구성되어 있습니다. 본 글에서는 사전 구축에 집중하여 살펴보겠습니다. BPE는 형태소 분석을 통한 사전 구축 방법과 다르게 사용자가 학습 횟수를 임의로 조정하여 사전의 크기를 조절할 수 있는 특징이 있습니다. 사전 구축은 다음의 프로세스를 통해 이루어집니다. 먼저 논문에서는 BPE 토크나이저 구축 프로세스를 아래와 같이 언급하고 있습니다[(Sennrich et al., 2015)](https://scholar.google.co.kr/scholar?hl=ko&as_sdt=0%2C5&q=Neural+machine+translation+of+rare+words+with+subword+units&btnG=).

> - Firstly, we initialize the symbol vocabulary with the character vocabulary
> - We iteratively count all symbol pairs and replace each occurrence of the most frequent pair (‘A’, ‘B’) with a new symbol ‘AB’
> - Each merge operation produces a new symbol which represents a character n-gram. 
> - Frequent character n-grams (or whole words) are eventually merged into a single symbol, thus BPE requires no shortlist. 

이 내용을 조금 더 자세히 풀어보면...

**Step 1.** &nbsp; corpus 내의 sentence들을 어절 단위로 분리해 둡니다.

**Stpe 2.** &nbsp; 어절 단위로 분리된 token들을 character 단위로 분리(띄어쓰기 처리) 및 딕셔너리 생성(최종 vocabulary를 만들기 위한 기준으로 사용). 이때 각 word의 끝에는 `</w>` 토큰을 붙여줍니다.

**Stpe 3.** &nbsp; 가장 많이 등장한 pair를 찾아 merge한 후 딕셔너리를 업데이트 합니다.

**Stpe 4.** &nbsp; Step 3을 사용자가 지정한 횟수만큼 반복합니다(sentencepiece의 경우에는 최종 vocab size를 지정합니다)

**Stpe 5.** &nbsp; 완성된 딕셔너리를 통해 unique token만 남은 vocabulary를 생성

**Stpe 6.** &nbsp; final vocab size = # of initial vocab size(special token 포함) + # of merge operations

참고로 step 2에서 붙여주는 `</w>` 토큰은 생각보다 중요한 역할을 담당할 수 있습니다. 예를 들어 `est` 토큰을 생각해봅시다. 이 토큰은 `est ern`에 속할 수도 있고 `small est`에 속할 수도 있습니다. 이때 `est` 각 상황 마다 `est`토큰의 의미는 조금 다르게 느껴집니다. 만일 `est`가 word의 마지막에 등장하여 `est<\w>`로 처리한다면, 모델은 `est`의 의미를 좀더 명확히 인지할 수 있게 됩니다 [6].

<br>

## 3 &nbsp; BEP with python codes

corpus는 아래와 같이 주어졌다고 가정하고 시작하겠습니다.

``` python
corpus = [
    '이 영화를 영화관에서 직접 보지 못한 것을 평생 후회할 것 같다. 보는 내내 감탄을 자아냈고 절대 잊지 못할 여운으로 남을 것이다.',
    '평점이 이해되지 않는다 최소 9점은 받아야 한다 반전이 정말 끝내준다',
    '말로 표현할 수 없다 완벽한 영화 보는 내내 소름이 돋았다'
]
```



### 3.1 &nbsp; 어절 단위 분리 및 어절 내 띄어쓰기 처리

sentence를 어절 단위로 분리한 후, 어절 token 내에서 character 단위 분리를 한번에 수행해 보겠습니다. 아래의 함수는 그러한 역할을 수행합니다.

``` python
def get_dictionary(corpus):
    """ 데이터를 읽어와 단어 사전 구축
    Args:
        corpus: 여러 텍스트 데이터가 담긴 리스트
    Returns:
        dict(vocab): 문장 데이터 별 character 단위로 구분된(띄어쓰기) 후보 사전
    """
    dictionary = defaultdict(int)
    for line in corpus:
        tokens = preprocess(line).strip().split(" ")
        for token in tokens:
            dictionary[" ".join(list(token)) + " </w>"] += 1
    
    return dict(dictionary)
```
결과로 아래와 같은 `dictionary`가 생성됩니다.

``` python
dictionary = get_dictionary(corpus)
print(dictionary)

{'이 </w>': 1,
 '영 화 를 </w>': 1,
 '영 화 관 에 서 </w>': 1,
 '직 접 </w>': 1,
 '보 지 </w>': 1,
 '못 한 </w>': 1,
 '것 을 </w>': 1,
 '평 생 </w>': 1,
 '후 회 할 </w>': 1,
 '것 </w>': 1,
 '같 다 </w>': 1,
 '보 는 </w>': 2,
 '내 내 </w>': 2,
 '감 탄 을 </w>': 1,
 ...}
```



### 3.2 &nbsp; 가장 많이 등장한 character pair 찾기

dictionary에서 가장 많이 등장한 bi-gram pair를 찾아보겠습니다. 아래의 함수는 dictionary내의 bi-gram pair를 count하는 함수입니다.
``` python
def get_pairs(dictionary):
    """ 딕셔너리를 활용한 바이그램 페어 구축
        Args: 
            dictionary: 어절 token 별 character 단위로 분리된 dictionary
        Returns:
            pairs: bi-gram pair
    """
    pairs = defaultdict(int)
    for word, freq in dictionary.items():
        word_lst = word.split()
        for i in range(len(word_lst)-1):
            pairs[(word_lst[i], word_lst[i+1])] += freq
            
    return dict(pairs)
```
함수의 실행 결과는 다음과 같습니다.
``` python
pairs = get_pairs(dictionary)
print(pairs)

{('이', '</w>'): 4,
 ('영', '화'): 3,
 ('화', '를'): 1,
 ('를', '</w>'): 1,
 ('화', '관'): 1,
 ('관', '에'): 1,
 ('에', '서'): 1,
 ('서', '</w>'): 1,
 ('직', '접'): 1,
 ('접', '</w>'): 1,
 ('보', '지'): 1,
 ('지', '</w>'): 3,
 ('못', '한'): 1,
 ('한', '</w>'): 2,
 ('것', '을'): 1,
 ('을', '</w>'): 3,
 ('평', '생'): 1,
 ...}
```
특히 가장 많이 등장한 bi-gram pair를 찾기 위해서는 max 함수에 key값을 지정해 주면 됩니다.
``` python
print(max(pairs, key=pairs.get))

('다', '</w>')
```



### 3.3 &nbsp; best bi-gram pair를 merge하고 dictionary 업데이트

가장 많이 등장한 bi-gram pair를 dictionary에서 찾아서 merge(공백 제거)를 진행하는 함수입니다.
``` python
def merge_dictionary(pairs, dictionary):
    """ 가장 자주 등장한 bi-gram pair를 merge(공백 제거)
    Args:
        pairs: bi-gram pair
        dictionary: 문장 데이터 별 character 단위로 구분된(띄어쓰기) 후보 사전
    Returns:
        dict(result): 가장 자주 등장한 bi-gram pair를 merge한 dictionary
    """
    result = defaultdict(int)
    best_pair = max(pairs, key=pairs.get)
    for word, freq in dictionary.items():
        paired = word.replace(" ".join(best_pair), "".join(best_pair))
        result[paired] = dictionary[word]
    return dict(result)
```
예를 들어 현재 상태의 dictionary와 pair를 통해 dictionary를 업데이트 할 경우 아래와 같은 결과를 얻을 수 있습니다. 

가장 많이 등장한 ('다', '</w\>')의 경우 공백이 제거된 모습을 볼 수 있습니다.
``` python
temp = merge_dictionary(pairs, dictionary)
print(temp)

{...
 '것 </w>': 1,
 '같 다</w>': 1,
  ...
 '돋 았 다</w>': 1}
```



### 3.4 &nbsp; 반복을 통한 dictionary 업데이트

이제 반복을 통한 dictionary 업데이트를 진행해 보겠습니다. 서두에서 언급 했듯이 반복 횟수는 사용자가 지정하는 하이퍼파라미터입니다. 반복 횟수를 통해 사전의 크기를 조절 할 수 있습니다.
```python
iter_nums = 20
dictionary = get_dictionary(corpus)
for i in range(iter_nums):
    pairs = get_pairs(dictionary)
    dictionary = merge_dictionary(pairs, dictionary)
```
그 결과 아래의 dictionary를 얻을 수 있습니다. 공백이 포함되었던 어절 token 중 일부에서 공백이 제거된 모습을 확인 하실 수 있습니다.

``` python
print(dictionary)

{'이</w>': 1,
 '영화를</w>': 1,
 '영화관에서</w>': 1,
 '직접</w>': 1,
 '보 지</w>': 1,
 '못 한</w>': 1,
 '것 을</w>': 1,
 '평 생 </w>': 1,
 '후 회 할</w>': 1,
 '것 </w>': 1,
 '같 다</w>': 1,
 '보는</w>': 2,
 '내내</w>': 2,
 '감 탄 을</w>': 1,
 '자 아 냈 고 </w>': 1,
 '절 대 </w>': 1,
 '잊 지</w>': 1,
 '못 할</w>': 1,
 '여 운 으 로</w>': 1,
 '남 을</w>': 1,
 '것 이 다</w>': 1,
 '평 점 이</w>': 1,
 '이 해 되 지</w>': 1,
 '않 는 다</w>': 1,
 '최 소 </w>': 1,
 '점 은 </w>': 1,
 '받 아 야 </w>': 1,
 '한 다</w>': 1,
 '반 전 이</w>': 1,
 '정 말 </w>': 1,
 '끝 내 준 다</w>': 1,
 '말 로</w>': 1,
 '표 현 할</w>': 1,
 '수 </w>': 1,
 '없 다</w>': 1,
 '완 벽 한</w>': 1,
 '영화 </w>': 1,
 '소 름 이</w>': 1,
 '돋 았 다</w>': 1}
```



### 3.5 &nbsp; 최종 vocabulary 생성

반복 횟수만큼 업데이트 된 dictionary를 기반으로 vocabulary를 생성해 봅시다. vocabulary는 dictionary를 띄어쓰기 단위로 분리한 뒤, 그중 unique한 값들만 취하면 됩니다.
``` python
def get_vocab(dictionary):
    """ dictionary에서 최종 vocabulary를 만드는 함수
    Args:
        dictionary: merge가 수행된 dictionary
    Returns:
        dict(vocab): 최종 vocabulary
    """
    result = defaultdict(int)
    for word, freq in dictionary.items():
        tokens = word.split()
        for token in tokens:
            result[token] += freq
    return dict(result)
```
마지막으로 생성된 vocabulary를 확인 해 볼까요?
``` python
vocab = get_vocab(dictionary)
print(vocab)

{'이</w>': 4,
 '영화를</w>': 1,
 '영화관에서</w>': 1,
 '직접</w>': 1,
 '보': 1,
 '지</w>': 3,
 '못': 2,
 '한</w>': 2,
 '것': 3,
 '을</w>': 3,
 '평': 2,
 '생': 1,
 '</w>': 10,
 '후': 1,
 '회': 1,
 '할</w>': 3,
 '같': 1,
 '다</w>': 7,
 '보는</w>': 2,
 '내내</w>': 2,
 '감': 1,
 '탄': 1,
 '자': 1,
 '아': 2,
 '냈': 1,
 '고': 1,
 '절': 1,
 '대': 1,
 '잊': 1,
 '여': 1,
 '운': 1,
 '으': 1,
 '로</w>': 2,
 '남': 1,
 '이': 2,
 '점': 2,
 '해': 1,
 '되': 1,
 '않': 1,
 '는': 1,
 '최': 1,
 '소': 2,
 '은': 1,
 '받': 1,
 '야': 1,
 '한': 1,
 '반': 1,
 '전': 1,
 '정': 1,
 '말': 2,
 '끝': 1,
 '내': 1,
 '준': 1,
 '표': 1,
 '현': 1,
 '수': 1,
 '없': 1,
 '완': 1,
 '벽': 1,
 '영화': 1,
 '름': 1,
 '돋': 1,
 '았': 1}
```
<br/>

## 4 &nbsp; 참고자료

- [월간 자연어 처리 - BPE tutorial](https://github.com/Huffon/nlp-various-tutorials/blob/master/korean-bpe.ipynb?fbclid=IwAR10PD96wB4GDMby5VxKABpkKqwY4tpVI_wdv9mweAPzO8I2lj6GDWs_q08)
- [Segmentation using Subword (Byte Pare Encoding, BPE)](https://kh-kim.gitbooks.io/pytorch-natural-language-understanding/content/preprocessing/bpe.html)
- [단어 분리하기(Byte Pair Encoding, BPE)](https://wikidocs.net/22592)
- [Sennrich, R., Haddow, B., & Birch, A. (2015). Neural machine translation of rare words with subword units. arXiv preprint arXiv:1508.07909.](https://scholar.google.co.kr/scholar?hl=ko&as_sdt=0%2C5&q=Neural+Machine+Translation+of+Rare+Words+with+Subword+Units&btnG=)
- [Byte Pair Encoding — The Dark Horse of Modern NLP](https://towardsdatascience.com/byte-pair-encoding-the-dark-horse-of-modern-nlp-eb36c7df4f10)
- [bpe](https://notebook.community/ethen8181/machine-learning/deep_learning/subword/bpe#Applying-Encodings)
- [Park, K., Lee, J., Jang, S., & Jung, D. (2020). An Empirical Study of Tokenization Strategies for Various Korean NLP Tasks. *arXiv preprint arXiv:2010.02534*.](https://scholar.google.co.kr/scholar?hl=ko&as_sdt=0%2C5&q=An+Empirical+Study+of+Tokenization+Strategies+for+Various+Korean+NLP+Tasks&btnG=)