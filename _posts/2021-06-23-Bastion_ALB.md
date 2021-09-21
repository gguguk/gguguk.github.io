---
title: Bastion에서 internet-facing ALB로 요청을 보내고 응답 받기
author: Gukwon Koo
categories: [MLOps, AWS ECS]
tags: [mlops, aws, ecs, cpu, meory]
pin: false
math: true
comments: false
date: 2021-06-20 10:14:00 +0900
---

# Bastion에서 ELB로 요청을 보내고 응답 받기

<br>

> **Keyword**
>
> security group(보안 그룹), security group chaining, ELB(Elastic Load Balancer)

<br>

같은 AWS VPC 내부 존재하는 public ip를 가진 bastion에서 internet-facing ALB에 HTTP request를 보내서 응답을 받는 방법을 알아보려고 합니다. 사실 이 부분을 매우 쉽게 생각 했는데 생각 했던대로 요청이 오지 않아서 조금 당황하였습니다. 그래서 여러가지 문서를 찾아보고 security group chaining에 대해 제가 오해하고 있던 부분이 있었다는 것을 알게 되었습니다. 따라서 기본기를 잊지 말자라는 생각에 글을 정리해봅니다.

<br>

# Bastion에서 ELB로 HTTP request를 시도!

회사에서 업무를 하면서 개발 환경에서 머신러닝 모델 API에 응답 테스트를 해야할 일이 있었습니다. 개발 환경을 간략화 하면 다음과 같습니다. 같은 VPC 내부에 public ip를 가진 bastion과 public ip를 가진 ELB가 있습니다. 저는 bastion에서 ELB로 특정 형태의 request를 보냈고, 당연히 정상적인 응답이 돌아 올 것을 기대했습니다.

![](/assets/img/post_img/bastion_ELB.png)

<br>

# 어? 왜 time out error가 뜨지? 아... 보안 그룹 설정!

그러나 request가 ELB에 도달하지 못하고 time out error가 발생했습니다. 잠깐 생각해보니 ELB 보안 그룹에 bastion이 없었습니다. 잠깐의 실수를 반영하고 얼른 ELB의 보안 그룹을 다음과 같이 수정하였습니다. 

![](/assets/img/post_img/ELB_SG.png)

이때 security group chaining을 활용하였습니다. security group chaining은 보안 그룹의 source로 보안 그룹을 활용하는 방법인데요. 이는 source 보안 그룹이 할당된 instance의 ip를 허용하는 편리한 방법입니다.

> Allow inbound traffic from network interfaces (and their associated instances) that are assigned to the same security group.

Bastion instance의 ip를 허용해 주었으므로 당연히 정상 응답을 받을 수 있을 것이라고 기대했습니다.

# 그런데 또 time out error...

분명히 보안 그룹까지 뚫었는데 왜 time out error가 발생할까요? 해결 방법은 생각보다 간단했습니다. security group chaining을 제가 잘못 이해하고 있었습니다. AWS의 [security group document](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)의 *Security group rules* 섹션을 보면 다음과 같은 문장이 등장합니다.

> When you specify **a security group as the source** for a rule, traffic is allowed from the network interfaces that are associated with the source security group for the specified protocol and port. Incoming traffic is allowed based on the private IP addresses of the network interfaces that are associated with the source security group (**and not the public IP or Elastic IP addresses**). 

정리하면, security group을 source로 사용하는 security group chaing은 해당 security group이 할당된 인스턴스의 private ip의 traffic을 허용하는 것이지 **public ip의 traffic을 허용하는 것이 아니라는 것**입니다. bastion은 public subnet에 배치되어 있으므로 public ip를 가지게 되는데요. 제가 bastion에서 ELB에 HTTP request를 보내는 것은 출발지의 ip가 bastion의 public ip가 되고, 목적지의 ip는 ELB의 public ip가 되는 것입니다. **따라서 ELB는 bastion의 public ip의 traffic을 허용하도록 보안 그룹을 작성해야 request에 대한 정상적인 응답을 받을 수 있습니다.**

다음과 같이 ELB의 보안 그룹을 수정하면 됩니다.

![](/assets/img/post_img/bastion_ELB_public.png)

<br>

# Reference

- [Security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)