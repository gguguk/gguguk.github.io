---
title: IRSA의 원리를 파헤쳐보자 1 - K8S Admission Webhook
author: Gukwon Koo
categories: [MLOps, Kubernetes]
tags: [kubernetes, k8s, admission controller, admission webhook, pod identity webhook]
pin: false
math: true
comments: true
date: 2022-09-13 23:25:00 +0900
---

IRSA(IAM Role for Service Account)는 AWS EKS에서 파드 단위로 권한을 관리하기 위한 방법 또는 프로세스입니다. 요즘에 회사에서 kubeflow를 셋팅하고 있는데 AWS의 특정 리소스(S3 등)와의 통신을 위해 파드에 일정 권한을 부여해야할 상황이 자주 발생합니다. 저는 이때 주로 IRSA를 활용하여 업무를 진행하고 있습니다. IRSA를 활용하면 권한 관리가 매우 수월하고, 보안과 관련된 사항이 노출될 위험도가 낮기 때문입니다. IRSA의 대안으로 `Secret`을 활용할 수 있습니다. 그러나 `Secret`은 [namespaced resource 이므로](https://stackoverflow.com/q/46297949) 작업성이 더 떨어진다고 느껴져서 그다지 선호하지 않습니다(하지만 직관적인 방법인 것은 사실입니다).

IRSA를 활용하는 것 자체는 그다지 어려운 일은 아닙니다. AWS IAM의 체계와 쿠버네티스 기본 개념만 있다면 손쉽게 적용할 수 있습니다. 그러나 IRSA의 동작 방식을 이해하기 위해서는 많은 배경지식이 필요합니다. 이 시리즈를 시작한 목적도 제가 현업에서 IRSA의 동작 과정상 필요한 배경지식을 제대로 파악하지 못 하고 단순 활용만 하고 있었기에 제대로 된 정리가 필요하다는 생각이 들어서였습니다.

그래서 준비 했습니다. *IRSA의 원리를 파헤쳐보자* 시리즈! 앞으로 IRSA의 작동 원리를 완벽하게 이해하기 위해서 필요한 요소들을 정리하고자 합니다. 그 첫번째 순서로 쿠버네티스 admission webhook을 다뤄보겠습니다. admission webhook은 IRSA가 이루어지기 위한 쿠버네티스쪽의 필수 오브젝트입니다.

<br>

# Admission Controller란?

### Admission Controller

![https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/](https://www.oreilly.com/library/view/kubernetes-best-practices/9781492056461/assets/kubp_1701.png)_admission control(출처: [Kubernetes Admission Control](https://www.digihunch.com/2022/01/kubernetes-admission-control/))_

**admission controller**란 kubernetes api server를 호출 했을 때, 요청 내용을 가로채서(intercept) 변형(mutating)하거나 검증(validating)하는 쿠버네티스 plugin의 집합을 가르킵니다. admission controller가 개입하여 무엇인가 하는 것을 admission control이라고도 표현할 수 있겠습니다. 앞서 언급 했지만 admission control은 크게 2단계로서 변형(mutating), 검증(validating) 단계가 있습니다. 모든 단계를 성공적으로 통과한 요청은 `etcd`에 저장(persistence) 됩니다.

<br>

변형(mutating)은 요청 내용을 수정하는 것을 말합니다. 예를 들어 특정 조건을 만족시킬 때 환경 변수를 추가적으로 주입시키는 등의 행위를 할 수 있습니다. 검증(validating)은 해당 요청 내용이 실제 쿠버네티스 클러스터에 적용되어도 되는지 확인합니다. 만약 부적절한 요청일 경우 거절 될 수 있습니다. 예를 들어서 특정 네임스페이스가 차지할 수 있는 리소스의 총량을 초과한다면 해당 요청은 거절 될 수 있습니다. 너무 추상적인 개념 위주로만 설명하니 별로 와 닿지 않네요. 구체적인 사례를 들어 볼까요? 여러분이 아실만한 [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-does-each-admission-controller-do)에는 다음과 같은 것들이 있습니다.

- [LimitRange](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#limitranger), [ResourceQuota](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#resourcequota): 특정 네임스페이스의 자원의 한계치를 컨트롤 합니다.
- [ServiceAccount](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#serviceaccount): 자주 사용하던 ServiceAccount도 admission controller의 한 종류입니다. ServiceAccount에 대한 자동화(automation)가 구현되어 있습니다.
- [NamespaceLifecycle](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#namespacelifecycle): `Terminating` 단계에 있는 namespace에 object가 배포되는 것을 거절(deny) 합니다.

<br>

이러한 admission controller는 쿠버네티스 클러스터를 생성하면 기본적으로 kube-apiserver에 binary 형태로 컴파일 되어 존재합니다. 만일 새로운 admission controller을 만들었고 이를 쿠버네티스 클러스터에 적용하고 싶다면, 이를 binary로 컴파일해서 kube-apiserver에 적용해야 합니다. 이러한 작업이 자주 발생한다면 어떨까요? 컴파일을 반복적으로 수행해야 하기 때문에 번거롭고 유연성이 떨어질 것입니다. 이와 같은 불편함을 해결하고자 **admission webhook**이라는 개념이 등장하게 되었습니다.

<br>

### Admission Webhook

![](https://miro.medium.com/max/4800/0*H5j2uU_zFUEU__Uz.jpg)_admission webhook (출처: [Diving into Kubernetes MutatingAdmissionWebhook](https://medium.com/ibm-cloud/diving-into-kubernetes-mutatingadmissionwebhook-6ef3c5695f74))_

admission webhook은 쿠버네티스 1.9 버전부터 등장 하였습니다. 변형(mutating)을 수행하는 `MutatingAdmissionWebhooks`과 검증(validating)을 수행하는 `ValidatingAdmissionWebhooks`를 admission webhook이라고 합니다. admission webhook은 정해진 형식에 따른 request를 받아서 그 request를 변형(mutating)하거나 검증(validating)하는 HTTP [callback](https://leffept.tistory.com/329)을 뜻합니다. 헷갈릴 수도 있지만 `MutatingAdmissionWebhooks`와 `ValidatingAdmissionWebhooks`은 여전히 admission controller 입니다.  

> 여기서 말하는 callback은 [특정 이벤트에 따라 호출되어지는 함수](https://satisfactoryplace.tistory.com/18)라고 생각하는게 이해하기 쉽습니다. admission controller 자체가 특정 조건을 만족할 때 비로소 작동하는 plugin이라는 정의를 떠올려 보면 일맥 상통하는 말임을 알 수 있습니다.

<br>

실제 admission control 로직을 처리하는 주체는 admission webhook이 아닙니다. `kube-apiserver` 밖에 존재하는 흔히 떠올릴 수 있는 API 서버인 webhook admission server가 admission control 로직을 처리합니다.  `kube-apiserver`로 보내진 요청값이 특정 조건을 만족했다면(`MutatingWebhookConfiguration`에 선언된 규칙), `MutatingAdmissionWebhooks`과 `ValidatingAdmissionWebhooks` 단계에서는 해당 조건과 연결된 webhook admission server에 admissionReview라는 데이터를 POST 방식의 body로 실어 보냅니다. 그리고 해당 데이터를 처리하도록 구현된 webhook admission server(API 서버)는 처리 결과를 응답으로 돌려줍니다. 

<br>

정리하자면 일반적인 admission webhook은 일반적인 admission controller와 다르게 admission control 로직을 외부의 API에 맡기기 때문에 새로운 admission control 로직을 적용하고자 할 때 컴파일을 거칠 필요가 없습니다. 그저 어떤 조건을 만족할 때 어느 API로 보낼지를 결정하고(`MutatingWebhookConfiguration`), API 내의 처리로직만 신경 쓰면 됩니다. 그렇기 때문에 일반적인 admission controller 보다 유연성이 높다라고 할 수 있습니다. admission webhook의 형태로 구현된 admission controller의 사례는 다음과 같은 것들이 있습니다.

- Istio Envoy Sidecar: 파드 생성 요청이 오면 해당 파드에 Envoy Sidecar 컨테이너를 강제로 주입합니다.
- `StorageClass` 프로비저닝: PVC가 생성되는 것을 지켜보다가 자동으로 PVC에 미리 선언된 StorageClass를 주입합니다.

<br>

admission webhook의 구현과 관련된 자세한 사항은 본 글의 범위를 벗어나므로 자세한 사항은 [링크1](https://coffeewhale.com/kubernetes/admission-control/2021/04/28/opa1/), [링크2](https://medium.com/ibm-cloud/diving-into-kubernetes-mutatingadmissionwebhook-6ef3c5695f74) 등의 글을 참고해주세요.

> 본 글의 주제와 크게 관련이 없어 설명하진 않았지만 admission webhook이 작동하기 전에 `kube-apiserver`를 호출한 개체가 승인된 개체인지 인증(authentication)하는 과정과 그 개체가 특정 행위를 수행할 수 있는 권한이 있는지 인가(authorization)하는 과정이 먼저 선행됩니다.

<br>

# Admission Webhook 예시: Pod Identity Webhook

EKS 클러스터를 셋팅하면 기본적으로 pod-identity-webhook `MutatingWebhookConfiguration` 오브젝트가 생성됩니다. 이는 IRSA(IAM Role for Service Account)를 구현하기 위한 admission webhook, 좀 더 정확히는 MutatingAdmissionWebhooks admission controller가 어떤 조건을 만족 시켰을 때 어느 API 서버에 요청을 보내야할지를 정의해 놓은 매니패스트입니다(EKS는 AWS의 매니지드 K8S이므로 사용 편의성을 위해서 클러스터 생성시 자동으로 주입시켜주는 것 같습니다).

<br>

좀 더 자세히 말하면 pod-identity-webhook은 파드가 생성될 때 주입되는 `ServiceAccount`에 `medata.annotations.eks.amazonaws.com/role-arn`가 선언되어 있을 경우, 이에 맞게 파드의 환경 변수 설정을 변형(mutating)시키는 역할을 수행하는 admission webhook이라고 할 수 있습니다. 왜 하필 role-arn인지는 향후 포스팅에서 차차 자세히 정리할 예정이니 본 글에서는 변형(mutating)이 어떻게 일어나는지 결과 자체만 집중해 보죠.

<br>

먼저 아래와 같은 매니패스트를 배포 해보겠습니다. IAM Role이 지정된 `ServiceAccount`와 그 `ServiceAccount`를 활용하는 파드를 생성하는 매니패스트입니다.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::1234567890:role/mutating-webhook-tutorial-role  # 적절한 값으로 수정 필요
  name: mutating-webhook-tutorial-sa
  namespace: default
---
apiVersion: v1
kind: Pod
metadata:
  name: mutating-webhook-tutorial-pod
spec:
  serviceAccount: mutating-webhook-tutorial-sa
  containers:
  - name: mutating-webhook-tutorial
    image: busybox
    command: ['sh', '-c', 'echo Mutating Webhook Tutorial!! && sleep 3600']
```

<br>

위 매니패스트를 작성하고 쿠버네티스에 배포하면 `Pod`에 다음과 같은 값들이 추가적으로 설정되어 있음을 알 수 있습니다.

- env[*].name
- env[*].value

```yaml
apiVersion: v1
kind: Pod
spec:
  ...
    env:
    - name: AWS_DEFAULT_REGION
      value: ap-northeast-2
    - name: AWS_REGION
      value: ap-northeast-2
    - name: AWS_ROLE_ARN
      value: arn:aws:iam::1234567890:role/mutating-webhook-tutorial-role
    - name: AWS_WEB_IDENTITY_TOKEN_FILE
      value: /var/run/secrets/eks.amazonaws.com/serviceaccount/token
  ...
    volumeMounts:
    - mountPath: /var/run/secrets/eks.amazonaws.com/serviceaccount
      name: aws-iam-token
  ...
  volumes:
  - name: aws-iam-token
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          audience: sts.amazonaws.com
          expirationSeconds: 86400
          path: token
```

분명히 매니패스트를 배포할 당시에는 이와 같은 환경 변수를 어디에도 설정하지 않았습니다. 그렇데 어떻게 실제 배포된 파드에 저런 환경 변수들이 셋팅되어 있을까요? 이는 admission webhook인 pod-identity-webhook가 요청 내용을 가로채서 파드의 환경 변수에 `AWS_DEFAULT_REGION`, `AWS_REGION`, `AWS_ROLE_ARN`, `AWS_WEB_IDENTITY_TOKEN_FILE`를 생성하고, `serviceAccountToken`을 파드에 볼륨 마운트하는 내용으로 변형(mutating) 했기 때문입니다. 물론 검증(validating) 단계도 정상적으로 통과 했기 때문에 해당 요청 내용은 etcd에 적재 되었을 것입니다. 이와 관련된 좀 더 자세한 내용은 [링크](https://github.com/aws/amazon-eks-pod-identity-webhook#eks-walkthrough)를 참조해 보시면 좋을 듯 합니다.

<br>

# 마무리

간단하게 admission controller, 그 중에서 특별한 녀석들인 admission webhook에 대해서 알아 보았습니다. 쿠버네티스 api server에 보내지는 요청 내용을 중간에서 변형(mutating) 하거나 검증(validating) 할 수 있으므로 안 그래도 자유로운 쿠버네티스에 더 유연성을 더해줄 수 있는 컴포넌트라는 생각이 듭니다.  단편적인 사실만 놓고 보면 이게 어떻게 파드별로 권한을 제한 하는데 쓰일까 싶은데요. 마운트 되는 환경 변수를 자세히 보면 뭔가 AWS IAM과 관련된, 즉 권한을 관리하는 서비스와 밀접하게 관련되어 있다는 사실을 알 수 있습니다.

<br>

마지막으로 본 글에서는 IRSA를 이해하기 위한 최소한의 지식만을 정리했는데요. 사실 admission webhook도 정리하고자하면 할게 많습니다. 나중에 제가 현업에서 admission webhook를 구현해야 하는 순간이 올지도 모르겠네요. 향후 필요성이 느껴지면 좀더 심화된 수준에서 정리를 해보겠습니다.

<br>

# 참고자료

- [What are Kubernetes admission controllers?](https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/#what-are-kubernetes-admission-controllers)
- [Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [[Webhook\] 웹훅이란?](https://leffept.tistory.com/329)
- [쿠버네티스 Admission Control #1](https://coffeewhale.com/kubernetes/admission-control/2021/04/28/opa1/)
- [EKS에서 쿠버네티스 포드의 IAM 권한 제어하기: Pod Identity Webhook](https://tech.devsisters.com/posts/pod-iam-role/)
- [콜백 함수(Callback)의 정확한 의미는 무엇일까?](https://satisfactoryplace.tistory.com/18)
- [Diving into Kubernetes MutatingAdmissionWebhook](https://medium.com/ibm-cloud/diving-into-kubernetes-mutatingadmissionwebhook-6ef3c5695f74)