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

 [여기](https://jwt.io/#debugger-io?token=eyJhbGciOiJSUzI1NiIsImtpZCI6Il9MTTM0aHpjYmYwck5sWGNTcUZIVExEazc2M2xfTkZTenBkb3JyMHk1UlkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tNjRyaG4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjI3NTI0OGFhLTQyNjMtNDU1My1iZDlmLWY3M2Y2MjAwYTQyZSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.xIgO11X5hM5mHlPak8wcqHKq98BnppWq2OM07URa0JdXc4TeMySI534sHXlFiUNJfA7OX7_x4JtLyo7rxPQKl1c7lHnt2DYFMDhXyui9v6x7Y_IEH4vSe-7Vo5iyv0Nf9kKY8aKq5fqOS2MwsomZnrjhpDKBU6-mlxEF9XKtJH5YKN3FZ-ZDv-cJXlrTAUFwPj_KfY_ypAKBdhDtBJQTQKgQrNZXCDTXEJ4dqUhnPikLwv6_pwC0w1Kn9EyBLUfPjXka8A8kdsbl1RKGkfxetOR9-AJUlemg4ugCt7A5FP-nSF_95Q6RBYcCwgYg1595pj3YEOT8r-05bxOQEt8SrA)에 방문해서 토큰 정보를 붙여 넣으면 토큰의 정보를 디코딩 할 수 있습니다. 토큰의 페이로드 부분만 떼 놓고 보면 아래의 정보가 담겨 있습니다.

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

토큰의 발급자(iss)가 누구인지와 엔드 유저(sub)가 누구인지 적혀 있습니다. 다시 말해 `kubernetes/serviceaccount`가 이 토큰을 발급 했으며, `system:serviceaccount:default:default`가 이 토큰의 엔드 유저입니다. 이 토큰은 서비스 어카운트를 사용하는 파드에 마운트 됩니다 /var/run  경로를 찾아가 보면 찾을 수 있습니다. 파드가 kube-apiserver와 통신을 할 때, header 부분에 토큰을 실어서 보내게 되는데 kube-apiserver에서는 이 토큰이 유효한지 판단한후 토큰에서 서비스 어카운트의 이름을 얻어냅니다. 그리고 서비스 어카운트에 바인딩 된 롤을 확인하고 인가(authorization)를 통해 특정 행위를 수행할 권한이 있는지를 확인하고 요청 내용을 처리합니다.

<br>

그러나 기본 서비스 어카운트의 토큰에는 최대 단점이 존재합니다. 발급된 토큰에 유효기간이 따로 지정되어 있지 않기 때문에 토큰이 영원히 살아있게 됩니다. 만약 어드민 권한이 롤 바인딩 된 서비스 어카운트의 토큰이 탈취 당한다면 심각한 보안상 위험 요소가 될 것입니다. 물론 토큰이 탈취 당하지 않도록 주의하는게 최선이겠지만, 만일 탈취 당하더라도 토큰의 유효기간이 있으면 일정 기간이 지난 후에는 해당 토큰이 쓸모 없어지게 되므로 보안의 수준을 한층 끌어 올릴 수 있을 것입니다. 그렇다면 서비스 어카운트의 토큰에 어떻게 유효기간을 설정할 수 있을까요? 쿠버네티스에서 제공하는 Service Account Token Volume Projection이라는 기능을 활용하면 가능합니다.

<br>

# Service Account Token Volume Projection

기본 service account은 토큰 만료 시간이 무제한인 단점이 있음. 이를 해결하고자 kubernetes 버전 x 부터는 Service Account Token Volume Projection이라는 기능을 도입하였습니다.

<br>

# IRSA(IAM Role for Service Account)

IRSA는 Service Account Token Volume Projection 기능 활용하여 AWS가 구축한 프로토콜에 인증과 인가를 한것

# 마무리

정리 내용 작성

<br>

# 참고자료

- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#before-you-begin)