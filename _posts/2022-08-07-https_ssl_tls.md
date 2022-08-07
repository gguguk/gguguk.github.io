---
title: HTTPS의 동작 원리와 그 구성 요소들
author: Gukwon Koo
categories: [CS, Network]
tags: [https, ssl, tls, ca, certificate]
pin: false
math: true
comments: true
date: 2022-08-07 10:28:00 +0900
---

최근 회사에서 MLOps 업무를 진행 중입니다. 그 과정에서 HTTPS 연결을 셋팅해야 하는 일이 있는데, 이와 관련된 개념들이 헷갈리고 혼재된 것들이 스스로 많다고 느껴서 이참에 정리 해보려고합니다. 참고로 글을 나눌까 하다가... 어짜피 다 관련된 내용들이라 한 글에 다 정리하였습니다. 따라서 글이 좀 길 수는 있습니다만 관심 있으신 분들이라면 천천히 읽어 보시길 추천드립니다. 🐢

본 글에서는 다음의 내용을 다룹니다.

- HTTPS
- SSL/TLS
- 인증 기관(CA) 및 인증서(SSL Cerificate)
- 대칭키 및 비대칭키(공개키, 개인키)
- SSL Handshake

<br>



# 네트워크에서는 아무도 믿지 마라!

---

HTTPS와 SSL/TLS는 네트워크에서 그 누구도 신뢰할 수 없다는 전제를 바탕으로 합니다. 우리는 거의 매일 접속하는 네이버, 구글 등의 사이트에 접속해서 많은 일을 합니다. 예를 들어 네이버에 접속하기 위해서 우리는 브라우저에 [https://www.naver.com](http://www.naver.com)을 입력하고 엔터를 누르게 됩니다. 그리고 초록색 화면이 잔뜩한 사이트의 메인 페이지가 브라우저에 뜨게 되죠. 여러분이 보고 계신 화면은 진짜 우리가 접속을 의도 했던 ‘네이버'가 맞나요? 어떻게 신뢰할 수 있을까요?

당연한 그 사이트는 진짜 내가 접속하려고 했던 네이버가 맞습니다. 그러면 어떻게 브라우저는 진짜 네이버가 맞는지를 신뢰하고 나에게 그 화면을 보여주는 것일까요? 바로 이 '신뢰'를 형성하는 네트워크 기술이 HTTPS, SSL/TLS 이라고 할 수 있습니다. 그리고 이들은 네트워크의 '보안'을 향상시키기 위한 장치들입니다.

<br>

# 헷갈리는 용어 정리

---

### HTTP vs HTTPS

HTTP는 하이퍼텍스트(HTML) 파일을 주고 받을 때 사용하는 통신 규약입니다(HTTP에 관한 자세한 이야기는 좋은 문서들이 많으니 생략하겠습니다). 브라우저에 http://www.naver.com을 입력하는 행위는 http를 활용하여 네이버와 통신하겠다는 의미입니다.

그럼 HTTPS는 무엇일까요? 여러 자료를 참고해 보면 HTTP + {Secure, Over SLL, Over TLS} 등으로 다양한 용어가 혼재되어 있습니다. 하지만 보안이 강화된 HTTP를 의미한다는 것이 본질적 내용입니다. HTTPS는 SSL/TLS 프로토콜 위에서 동작하는 HTTP의 보안이 강화된 버전이라고 할 수 있습니다.

### SSL VS TLS

SSL(Secure Socket Layer)과 TLS(Transport Layer Security)은 본질적으로 같은 용어입니다. SSL은 1994년 넷스케이프에서 처음 제안하였고, 이후에 표준화 관리 기구인 IETF(국제 인터넷 표준화 기구)에서 표준화 하면서 정의한 용어가 TLS입니다. 따라서 TLS와 SSL은 버전이 다른 정도라고 이해하면 되겠습니다.

<br>

# CA(Certificate authority)

---

적어도 우리들에겐 네이버(https://www.naver.com)는 믿을만한, 신뢰할만한 웹사이트입니다. 그러나 네트워크에는 셀수 없이 많은 웹사이트가 존재할텐데, 이 중에는 개인 정보를 탈취하기 위해 만들어진 웹사이트도 있을 것입니다. 따라서 브라우저는 유저가 접속하려는 웹사이트가 신뢰할만한 곳인지 아닌지를 알아내는 것부터가 보안의 시작이라고 할 수 있겠습니다.

웹사이트의 신뢰성을 인정해주는 민간 기업들이 있는데 이러한 기업들을 CA라고 합니다. CA에는 계층 구조가 있는데 가장 최상위의 CA를 루트 인증 기관(Root CA)라고 하며, 그 밑에 있는 CA들을 중간 인증 기관(intermediate CA)라고 합니다.

SSL/TLS를 이용하여 암호화된 통신 방식을 활용하고 싶은 웹사이트는 이러한 CA들을 통해서 ‘인증서'를 구입하여야 합니다. CA들은 해당 웹사이트를 다각도로 평가하고 ‘인증서'를 발급해줍니다. 이 인증서가 있는 웹사이트는 신뢰할만한 기관에서 보안과 신뢰성을 보장해준 것이라고 할 수 있습니다.

각 브라우저는 공인된 CA의 리스트와 해당 CA의 공개키를 미리 내장하고 있습니다. 이는 인증서가 CA측의 개인키로 암호화 되어 있기 때문에, 인증서의 내용을 읽기 위해서는 CA의 공개키가 필요하기 때문입니다. 공개키는 아래에서 자세히 설명할 것이며, 지금은 그냥 CA가 가진 시크릿(secret)이라고 생각하고 넘어가 주세요.

<br>

# SSL 인증서(SSL Certificate)

---

![네이버의 인증서 내용](/assets/img/post_img/ssl_certificate_example.png){:width="450"}_네이버에 접속 했을 때 인증서 내용_

그렇다면 인증서에는 어떤 내용이 적혀 있을까요? 다음의 내용이 포함되어 있습니다.

- 웹사이트의 정보
  - 인증서를 발급한 CA, 도메인 주소
  - 웹사이트의 신뢰성 검증을 위한 용도
- **웹사이트의 공개키 정보**
  - 공개키 내용, 공개키 암호화 방법
  - 클라이언트와 서버가 데이터를 주고 받을 때의 보안과 관련된 사항

도메인 주소나 공개키는 서버측에서 CA에서 인증서를 받고자 할 때 제출해야하는 정보입니다. CA는 웹사이트와 웹사이트의 공개키 정보를 CA측의 개인키로 암호화 합니다. 이것이 바로 인증서의 실체입니다. 위의 이미지는 해당 인증서를 CA의 공개키로 복호화한 내용이라고 할 수 있겠네요. **그리고 인증서에는 웹사이트의 공개키가 포함 되어 있다는 사실을 꼭 기억해주세요!** 이후 설명할 SSL handshake 단계를 이해하기 위해 꼭 필요한 내용입니다.

<br>

# 대칭키 및 비대칭키

---

CA 및 인증서의 내용을 다루다보니 계속해서 ‘공개키'라는 단어가 등장했습니다. 이를 이해하기 위해서는 암호화 방식인 대칭키 및 비대칭키에 대한 개념적인 이해가 필요합니다

- 대칭키(symmetric cryptography): 암호화와 복호화에 같은 비밀키를 사용 하는 방식
- 비대칭키(asymmetric cryptography): 암호화와 복호화에 다른 비밀키를 사용하는 방식

> 참고로 앞으로 계속 등장할 비밀키란 일종의 랜덤한 숫자나 문자입니다. 평문(plain text)과 비밀키로 어떤 연산을 수행하면 암호문(ciphertext)을 얻을 수 있습니다. 어떤 암호화 방법이든지 이를 구현하는 알고리즘의 종류는 매우 많고 이에 따른 동작 방식도 다릅니다. 

### 대칭키

![대칭키 암호화 방식](https://upload.wikimedia.org/wikipedia/commons/2/27/Symmetric_key_encryption.svg)_대칭키 암호화 방식_

대칭키 암호화 방식은 암호화와 복호화에 동일한 비밀키를 사용합니다. 예를 들어 보죠. Bob이 Alice에서 ‘Hello Alice!’라는 평문(plain text)을 암호화 해서 보내고 싶다고 해봅시다. 그리고 Bob과 Alice는 암호화에 쓰인 비밀키를 서로 공유하고 있습니다. 따라서 Bob은 이 비밀키를 활용하여 평문을 암호화한 후 Alice에게 보냅니다. Alice는 암호화에 사용된 비밀키를 가지고 있기 때문에 이를 평문('Hello Alice!')으로 복호화 할 수 있습니다. 대표적인 대칭키 알고리즘은 AES(Advanced Encryption Standart), DES(Data Encryption Standart) 등이 있습니다.

현대의 암호화 기법은 매우 복잡하여 암호문을 보고 평문을 알아내는 것은 사실상 불가능하다고 합니다. 그렇다면 대칭키 암호화 기법은 항상 안전하다고 할 수 있을까요? 그렇지 않습니다. 만일 암호화와 복호화에 사용되는 비밀 자체를 누군가가 탈취하면 암호화를 한 의미가 없어지겠죠. 그러나 모순적으로 대칭키 방식이 정상적으로 이루어지려면 비밀키(key)의 '공유'가 선행되어야 합니다. 따라서 이제는 이 비밀키(key)를 어떻게 하면 내밀하게 공유할 수 있을지를 고민해 봐야 하겠습니다.

<br>

### 비대칭키

![비대칭키 암호화 방식](https://upload.wikimedia.org/wikipedia/commons/f/f9/Public_key_encryption.svg)_비대칭키 암호화 방식_

우리는 대칭키 암호화 방식을 살펴보면서, 비밀키 그 자체를 어떻게 내밀하게 공유할 수 있을까에 대한 의문을 가지게 되었습니다. 그런데 이렇게 한번 생각 해보면 어떨까요? **같은** 비밀키(key)를 공유해서 문제가 발생하는 것이니 서로 **다른** 두 개 비밀키를 만드는 겁니다. 그래서 하나는 누구에게든 공유하되, 나머지 하나는 비밀키를 만든 사람만 가지고 있는 것이죠. 그리고 공유 받은 키로 암호화한 내용은 무조건 비밀키를 만든 사람만이 가지고 있는 키로만 복호화를 할 수 있는 메커니즘을 설계하는 겁니다. 이를 공개키(public key) 및 개인키(private key) 기반의 비대칭키 암호화 방식이라고 하고 가장 대표적으로 RSA 알고리즘을 들 수 있습니다.

- 누구에게나 공유할 수 있는 비밀키: 공유키(public key)라고 합니다.
- 만든 사람만 소유하는 비밀키: 개인키(private key)라고 합니다.

비대칭키의 작동 원리상 핵심은 공개키로 암호화 하면 이에 대응하는 개인키로만 복호화되고, 개인키로 암호화 하면 이에 대응되는 공개키로만 복호화가 가능합니다. 즉 서로 양방향으로 암호화 복호화가 가능한데, 어떤 키로 암호화 하느냐에 따라 사용 분야가 다릅니다:

- 공개키로 암호화: 데이터 보안에 중점을 둡니다. 공개키로 암호화된 데이터는 개인키를 가진 사람만이 복호화를 할 수 있기 때문입니다. 개인키만 공유하지 않는다면 공개키로 암호화된 내용을 중간에서 누군가 탈취하더라도 평문으로 복호화 하는 것이 불가능합니다. 개인키를 공유하지 않기 때문에 대칭키 방식의 단점을 극복할 수 있습니다.
- 비밀키로 암호화: 사용자 인증(authentication)에 중점을 둡니다. 비밀키로 암호화된 데이터는 공개키를 가진 불특정 다수가 복호화를 할 수 있습니다. 따라서 비밀키로 암호화하는 것은 데이터의 보호 목적이 아닙니다. 개인키는 공유하는 키가 아니기 때문에 원칙적으로는 개인키와 사람은 일대일 대응 관계입니다. 따라서 누군가가 보내온 메시지를 그 사람의 공개키로 복호화에 성공 했다면 이는 세상에 하나뿐인 개인키를 가진 사람이 보내온 메시지이므로 이 사람의 신원을 확인할 수 있습니다. 이를 공인 인증 체계의 바탕이 되는 전자 서명이라고 합니다.

비대칭키의 메커니즘을 통해 대칭키 방식의 보안상 문제점을 완벽하게 해결할 수 있습니다.

<br>

### 대칭키와 비대칭키의 결합

지금까지 대칭키와 비대칭키의 개념을 살펴보았는데, 여기까지만 보면 비대칭키만 사용하면 될 것 같습니다. 대칭키 방식은 너무 구식처럼 보이네요. 그러나 HTTPS 및 SSL/TLS는 대칭키과 비대칭키 방식을 결합해서 사용합니다. 사실 대칭키과 비대칭키는 각각의 장단점이 명확해서 상호 보완적인 관계입니다.

- 대칭키 방식: 대칭키는 보안에 취약하지만 알고리즘 작동 방식이 단순하여 암호화/복호화 속도가 빠릅니다.
- 비대칭키 방식: 보안이 강력하지만 알고리즘 작동 방식이 복잡하여 암호화/복호화 속도가 느립니다.

그렇다면 이렇게 생각 해보죠. 실제 데이터를 주고 받을 때는 대칭키 방식을 이용하여 암호화/복호화를 하는 겁니다. 데이터를 주고 받는 속도가 느리면 안되니까요. 그러면 서로 간에 키를 공유해야 하기 때문에 보안에 문제가 생기죠. 이때 비대칭키 방식을 적용합니다. 대칭키 자체를 비대칭키 방식으로 암호화 하는 겁니다. 헷갈릴 수 있는데, 대칭키 자체를 암호화/복호화의 대상으로 보고 대칭키에 대한 공개키, 개인키를 만듭니다.

- 데이터: 대칭키를 이용하여 암호화/복호화
- 대칭키 자체: 대칭키에 대한 공개키, 개인키를 만들어서 암호화/복호화

<br>

# SSL Handshake

---

![ssl-handshake](https://cf-assets.www.cloudflare.com/slt3lc6tev37/5aYOr5erfyNBq20X5djTco/3c859532c91f25d961b2884bf521c1eb/tls-ssl-handshake.png)_SSL handshake 과정. 이 전에 TCP 3-way handshake가 수행된다._

이제 우리는 SSL/TLS 프로토콜을 이용하여 암호화 통신을 수행할 때 필요한 배경 지식을 갖추게 되었습니다. 이제 암호화 통신이 실제로 일어나는 과정을 살펴 볼 시간입니다!

> 참고로 SSL handshake는 TCP 3-way handshake을 통해 서로간에 연결이 수립된 이후에 진행됩니다. 본 글에서는 TCP 3-way handshake 과정에 대한 설명은 생략 하겠습니다.

**1. Client Hello**

- 클라이언트(브라우저) 측에서 서버에 SSL 통신을 시작하는 것을 알리는 단계입니다. 다음의 데이터를 서버로 보냅니다:
  - **클라이언트 랜덤 데이터(client random data)**: 대칭키를 만들기 위해 필요한 재료입니다. 뒤에서 다시 살펴보겠습니다.
  - 클라이언트가 지원하는 TLS 버전
  - 클라이언트가 지원하는 암호화 방식들(chiper suite)

**2. Sever Hello**

- 서버(웹사이트)에서 클라이언트에 응답하는 단계입니다. 서버는 다음의 데이터를 클라이언트로 보냅니다:
  - **인증서(SSL certificate)**: 서버의 신뢰성을 검증 받기 위해 CA에게 발급 받은 인증서를 건냅니다. 인증서에는 서버의 공개키가 포함되어 있습니다. 따라서 서버의 공개키도 클라이언트측에 넘어갑니다.
  - **서버 랜덤 데이터(server random data)**: 대칭키를 만들기 위해 필요한 재료입니다. 뒤에서 다시 살펴보겠습니다.
  - 서버가 선택한 암호화 방식: client hello 단계에서 전달받은 암호화 방식 중에서 가장 보안 수준이 높은 암호화 방식을 선정합니다. 이를 통해 암호화 방식에 대한 협상을 종료합니다.

**3. 인증(Authentication)**

- 클라이언트가 서버의 신뢰성을 검증하는 단계입니다.
- 다음의 사항을 검증합니다:
  - 인증서를 발급한 CA가 공인된 CA지를 확인하기 위해 브라우저에 내장된 CA 리스트에서 찾아봅니다. 만일 공인된 CA가 발급한 인증서가 아니라면 브라우저에 신뢰 할수 없는 인증서라는 경고 메시지를 띄웁니다.
  - 브라우저에 내장된 CA의 공개키로 인증서의 내용을 복호화 합니다. 그리고 인증서에 적힌 도메인 주소 등의 서버와 관련된 정보를 확인하여 연결을 수립하려는 서버의 진위 여부를 검증합니다.

**4. 세션키(session key) 생성**

- 클라이언트는 서버를 신뢰할 수 있게 되었습니다. 이제 데이터를 암호화 하여 주고 받기 위해 세션키를 생성를 합니다.
- 다음의 과정을 거칩니다:
  - 클라이언트는 클라이언트 랜덤 데이터와 서버 랜덤 데이터를 알고 있습니다. 이 둘을 조합하여 pre master secret이라는 비밀키를 생성합니다. 그리고 이를 서버측에 공유해야 합니다. 이때 드디어 인증서에 포함된 서버의 공개키를 활용합니다. 서버의 공개키로 pre master secret을 암호화 하여 서버에 전달합니다.
  - 서버는 개인키로 암호화된 pre master secret을 복호화 합니다.
  - 이제 클라이언트와 서버는 client random data, sever random data, pre master secret를 모두 공유하고 있는 상태입니다. 각자는 이 세가지 데이터를 조합하여 master secret이라는 키를 만들고 이를 통해 최종적으로 session key를 생성합니다. 이 session key는 서로 동일하게 가지고 있는 대칭키입니다.

**5. SSL Handshake 종료**

- 서버와 클라이언트가 SSL handshake가 종료 되었음을 서로에게 알립니다.

**6. session key를 이용한 통신**

- 4단계에서 만들어진 session key를 통해 서로간에 데이터를 주고 받을 때 암호화 및 복호화를 수행합니다.
- session key는 대칭키이므로 암호화 및 복호화 두가지 모두 수행할 수 있습니다.

<br>

# 최종 한장 정리

---

지금까지 정말 많은 내용을 학습 했습니다. 대칭키, 비대칭키(공개키, 개인키), 인증기관(CA), 인증서, SSL/TLS Handshake 등이요. 그리고 이러한 구성요소들위에서 HTTP 통신을 하는 것이 결국 HTTPS를 의미하는 것임을 알 수 있었습니다.

이 과정을 한눈에 알기 쉽게 정리해주신 자료([미닉스 김인성님 블로그](https://minix.tistory.com/397?category=429082))가 있어서 가져와 보았습니다.

![SSL 한장에 정리](/assets/img/post_img/SSL_Overview.png){:width="700"}_SSL 한장에 정리!_

<br>

# References

---

- [What is SSL? - SSL definition](https://www.cloudflare.com/learning/ssl/what-is-ssl/)
- [What is an SSL certificate?](https://www.cloudflare.com/learning/ssl/what-is-an-ssl-certificate/)
- [What is a session key? - Session keys and TLS handshakes](https://www.cloudflare.com/learning/ssl/what-is-a-session-key/)
- [How does SSL work? - SSL certificates and TLS](https://www.cloudflare.com/ko-kr/learning/ssl/how-does-public-key-encryption-work/)
- [How does public key encryption work? - Public key cryptography and SSL](https://www.cloudflare.com/ko-kr/learning/ssl/how-does-public-key-encryption-work/)
- [What happens in a TLS handshake? - SSL handshake](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)
- [위키백과 - HTTPS](https://ko.wikipedia.org/wiki/HTTPS)
- [위키백과 - RSA 암호](https://ko.wikipedia.org/wiki/RSA_암호)
- [정보 통신 기술 용어 해설 - SSL/TLS](http://www.ktword.co.kr/test/view/view.php?m_temp1=1957)
- [대칭키 암호 알고리즘](http://wiki.hash.kr/index.php/대칭키_암호_알고리즘)
- [공개키, 비공개 키 원리](https://brunch.co.kr/@artiveloper/24)
- [웹툰 SSL이란 무엇인가 1/2](https://minix.tistory.com/395)
- [웹툰 SSL이란 무엇인가 2/2](https://blog.daum.net/haha25/5390206?np_nil_b=-3&categoryId=228508)
- [HTTPS 통신과정 쉽게 이해하기 #1(HTTPS)의 개요](https://aws-hyoh.tistory.com/34)
- [HTTPS 통신과정 쉽게 이해하기 #2(Key가 있어야 문을 열 수 있다)](https://aws-hyoh.tistory.com/38)
- [HTTPS 통신과정 쉽게 이해하기 #3(SSL Handshake, 협상)](https://aws-hyoh.tistory.com/39)
- [HTTPS 통신과정 쉽게 이해하기 #4(Cipher Suite, 암호문의 집합)](https://aws-hyoh.tistory.com/47)
- [HTTPS 통신과정 쉽게 이해하기 #5(CA, 인증기관)](https://aws-hyoh.tistory.com/59)