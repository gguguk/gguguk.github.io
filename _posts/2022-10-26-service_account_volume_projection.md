---
title: IRSA의 원리를 파헤쳐보자 2 - K8S Sevice Account와 Service Account Token Volume Projection
author: Gukwon Koo
categories: [MLOps, Kubernetes, IRSA]
tags: [kubernetes, k8s, service account, service account token volume projection, irsa]
pin: false
math: true
comments: true
date: 2022-10-26 22:12:00 +0900
---

IRSA의 원리를 파헤쳐보자 시리즈의 마지막 글입니다. 저번 글에서는 admission webhook을 학습했습니다. 요약하자면 EKS 클러스터를 설치하면 control plane에 pod identity webhook이라는 webhook server(일종의 API 서버)가 함께 배포되고, pod identity webhook은 서비스 어카운트에 iam arn이 annotation 되어 있으면 요청 내용을 변형하여 파드를 생성합니다. 생성된 파드에는 **환경 변수**와 **토큰값**이 저장되어 있어서 이를 활용해 AWS 리소스에 접근할 수 있게 됩니다.

이번 글에서는 파드에 생성되는 환경 변수와 토큰값이 무엇인지 살펴보고, 최종적으로 어떻게 파드가 AWS 리소스와 통신을 할 수 있게 되는지 살펴보겠습니다.

IRSA의 원리를 파헤쳐보자 시리즈

1. [IRSA의 원리를 파헤쳐보자 1 - K8S Admission Webhook](https://gguguk.github.io/posts/admission_webhook/)
2. **IRSA의 원리를 파헤쳐보자 2 - K8S Sevice Account와 Service Account Token Volume Projection**

<br>

#  서비스 어카운트(service account)와 토큰(token)

쿠버네티스 클러스터에 파드를 생성하면 같은 네임스페이스에 `default`라는 서비스 어카운트(service account)와 여기에 마운트되는 시크릿(secret)이 자동적으로 생성됩니다. 이 시크릿을 살펴보면 토큰(token)이라는 필드가 있고, 매우 긴 스트링이 할당되어 있습니다. default 네임스페이스에 default라는 이름을 이름을 가진 서비스 어카운트의 정보를 확인해봅시다.

```bash
kubectl describe sa default -n default
```

그러면 `default-token-64rhn`라는 이름을 가진 시크릿이 해당 서비스 어카운트에 마운트 되어 있는 것을 확인할 수 있습니다.

```bash
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   default-token-64rhn
Tokens:              default-token-64rhn
Events:              <none>
```

그럼 저 시크릿에 어떤 정보가 있는지 확인해볼까요? 다음의 명령어로 확인 가능합니다.

```bash
kubectl describe secrets default-token-64rhn -n default
```

그러면 서두에서 언급했던 토큰(token) 정보가 저장되어 있음을 확인할 수 있습니다.

```bash
Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Il9MTTM0aHpjYmYwck5sWGNTcUZIVExEazc2M2xfTkZTenBkb3JyMHk1UlkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tNjRyaG4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjI3NTI0OGFhLTQyNjMtNDU1My1iZDlmLWY3M2Y2MjAwYTQyZSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.xIgO11X5hM5mHlPak8wcqHKq98BnppWq2OM07URa0JdXc4TeMySI534sHXlFiUNJfA7OX7_x4JtLyo7rxPQKl1c7lHnt2DYFMDhXyui9v6x7Y_IEH4vSe-7Vo5iyv0Nf9kKY8aKq5fqOS2MwsomZnrjhpDKBU6-mlxEF9XKtJH5YKN3FZ-ZDv-cJXlrTAUFwPj_KfY_ypAKBdhDtBJQTQKgQrNZXCDTXEJ4dqUhnPikLwv6_pwC0w1Kn9EyBLUfPjXka8A8kdsbl1RKGkfxetOR9-AJUlemg4ugCt7A5FP-nSF_95Q6RBYcCwgYg1595pj3YEOT8r-05bxOQEt8SrA
```

 [여기](https://jwt.io/#debugger-io?token=eyJhbGciOiJSUzI1NiIsImtpZCI6Il9MTTM0aHpjYmYwck5sWGNTcUZIVExEazc2M2xfTkZTenBkb3JyMHk1UlkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tNjRyaG4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjI3NTI0OGFhLTQyNjMtNDU1My1iZDlmLWY3M2Y2MjAwYTQyZSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.xIgO11X5hM5mHlPak8wcqHKq98BnppWq2OM07URa0JdXc4TeMySI534sHXlFiUNJfA7OX7_x4JtLyo7rxPQKl1c7lHnt2DYFMDhXyui9v6x7Y_IEH4vSe-7Vo5iyv0Nf9kKY8aKq5fqOS2MwsomZnrjhpDKBU6-mlxEF9XKtJH5YKN3FZ-ZDv-cJXlrTAUFwPj_KfY_ypAKBdhDtBJQTQKgQrNZXCDTXEJ4dqUhnPikLwv6_pwC0w1Kn9EyBLUfPjXka8A8kdsbl1RKGkfxetOR9-AJUlemg4ugCt7A5FP-nSF_95Q6RBYcCwgYg1595pj3YEOT8r-05bxOQEt8SrA)에 방문해서 토큰 정보를 붙여 넣으면 토큰의 정보를 디코딩 할 수 있습니다. 토큰의 페이로드 부분만 떼 놓고 보면 아래의 정보가 담겨 있습니다. 이 토큰은 JWT(Json Web Token) 형태입니다.

```bash
{
  "iss": "kubernetes/serviceaccount",
  "kubernetes.io/serviceaccount/namespace": "default",
  "kubernetes.io/serviceaccount/secret.name": "default-token-64rhn",
  "kubernetes.io/serviceaccount/service-account.name": "default",
  "kubernetes.io/serviceaccount/service-account.uid": "275248aa-4263-4553-bd9f-f73f6200a42e",
  "sub": "system:serviceaccount:default:default"
}
```

토큰의 발급자(`iss`)가 누구인지와 엔드 유저(`sub`)가 누구인지 적혀 있습니다. 다시 말해 `kubernetes/serviceaccount`가 이 토큰을 발급 했으며, `system:serviceaccount:default:default`가 이 토큰의 엔드 유저입니다. 이 토큰은 서비스 어카운트를 사용하는 파드에 마운트 됩니다 `/var/run/secrets/kubernetes.io/serviceaccount` 경로를 찾아가 보면 찾을 수 있습니다. 파드가 kube-apiserver와 통신을 할 때, header 부분에 토큰을 실어서 보내게 되는데 kube-apiserver에서는 이 토큰이 유효한지 판단한 후 토큰에서 서비스 어카운트의 이름을 얻어냅니다. 그리고 서비스 어카운트에 바인딩 된 롤을 확인하고 인가(authorization)를 통해 특정 행위를 수행할 권한이 있는지를 확인하고 요청 내용을 처리합니다.

<br>

그러나 이러한 기본 토큰은 쿠버네티스 내에서 사용하기 설계되었기 때문에 다음의 문제점이 있습니다:

- 토큰에 유효기간이 없습니다. 만일 어드민 롤이 바인딩된 서비스 어카운트의 토큰이 유출된다면 보안에 심각한 문제가 발생할 것입니다.
- audience 개념이 없습니다. audience는 이 토큰을 활용하여 여러 서비스나 리소스에 접근하는 대상을 말합니다. 만일 어떤 유저가 웹사이트 A가 토큰을 웹사이트 B에 넘겨 주었을 때, 웹사이트 B가 웹사이트 A를 가장하여 기밀한 자원에 엑세스할 수 있습니다.
- 외부 서비스와 쉽게 통합되기 어렵습니다. 쿠버네티스 기본 서비스 어카운트 토큰은 쿠버네티스에서만 사용하는 필드들이 많이 있습니다. 따라서 OAuth2.0이나 OIDC 프로토콜 위에 구현된 웹서비스가 쿠버네티스 기본 토큰을 활용하기 위해서는 추가적으로 기능 개발을 해야하는 수고가 있습니다.

이러한 문제들은 쿠버네티스 1.21 이상부터 제공하는 **Service Account Token Volume Projection**이라는 기능을 활용하면 해결할 수 있습니다.

<br>

# Service Account Token Volume Projection

위에서 살펴본 것처럼 기본 쿠버네티스 서비스 어카운트가 제공하는 토큰은 내부에서의 활용만 고려하여 설계 되었기 때문에 외부 서비스에 엑세스 하기 위해서는 추가적인 정보들이 필요합니다. 추가적인 정보들엔 무엇이 있고, 왜 필요한지에 대해서는 이어지는 글에서 더 자세히 설명할 예정입니다. 지금은 기본 서비스 어카운트의 토큰은 외부의 리소스에 접근하는데 사용하기엔 뭔가 부족하구나~ 그래서 추가적인 정보를 주입해 주어야 하는구나~ 라고 생각하고 넘어가주세요.

쿠버네티스 1.12 이후 부터는 Service Account Token Volume Projection라는 기능을 사용할 수 있는데, 이를 통해 kubelet이 토큰에 추가적인 정보(토큰의 유효기간, audience 등)를 주입시킬 수 있게 됩니다. 예를 들어 파드 생성시 토큰에 <u>유효시간: 1시간</u>, <u>audience: foobar.com</u>이라는 정보를 주입하고 싶으면 아래와 같이 매니패스트를 작성하면 됩니다. 새롭게 추가된 필드는 다음과 같습니다:

- `spec.volumes[0].projected.source[0].serviceAccountToken.expirationSeconds: 7200` ->  유효시간 2시간
- `spec.volumes[0].projected.source[0].serviceAccountToken.audience`-> audience는 vault

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basic-debian-pod-bound-token
  namespace: default
spec:
  serviceAccountName: default
  containers:
  - image: debian
    name: main
    command: ["sleep", "infinity"]
    volumeMounts:
    - name: my-bound-token
      mountPath: /var/run/secrets/my-bound-token
  volumes:
  - name: my-bound-token
    projected:
      sources:
      - serviceAccountToken:
          path: token
          audience: foobar.com
          expirationSeconds: 3600
```

<br>

위 매니패스트로 생성된 파드에는 `/var/run/secrets/my-bound-token`  경로에 다음과 같은 토큰이 마운트 되어 있습니다.

```json
{
 "aud": [
   "foobar.com"
 ],
 "exp": 1636151360,
 "iat": 1636147760,
 "iss": "https://oidc.eks.ap-northeast-2.amazonaws.com/id/B0678ED568FC12BBC37256BBA2A4BB53",
 "kubernetes.io": {
   "namespace": "default",
   "pod": {
     "name": "basic-debian-pod-bound-token",
     "uid": "a593ded9-c93d-4ccf-b43f-bf33d2eb7635"
   },
   "serviceaccount": {
     "name": "default",
     "uid": "ab71f2b0-abca-4bc7-905a-cc9b2d6832cf"
   }
 },
 "nbf": 1636147760,
 "sub": "system:serviceaccount:default:default"
}
```

기본 서비스 어카운트의 토큰과 생김새가 많이 다릅니다. 가장 크게 변화된 부분은 `aud`, `exp`, `iat` 필드가 있다는 점인데요, 이 토큰은 OIDC의 표준 포맷을 따른 다른다는 점이 중요합니다. 따라서 쿠버네티스 파드와 외부 서비스를 쉽게 통합할 수 있게 되었습니다. 또한 토큰의 유효기간(`exp`) 필드가 생겨서 외부 서비스가 이 토큰의 유효성을 쉽게 검증할 수 있습니다.

<br>

# 마무리

본 글에서는 IRSA를 이해하기 위한 배경 지식으로 쿠버네티스 서비스 어카운트와 서비스 어카운트 볼륨 프로젝션에 대해서 살펴보았습니다. 핵심은 기본 서비스 어카운트에 부족한 정보들을 서비스 어카운트 볼륨 프로젝션을 활용하여 추가적으로 주입시킬 수 있다는 것입니다. 그리고 새롭게 추가되는 정보들은 OIDC 프로토콜의 표준 포맷을 잘 따르기 때문에, 이에 기반하여 구축된 서비스들과 쉽게 통합될 수 있다는 점이 큰 장점입니다.

다음 시간에는 지금까지 2개의 시리즈 글에서 배웠던 내용을 통해 IRSA의 전체적인 프로세스를 이해하는 시간을 가져볼 수 있을 것 같습니다. 저번 글에서 살펴본 pod identity webhook이 추가한 정보들은 무엇인지, 서비스 어카운트 볼륨 프로젝션을 통해 파드에 마운트된 토큰은 어떤 방식으로 활용되는지 등을 좀 더 구체적으로 살펴보겠습니다. 그리고 이번 시리즈 글을 마무리 하도록 하죠!

<br>

# 참고자료

- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#before-you-begin)
- [지구별 여행자 - Service Account Token Volume Projection](https://kangwoo.kr/2020/02/13/service-account-token-volume-projection/)
- [OpenID Connect explained](https://connect2id.com/learn/openid-connect)
- [What GKE users need to know about Kubernetes' new service account tokens](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-bound-service-account-tokens?hl=en)