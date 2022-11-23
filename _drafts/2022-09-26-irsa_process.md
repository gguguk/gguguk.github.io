---
title: IRSA의 원리를 파헤쳐보자 2 - OAuth2.0
author: Gukwon Koo
categories: [MLOps, Kubernetes]
tags: [kubernetes, k8s, oauth2.0, oidc, irsa]
pin: false
math: true
comments: true
date: 2022-11-22 20:04:00 +0900
---

IRSA의 원리를 파헤쳐보자 시리즈의 두번째 글입니다. 지난 시간에는 IRSA를 가능하게 하는 가장 근본적인 컴포넌트인 [Admission Webhook](https://gguguk.github.io/posts/admission_webhook/)에 대해서 살펴 보았습니다. 이후 시리즈 글에서 더 자세히 살펴 보겠지만 pod identity webhook의 mutating 결과로 파드에 어떤 token이 주어지게 됩니다. 이 token은 파드가 AWS의 특정 리소스와 통신하기 위한 임시 자격 증명을 발급 받는데에 사용됩니다. 

이러한 일련의 과정은 OIDC(OpenID Connect)라는 프로토콜의 개념에 기반하여 이루어집니다. 그러나 OIDC를 이해하기 위해서는 먼저 OAuth2.0를 이해해야 합니다. OIDC는 OAuth2.0 위에서 동작하는 프로토콜이기 때문입니다. 따라서 이번 글에서는 IRSA를 이해하기 위한 배경지식으로서 OAuth2.0을 (수박 겉핥기 식으로 🍉) 살펴보겠습니다

사실 공부하면서, IRSA가 완벽하게 OAuth2.0이나 OIDC를 따르는 것은 아니라는 생각이 들었습니다. 그래서 본 글을 정리할지 말지 고민을 많이 했는데, OAuth2.0과 OIDC에서 나오는 개념들이 많이 등장하기 때문에 본 주제를 정리하기로 하였습니다. 머신러닝 엔지니어로서 언젠간 반드시 도움이 될 것이라고 생각하며... 😭

IRSA의 원리를 파헤쳐보자 시리즈 - 

1. [IRSA의 원리를 파헤쳐보자 1 - K8S Admission Webhook](https://gguguk.github.io/posts/admission_webhook/)
2. IRSA의 원리를 파헤쳐보자 2 - OAuth2.0
3. IRSA의 원리를 파헤쳐보자 3 - OIDC
4. IRSA의 원리를 파헤쳐보자 4 - K8S Sevice Account와 Service Account Token Volume Projection

<br>

# 인증(authentication)과 인가(authorization)

![authentication_and_authorization](https://www.okta.com/sites/default/files/styles/1640w_scaled/public/media/image/2020-10/Authentication_vs_Authorization.png?itok=uBFRCfww)_인증과 인가의 차이(출처: [okta](https://www.okta.com/kr/identity-101/authentication-vs-authorization/))_

앞으로의 내용에 인증(authentication)과 인가(authorization)이라는 단어가 계속해서 등장할 것입니다. 인증과 인가는 한국어로 보면 큰 차이가 없어 보이는 단어인데요, 보안의 맥락에서는 엄연히 다른 개념입니다. 먼저 인증이란 사용자의 신원(identity)를 확인 및 검증 하는 행위입니다. 인증의 대표적인 예로는 아이디와 비밀번호를 통한 로그인이 있습니다. 한편 인가는 사용자에게 특정 리소스나 기능에 접근할 수 있는 권한 또는 권한의 범위(scope)를 부여하는 행위를 말합니다. 우리가 네이버에 로그인(인증)을 했다고 하더라도 다른 사람의 데이터에 접근할 수 있는 권한은 없습니다. 다시 말해 타인의 데이터에 접근하는 인가는 받지 못한 것입니다.

이어지는 글에서 더 자세히 설명하겠지만 인증과 인가는 각각 OIDC, OAuth2.0와 관련이 깊습니다. OIDC는 인증을 담당하는 프로토콜이며, OAuth2.0은 인가를 담당하는 프로토콜입니다. 그렇다고 해서 OIDC와 OAuth2.0이 아예 별개의 개념이진 않습니다. 오히려 서로는 매우 깊은 관련이 있습니다.

<br>

#  OAuth2.0

### OAuth2.0, 왜 필요할까?

![](/assets/img/post_img/oauth_example.png){:width="350"}_카카오, 페이스북, 구글, 애플 아이디로 다른 사이트에 로그인 할 수 있습니다._

만일 우리가 하나의 웹서비스를 만들었다고 하죠. 그리고 그 서비스에 접속하는 사용자가 구글 캘린더에 등록한 일정을 보여주는 화면을 만들고 싶습니다. 그렇다면 우리 서비스는 사용자를 대신해서 구글에 접속해서 캘린더 정보를 가져올 수 있어야겠죠. 가장 쉬운 방법은 무엇일까요? 바로 우리의 구글 아이디와 비밀번호를 웹서비스에 전달해준 다음, 웹서비스가 그 정보를 그대로 이용해서 구글 캘린더의 정보를 가져오면 됩니다. 그러나 직관적으로 생각해봐도 이는 그리 좋은 방법은 아닙니다. 이러면 웹서비스가 사용자의 아이디와 비밀번호를 관리해야하는 부담감이 있고, 구글에서는 우리 웹서비스를 신뢰하기가 어렵죠.

이러한 문제를 해결하기 위해 OAuth가 등장했습니다. OAuth는 최초 1.0 버전이 있은 뒤로, 현재 거론되는 OAuth는 대부분 OAuth2.0입니다(OAuth2.0은 하위 버전과 호환되지 않는다고 합니다). OAuth는 표준화된 방법과 절차에 따라 권한을 인가(authorization)하기 위한 프로토콜입니다. 혹시 '~로 로그인 하기'와 같은 소셜 로그인 기능을 사용해 보신적 있지 않으신가요? 사용자가 특정 웹사이트에 접속한 다음, '구글 아이디로 로그인 하기' 버튼을 누르면 아이디와 비밀번호를 입력하고, 어떤 권한을 허용할 것인지 묻는 창이 나타납니다. 그리고 허용하면 구글 아이디를 통해서 다른 웹사이트에 로그인이 이루어집니다. 이러한 과정은 결론적으로 특정 웹사이트에게 구글 사용자로서의 권한을 '위임'해준 것입니다. 지금은 매우 단순하게 표현했지만, 사실 매우 복잡한 과정이 밑단에서 이루어져야 하며 그 과정을 표준화한 것이 바로 OAuth2.0이라고 이해하시면 됩니다.

<br>

### OAuth2.0의 주체

지금까지 사용자, 웹서비스, 구글 등의 표현을 사용하였는데요, 이를 OAuth 진영에서 사용하는 용어로 바꿔보겠습니다. 총 4가지의 주체가 있습니다.

- 리소스 소유자(resource owner):
  
  예시에서 '사용자'입니다. 리소스에 대한 접근 권한을 부여할 수 있는 주체입니다. 만일 리소스 소유자가 사람이라면 엔드 유저(end-user)라고도 불립니다.
- 리소스 서버(resource sever):
  
  예시에서 '구글'입니다. 리소스를 호스팅 하는 주체입니다. 엑세스 토큰(access token)을 확인하여 리소스에 접근하려는 요청을 허가할지 거부할지 결정 할수 있습니다. 아래에서 설명할 인가 서버와 합쳐서 표현하기도 합니다.
  
  > 리소스 서버에게 클라이언트가 누구인지 미리 알려주는 과정이 선행되어야 OAuth 2.0 프로토콜이 제대로 동작합니다. 이는 리소스 서버에서 client id, client secret를 발급받고, redirect uri를 등록 해두는 행위를 말합니다. 아래 OAuth2.0 동작 흐름에서 더 자세히 설명하겠습니다.
- 클라이언트(client):
  
  예시에서 '우리 웹서비스'입니다. 리소스 오너를 대신하여 리소스에 대한 접근을 요청 하는 주체입니다.
  
  > 단어 때문에 리소스 소유자와 클라이언트를 헷갈릴 수도 있습니다. 그러나 클라이언트라는 단어는 상대적인 개념입니다. 우리 웹서비스는 리소스 서버나 인가 서버 입장에서 보았을 때 클라이언트이므로 이러한 이름을 갖게 되었다고 생각할 수 있습니다.
- 인가 서버(authorization server):
  
  예시에서 '구글'입니다. 클라이언트에서 엑세스 토큰(access token)을 발급하는 주체입니다. 리소스 오너가 자신의 신분을 성공적으로 증명했을 때 엑세스 토큰을 발급 해 줍니다. 위에서 설명한 리소스 서버와 합쳐서 표현하기도 합니다.

<br>

### OAuth2.0의 동작 흐름(메커니즘)

먼저, OAuth2.0은 클라이언트가 리소스 소유자에게 인가를 얻어서 리소스 소유자 대신에 리소스 서버에 접근할 수 있는 방법이나 절차를 정의한 프로토콜이라는 사실을 잊지 말아야 합니다. 클라이언트는 인가를 잘 받았다는 징표로서 최종적으로 access token이라는 것을 얻을 수 있게되고, 이를 통해 정해진 권한 범위(scope) 내에서 리소스 서버의 자원을 마음껏 이용할 수 있게 되는 것입니다.

클라이언트가 access token, 즉 인가를 받기 위한 방법은 여러가지가 있습니다. 각 방식마다 장단점이 존재한다고 하는데, 더 자세한 내용은 [관련 자료](https://surprisecomputer.tistory.com/41)를 참고해주세요(구체적인 내용은 더 공부하고 정리해 보겠습니다). 본 글에서는 Authorization Code Grant 방식에 집중해서 살펴보겠습니다.

- [Authorization Code Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1): 클라이언트가 웹 서버인 경우 사용. 안정성이 높아 일반적으로 많이 사용하는 방식.
- [Implicit Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.2): 인가 서버에서 클라이언트에 곧바로 access token을 발급함. 브라우저에 access token이 그대로 노출되기 때문에 안전하지 않음.
- [Resource Owner Password Credentials](https://datatracker.ietf.org/doc/html/rfc6749#section-4.3): 리소스 소유자의 인증 정보가 클라이언트에 전송된 다음 바로 인증 서버로 전송해도 되는 경우 사용
- [Client Credentials Grant](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4): 클라이언트가 리소스 소유자와 동일할 경우 사용

<br>

![](/assets/img/post_img/oauth2_flow.png)_OAuth2.0: Authorization Code Grant 흐름도_

<br>

1. 리소스 서버에 우리 웹서비스를 클라이언트로 등록

   ![](/assets/img/post_img/google_oauth_client.png)_client_id, client_secret을 발급받고, redirect_uri를 등록합니다._

   우리 웹서비스를 클라이언트를 등록하고 `client_id`와 `client_secret`을 발급 받아둡니다. 그리고 리소스 소유자가 인증에 성공 했을 때 authorization code와 함께 사용자를 돌려 보낼 `redirect_uri`를 설정해야합니다. `redirect_uri`는 기본적으로 보안을 위해 https를 사용해야하지만 localhost의 경우는 예외적으로 http로 설정할 수 있습니다. `client_id`와 `client_secret` 또한 인가를 받으려는 클라이언트가 사전에 승인된(등록된) 대상인지를 확인하고 `access_token`을 발급하는데 사용됩니다. `client_id`는 공개되어도 상관 없지만 `client_secret`은 절대 유출되면 안 되는 정보입니다.

   <br>

2. 로그인 요청 및 유저 로그인 진행

   ![](/assets/img/post_img/oauth_login.png)_리소스 소유자가 로그인 하게하고, 로그인 성공시 허용할 권한 동의 여부를 묻는다(인가)._

   리소스 소유자가 클라이언트의 웹서비스에서 '구글 아이디로 로그인 하기' 버튼을 클릭합니다. 그러면 클라이언트는 리소스 소유자를 인가 서버의 로그인을 담당하는 웹페이지로 보냅니다(예를 들어 구글 로그인 화면 등). 이 때 `client_id`, `redirect_uri`, `response_type`, `scope` 등의 파라미터를 쿼리 스트링을 URL에 포함합니다. 요청은 다음과 같은 형태입니다.

   ```http
   https://accounts.google.com/o/oauth2/v2/auth?
    scope=scope&
    response_type=code&
    redirect_uri=http://localhost:5000/callback&
    client_id=client_id
   ```

   쿼리 스트링에는 보다 [다양한 파라미터](https://developers.google.com/identity/protocols/oauth2/web-server#httprest_1)를 포함할 수 있지만, 필수적인 파라미터는 다음과 같습니다.

   - `client_id`: 리소스 서버에 애플리케이션(웹서비스)를 등록하고 발급 받은 클라이언트 id입니다.
   - `redirect_uri`: 리소스 소유자가 로그인에 성공할 경우 리소스 소유자를 리디렉션 하는 주소입니다. 애플리케이션을 등록할 때 설정해둔 값과 정확히 일치해야 합니다. 그렇지 않을 경우 `redirect_uri_mismatch` 에러가 발생하게 됩니다.
   - `response_type`: authorization code grant 방식으로 진행할 경우 `code`라고 지정합니다.
   - `scope`: 인가를 해줄 권한의 범위를 뜻합니다. 로그인에 성공한 후에 사용자에게 권한 동의 여부를 묻는데, 이와 관련된 파라미터입니다.

   <br>클라이언트가 위 주소로 리소스 소유자를 이동시키면 유저는 자신의 아이디와 비밀번호를 입력하여 리소스 소유자임을 인증합니다. 최종적으로는 scope에 포함된 권한 인가를 동의 하겠냐는 안내 페이지가 보여지는데, 리소스 소유자가 이에 동의를 하면 클라이언트는 리소스 소유자를 대신하여 해당 권한을 가지고 리소스 서버에 자원을 요청할 수 있는 발판이 마련된 것입니다.

   <br>

3. authorization code 발급
   2번 과정을 통해 리소스 소유자가 인증을 성공적으로 수행하였다면, 인증 서버는 사전에 등록해둔 redirect_uri로 사용자를 이동시킵니다. 이때 중요한 점은 쿼리 스트링에 authorization code가 포함된다는 것입니다. authorization code는 최종적으로 access token을 발급받기 위한 임시적 성격을 가지고 있으며, 일반적으로 수명이 매우 짧습니다.

   <br>
   
4. access token 교환 요청
   <br>

5. access token 발급
   <br>

6. access token으로 리소스 요청
   <br>

7. access token 검증 후 리소스 응답





### access token

엑세스 토큰 교환 과정을 안전하게 하기 위해서는 authorization code를 발급 받고, 이를 신뢰할 수 있는 백엔드에 보낸뒤 백엔드에서 token enpoint에 요청을 해서 access token을 교환하는 과정으로 이루어져야 한다. 이렇게 하면 브라우저에서 직접적으로 access token을 교환하지 않기 때문에 보다 안전한 방법이라고 할 수 있다. implicit grant는 클라이언트가 곧바로 access token을 발급 받기 때문에 브라우저 상에 access token이 그대로 노출되버린다.



<br>

# OIDC(OpenID Connect)

### OIDC란?

**OIDC(OpenID Connect)란 <u>OAuth 위에서</u> 동작하는 사용하는 인증(authentication)을 위한 프로토콜**입니다. 앞서 우리는 위에서 OAuth2.0에 대해서 배웠습니다. 인가를 담당하는 표준화된 프로토콜이라고 했죠. 그리고 인가를 하기 위해서 필연적으로 인증 과정이 수반되었습니다(구글 로그인, 페이스북 로그인 등) 이러한 사실 때문에 많은 사람들이 OAuth를 인가와 인증 두 가지 목적을 모두 담당하는 프로토콜이라고 오해하고 있습니다.(여기에 저도 포함입니다... 🙋‍♀️)

<br>

### OIDC와 OAuth를 구분하자

그러나 이는 엄청난 오해입니다. OAuth와 인증은 초콜릿과 퍼지(일종의 캔디)로 비유할 수 있습니다. 초콜릿은 다른 여러 식재료의 재료가 되면서도 그 자체가 식재료가 될 수 있습니다. 그리고 퍼지는 초콜릿을 조합하여 만들 수도 있지만 다른 식재료를 조합하여 만들 수도 있습니다(코코넛 감자 퍼지). 따라서 우리는 초콜릿을 퍼지와 동일한 것으로 취급하지는 않습니다. 또한 초콜릿을 초콜릿 퍼지와 동일하게 취급하는 것도 틀린 표현입니다. 초콜릿과 퍼지의 비유에서 **초콜릿은 OAuth**입니다. OAuth는 다른 기술을 위한 베이스가 되기도 하지만, 단독으로 인가의 기능을 수행할 수도 있습니다. **퍼지는 인증**입니다. 인증을 위해서는 다양한 기술이 활용됩니다. OAuth도 인증을 구현하기 위한 바탕이 되는 기술 중 하나일뿐입니다. **특히 OAuth 프로토콜 위에서 인증을 구현한 프로토콜이 바로 OIDC(OpenID Connect)**입니다.

이처럼 OAuth와 OIDC는 뗄려야 뗄 수 없는 관계입니다. 그러나 OAuth와 OIDC를 혼동하는 것은 [OAuth 공식 문서](https://oauth.net/articles/authentication/)에서도 지적하고 있는 매우 중대한 오류입니다(그 이유에 대해서는 부록에 번역 및 작성해 두었습니다). 정리하자면, OAuth는 인가를 위한 프로토콜인데, 특정 범위(scope)의 권한을 인가하기 위해서는 어쩔 수 없이 인증의 과정을 거치기는 합니다. 그러나 OAuth 흐름에서 발생하는 인증은 본래의 목적이 아닙니다. 하나의 수단에 불과합니다. 한편 OIDC는 OAuth 위에서 동작하는 인증 프로토콜입니다. OAuth가 인가를 위해 어쩔 수 없이 인증 과정을 거치는데, 이를 활용하여 사용자의 신원을 식별할 수 있는 `id_token`을 발급합니다(이때 일반적으로 `access_token`도 같이 발급됩니다).

<br>

### id_token

따라서 일반적으로는 OIDC 등을 활용하여 사용자의 정보를 담고 있는 id token을 따로 발급 하여 인증을 사용합니다. 이처럼 access token와 id token의 역할을 분리하면 OAuth 프로토콜의 원칙을 지키면서 인증 과정까지 함께 수행할 수 있는 장점이 있습니다. OIDC 프로토콜의 최종 결과로서 클라이언트는 `id_token`을 발급 받습니다. `id_token`은 JWT(Json Web Token) 형태이며, `Header`, `Payload`, `Signature`라는 총 3개의 파트로 구분 되어 있습니다.

<br>

`Header`에는 토큰 타입과 어떤 해시 암호화 알고리즘을 사용하는지 명시되어 있습니다.

```json
{
  "alg": "HS256",  // 해시 알고리즘 (HMAC, SHA256, RSA)
  "typ": "JWT" // 토큰 유형
}

```

<br>
`Payload`는 JWT의 핵심 내용이 명시 되어 있는 부분으로 여러 종류의 claim[^1]으로 구성되어 있습니다.

```json
{
  "iss": "https://accounts.google.com",
  "azp": "1234987819200.apps.googleusercontent.com",
  "aud": "1234987819200.apps.googleusercontent.com",
  "sub": "10769150350006150715113082367",
  "at_hash": "HK6E_P6Dh8Y93mRNtsDB1Q",
  "hd": "example.com",
  "email": "jsmith@example.com",
  "email_verified": "true",
  "iat": 1353601026,
  "exp": 1353604926,
  "nonce": "0394852-3190485-2490358"
}
```

특히 중요한 claim을 다음과 같습니다.

- `iss`(issuer): `id_token`을 발급한 주체(e.g., google, facebook, oidc provider)
- `aud`(audience): `id_token`을 활용하는 주체(e.g., 클라이언트, aws.sts)
- `sub`(subject): 리소스 오너(resource owner), 엔드 유저(end-user) (e.g., 사용자, 쿠버네티스 pod)
- `exp`: `id_token` 만료 시점
- `iat`: `id_token` 발급 시점

<br>

마지막으로 Signature는 id_token의 유효성을 검사 하기 위한 암호화된 문자열로 이루어져 있습니다. 해당 문자열은 id_token 발급자의 비밀키로 암호화 되어 있으므로, 발급자의 공개키로 복호화가 가능합니다(?).

<br>

### access token vs id token

|             | access token                                                 | id token                                                     |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| audience    | Access tokens are meant to be read by the resource server(Access tokens must not be read or interpreted by the OAuth client. The OAuth client is not the intended audience of the token). | ID tokens are meant to be read by the OAuth client           |
| information | Access tokens do not convey user identity or any other information about the user to the OAuth client. | An ID token contains information about what happened when a user authenticated.  The ID token may also contain information about the user such as their name or email address, although that is not a requirement of an ID token. |
| usage       | Access tokens should *only* be used to make requests to the resource server. | [ID tokens](https://oauth.net/id-tokens-vs-access-tokens/) must *not* be used to make requests to the resource server. |
| form        | [can be JWTs](https://oauth.net/2/jwt-access-tokens/) but may also be a random string | [JWTs](https://oauth.net/2/jwt/)                             |



### OIDC에서 id token을 발급 받는 과정

 다이어그램 그리기

<br>

# IRSA(IAM Role for Service Account) 프로세스

![](/assets/img/post_img/irsa_architecture.png)_IRSA High Level Architecture_

IRSA(IAM Role for Service Account)는 [이전 글]()에서 예시로 든 Pod identity webhook을 활용하여 Service Account Token Volume Projection 기능 구현하고, 이를 쿠버네티스 파드 별로 임시 자격 증명을 언급하고 AWS 리소스들에 엑세스 가능 하도록 하는 일종의 프로토콜입니다.

<br>

# 부록

## access_token을 유저 인증에 사용하면 안될까?

여기까지 오셨으면 OIDC와 OAuth와의 미묘한 차이를 이해하셨 것이라 생각합니다. 그런데 이렇게 생각해볼 수 있지 않을까요? OAuth는 어쨌든 인증 과정을 거쳐서 access_token을 발급해주었으니까 access_token을 인증 과정에 사용해도 되지 않을까요? 결론부터 말하면 이는 틀린 생각입니다. [oauth.net](https://oauth.net/articles/authentication/)에서는 OAuth로 인증을 하는 것의 위험성(*Common pitfalls for authentication using OAuth*)에 대해서 언급합니다.



### access token에는 유저를 식별할 수 있는 정보가 없습니다

access token을 발급 받는 과정에 사용자 인증이 선행되기 때문에 OAuth 자체가 인증도 담당하고 있다고 생각할 수 있습니다. 그러나 access token은 근본적으로 client를 위한 개념이 아닙니다. 다시말해, access token의 audience(토근 발급 대상, 토근의 최종 사용 주체)는 client가 아니라 resource server(protected resource)입니다. client 입장에서 access token은 아무런 의미 없는 string에 불과합니다(*the token is designed to be opaque to the client*). 극단적으로 말하면, access token의 포맷이 바뀌었어도 authorization server와 resource server간의 로직만 이상없다면 client 입장에서는 신경 쓸 일이 전혀 없습니다. 그저 resource server로 access token을 전달하기만 하면 되니까요. 또한 access token에는 사용자와 관련된 데이터(`user_id`나 `email` 등)는 전혀 없으므로 클라이언트가 사용자를 식별할 수 있는 방법은 없습니다. client가 access token으로 사용자를 식별한다는 행위는 애초에 불가능한 행위라는 것입니다.



### 유저의 인증 없이도 access token이 발급될 수 있습니다.

OAuth 과정에서 유저의 로그인을 통해 인증을 수행하고 최종적으로 access token을 발급 받을 수 있습니다. 그러나 이것이 access token을 발급 받을 수 있는 유일한 방법은 아닙니다. refresh token을 활용하여 유저의 인증 없이도 access token을 새로 발급 받을 수도 있습니다. 즉 access token을 발급 받았다는 사실 자체가 유저가 인증되었음을 완벽하게 보장할 수 없다는 뜻입니다.



### 인증 서버가 아닌 제 3자가 access token을 주입할 수도 있습니다.

만일 acces token이 URL 파라미터로 전달되는 implicit flow를 따르고, client가 access token의 유효성을 점검할 수 있는 메커니즘이 없다면 access token의 진위 여부를 판별하기 어렵습니다. 따라서 access token만으로 유저가 진짜로 인증했는지 여부를 판별하기 어렵습니다.



### access token을 발급 해준 client인지 여부를 검증하는 메커니즘이 없습니다.

예를 들어, A라는 client가 정상적인 방법을 통해 access token을 발급 받은 후에 B라는 client가 해당 access token을 그대로 사용한다면, 일반적인 OAuth API들은 client를 검증하는 메커니즘이 없으므로 정상적으로 resource server와 통신이 일어날 것입니다. B라는 client가 A라는 client가 발급 받은 access token을 사용함에 있어서 유저의 인증은 개입되지 않았다는 점에서 access token으로 유저 인증을 하기에는 부족한 면이 있습니다.



### access token에 대한 표준 형식이 없습니다.

OAuth 상에서 유저 인증 과정을 처리하려는 시도에 있어서 가장 큰 문제점은 access token의 [표준 형식이 정해져있지 않다는 것](https://www.rfc-editor.org/rfc/rfc6749#section-1.4)입니다(*access tokens can have different formats, structures, and methods of utilization*) 예를 들어서 google에서는 유저의 아이디를 user_id 필드에 넘겨주는데, 페이스북에서는 subject 필드에 넘겨줄 수도 있습니다. 두 필드는 의미상으로는 동일하지만, 이를 처리하기 위해서는 서로 다른 방식으로 처리하는 코드를 짜야하기 때문에 번거롭습니다.

<br>

# 참고자료

- [AWS EKS의 RBAC, IRSA 딥다이브](https://surgach.tistory.com/119)
- [https://developers.google.com/identity/protocols/oauth2/openid-connect](https://developers.google.com/identity/protocols/oauth2/openid-connect)
- [Naver D2 - OAuth와 춤을](https://d2.naver.com/helloworld/24942)
- [JWT (Json Web Token) Audience "aud" versus Client_Id - What's the difference?](https://stackoverflow.com/questions/28418360/jwt-json-web-token-audience-aud-versus-client-id-whats-the-difference)
- [OAuth 그리고 OpenID Connect](https://6991httam.medium.com/oauth%EB%9E%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-openid-8c46a65616e6)
- [ID Tokens vs Access Tokens](https://oauth.net/id-tokens-vs-access-tokens/)
- [ID Token and Access Token: What's the Difference?](https://auth0.com/blog/id-token-access-token-what-is-the-difference/)
- [The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749)
- [JWT(JSON Web Token)을 이용한 API 인증 - #1 개념 소개](https://bcho.tistory.com/999)
- [OIDC (OpenID Connect)](https://ssup2.github.io/theory_analysis/OIDC/)
- [인증과 인가 (권한 부여) 비교 – 특징 및 차이점](https://gguguk.github.io/posts/oidc/)
- [OpenID(OIDC) 개념과 동작원리](https://hudi.blog/open-id/)
- [OAuth 2.0 개념과 동작원리](https://hudi.blog/oauth-2.0/)
- [OAuth 개념 및 동작 방식 이해하기](https://tecoble.techcourse.co.kr/post/2021-07-10-understanding-oauth/)
- [OAuth 2.0 기반 인증 방식](https://surprisecomputer.tistory.com/41)
- [OAuth 2.0 - Authorization code Grant](https://yonghyunlee.gitlab.io/temp_post/oauth-authorization-code/)
- [OAuth 2.0 - Implicit Grant](https://yonghyunlee.gitlab.io/temp_post/oauth-Implicit/)
- [OAuth 2.0을 사용하여 Google API에 액세스하기](https://developers.google.com/identity/protocols/oauth2)
- [웹 서버 애플리케이션용 OAuth 2.0 사용](https://developers.google.com/identity/protocols/oauth2/web-server)



[^1]: JWT에서 claim이란 프로퍼티나 속성을 말합니다. `"iss": "https://abc.com"` 를 하나의 claim이라고 할 수 있으며, `iss`를 claim 이름, `https://abc.com`을 claim 값이라고 합니다.