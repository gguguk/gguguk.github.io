---
title: ECS의 task definition에서 soft/hard memory limit의 의미
author: Gukwon Koo
categories: [MLOps, AWS ECS]
tags: [mlops, aws, ecs, cpu, meory]
pin: false
math: true
comments: false
date: 2021-06-20 10:14:00 +0900
---

> 본 포스트는 [How Amazon ECS manages CPU and memory resources](https://aws.amazon.com/ko/blogs/containers/how-amazon-ecs-manages-cpu-and-memory-resources/)라는 글의 도움을 가장 많이 받았습니다. 본 포스트에 정리된 내용 보다 더 자세한 내용을 알고 싶으시면 해당 글을 참고해주세요.

<br>

최근 ECS를 다루면서 task definition을 정의할 때 container의 memory resource를 설정해야하는 일이 있었습니다. 이때 아래 그림과 같이의 container의 memory limit 설정을 해야 했는데요. document를 읽어봐도 정확히 어떤 의미인지 이해하기 어려웠습니다. 따라서 관련된 내용을 찾아보았고 task의 lauch type에 따라서 다른 의미를 가지게 된다는 것을 알게 되었습니다. 저희 팀에서는 인프라 관리하는 일에 최대한 신경을 덜 쓰기 위해 container instance로 Fargate를 선택하였는데요. 따라서 본 포스트에서는 Fargate lauch type을 선택했을 때 memory limit 설정에서 `soft limit`와 `hard limit`가 가지는 의미를 중점적으로 살펴보겠습니다. 

참고로 본 포스트는 docker container와 AWS ECS에 대한 기본적인 개념을 알고 있다는 가정에서 작성되었습니다. 틀린 설명이 있다면 지적해 주시면 감사 하겠습니다.

<br>

![](/assets/img/post_img/ECS_task_definition_memory_limitation.png)

<br>

---

# ECS Document를 먼저 보자

가장 먼저 task definition의 parameter가 설명되어 있는 [document](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions)를 살펴봅시다. Container Definitions의 `memory`와 `memoryReservation` 부분의 설명을 보면 다음과 같이 설명되어 있습니다.

> `memory`
>
> The amount (in MiB) of memory to present to the container. If your container attempts to exceed the memory specified here, the container is killed. 
>
> `memoryReservation`
>
> The soft limit (in MiB) of memory to reserve for the container. When system memory is under contention, Docker attempts to keep the container memory to this soft limit

`memory` 파라미터는 container가 쓸 수 있는 메모리의 총량을 설정하는 것이고, container가 설정된 용량을 넘어서 메모리를 사용하려고 시도하면 컨테이너가 죽는다고 하네요. 음... 이정도면 어느 정도는 이해가 되는 것 같습니다. 컨테이너는 container instance를 기반으로 띄워지는 process인데 해당 process가 활용할 수 있는 메모리의 총량을 제한하는 것이군요.

그런데 `memoryReservation` 파라미터의 설명을 보면 한번에 이해가 가지는 않습니다. 일단 정의를 잃어보면, 컨테이너를 위해 예약된(reserve) 메모리의 soft limit라고 하네요. 이 말도 이해가 잘 안가고... 또 시스템 메모리가 경쟁 상태에 있으면 도커 데몬이 설정된 soft limit 만큼의 메모리를 컨테이너를 위해 keep을 한다고 합니다. 일단 전반적으로 어떤 말인지 잘 이해가 가지 않습니다. 그리고 메모리를 예약(reserve) 한다는 개념도 생소하고, 이 것이 왜 필요한지 모르겠네요. 배경 설명이 필요할 것 같습니다.

<br>

---

# General rules of thumb with containers

먼저 도커 컨테이너가 호스트의 리소스와 어떻게 상호 작용하는 지에 대해 간단히 정리해봅시다.

- [도커 컨테이너는 프로세스](https://www.44bits.io/ko/post/is-docker-container-a-virtual-machine-or-a-process)입니다. 다시 말해, 도커 컨테이너는 호스트에 떠 있는 여러 프로세스 중 하나일 뿐입니다.
- 프로세스들은 호스트의 cpu, memory, 기타 자원을 공유(share)합니다.
- 특별한 이유가 없다면, **일반적으로 컨테이너는 호스트의 모든 cpu 및 메모리 자원에 접근하고 이를 활용할 수 있습니다.**
- 특별한 이유가 없다면, **같은 호스트에 떠 있는 컨테이너들은 호스트의 CPU, memory, 기타 자원들을 공유합니다.** 이는 호스트에 떠 있는 프로세스들이 자원을 공유하고 있는 것과 같은 이치입니다.

사실 위의 설명은 컨테이너만이 가지는 특성이라기 보다는 리눅스 프로세스들의 일반적인 특성입니다.

---

# ECS resource management options

ECS에서 리소스와 관련된 설정은 크게 두 가지로 구분할 수 있습니다.

- Task > Host
  - task가 host의 리소스를 얼마나 예약해 둘 수 있으며 활용 가능한 리소스의 최대치를 얼마나 둘지에 대한 설정입니다.
  - launch type이 EC2일 때만 설정할 수 있습니다.
  - 본 포스트에서 다루지 않습니다.
- Container > Task
  - **Task 내부의 container가 Task의 리소스**를 얼마나 예약해 둘 수 있으며, 활용 가능한 리소스의 최대치를 얼마나 둘지에 대한 설정입니다.
  - launch type이 EC2일 때와 **Fargate**일 때 모두 설정할 수 있습니다.
  - 본 포스트에서 다루는 부분입니다.

이와 같은 리소스에 관한 설정은 아래와 같이 task definition에서 정의됩니다.

```json
{
  "containerDefinitions": [
    {
      "cpu": 0,
      "memory": null,
      "memoryReservation": 1024,
    }
  ],
  "cpu": "1024",
  "memory": "2048",
  "compatibilities": [
    "EC2",
    "FARGATE"
  ],
}
```

<br>

그렇다면 리소스를 예약하는 것이 무슨 의미를 가지며 왜 필요한 걸까요? *General rules of thumb with containers* 섹션에서 언급한 것처럼 하나의 호스트에는 여러 개의 프로세스가 떠 있으며 해당 프로세스들은 호스트의 리소스를 공유합니다.

리소스를 공유한다는 것은 해당 리소스를 활용하는 대상들(e.g., 컨테이너) 사이에 자원에 대한 경쟁(contend)이 발생할 수 있다는 것을 의미합니다. 만일 ECS에 올려서 배포하려는 어플리케이션이 다른 프로세스들과의 리소스 경쟁에서 밀린다면 해당 어플리케이션 기반의 서비스가 제대로 작동할 수 있을까요? 당연하게도 해당 애플리케이션은 제대로 동작하지 않을 것이고 이러한 서비스는 고객들의 불편함을 초래할 것입니다. 

만일 서비스를 제공하는 컨테이너만이 활용할 수 있도록 미리 리소스를 예약(reservation) 해두면 어떨까요? 예약된 리소스는 해당 컨테이너만 활용할 수 있기 때문에 리소스 경쟁 때문에 애플리케이션이 실패하는 일을 미연에 방지할 수 있을 것입니다.

> Reservations are effectively telling ECS that “this specific containers needs this much memory and this much CPU” 

---

# Soft Memory Limit vs Hard Memory Limit

container instance로 Fargate를 선택한 경우, task가 host의 자원을 어떻게 활용할지에 대한 설정은 필요가 없습니다. EC2 launch type은 하나의 EC2에 복수 개의 task를 올릴 수 있고 각 task에는 복수 개의 컨테이너를 올릴 수 있습니다. 따라서 task를 만들 때 EC2 launch type을 선택하면 task가 호스트의 자원을 활용하는 설정와 container가 task의 자원을 활용하는 설정, 두 가지가 모두 필요합니다. 

그러나 Fargate launch type은 각자가 완벽하게 분리되어 있기 때문에 각 task는 서로 간에 CPU나 메모리 등의 자원을 절대 공유하지 않습니다. 따라서 container가 task의 자원을 활용하는 것과 관련된 설정만 진행하면 됩니다.

> Each Fargate task has its own isolation boundary and does not share the underlying kernel, CPU resources, memory resources, or elastic network interface with another task.

task definition의 `containerDefinitions`에 `memoryReservation` 필드가 예약 메모리와 관련된 설정입니다. 그리고 우리가 서두에서 보았던 memory의 `soft limit`는 사실 `memoryReservation` 필드에 들어갈 value를 정하는 것이었습니다. 한편 컨테이너가 활용할 수 있는 리소스의 최대치도 정할 수 있습니다. 이것이 바로 memory의 `hard limit` 설정이며, 해당 컨테이너가 `hard limit`를 초과하는 메모리를 사용하게 되면 해당 컨테이너는 죽게 됩니다. `hard limit`는 task definition에서 `memory`라는 필드명으로 등록됩니다.

> Hard and soft limits correspond to the `memory` and `memoryReservation` parameters, respectively, in task definitions.

memory의 soft limit와 hard limit는 둘 다 선택할 수 있거나 둘 중 하나만 선택할 수도 있습니다. 각각의 경우에는 다음과 같은 의미를 가집니다.

- soft limit만 설정
  - 예약(reservation) 메모리: specified soft limit value
  - 최대(ceiling) 메모리: specified task memory size
- soft limit와 hard limit 둘 다 설정
  - 예약(reservation): specified soft limit value
  - 최대(ceiling): specified hard limit value
- hard limit만 설정
  - 예약(reservation): specified hard limit value
  - 최대(ceiling): specified hard limit value

마지막으로 soft limit와 hard limit 둘 다 설정 했을 때의 예시를 통해 soft limit와 hard limit의 동작 원리를 이해해 봅시다.

> soft limit(`memoryReservation`): 256MB
> hard limit(`memory`): 512MB

위와 같은 경우에는 컨테이너에 기본적으로 256MB의 메모리가 할당(예약)됩니다. 물론 예약된 메모리를 100% 다 활용하지 않을 수도 있습니다. 그러나 만일 메모리 사용량이 512MB를 넘어서게 되면 해당 컨테이너는 자동으로 종료됩니다.

---

# Summary



우리는 AWS document에 memory 및 memoryReservation 파라미터를 이해 해보려고 노력하는 것에서 이야기를 시작해보았습니다. 이해가 가는 부분도 있고, 배경 지식이 없으면 이해되지 않는 부분도 있었죠. 특히 메모리 "예약"이라는 개념을 이해하기 위한 설명이 주를 이루었습니다. 도커 컨테이너에 대한 일반적인 이야기와 ECS에서 리소스를 관리하는 여러가지 설정에 대한 이야기도 했습니다. 이를 통해서 메모리 예약이 왜 필요하고 ECS에서 어떤 파라미터가 이와 관련된 설정을 지정할 수 있는지에 대해서도 배웠습니다. 여기까지 글을 정독 하셨다면 ECS document에 적혀 있는 `memory` 및 `memoryReservation` 부분을 무리 없이 이해하실 수 있을 것 같습니다. 혹시 잘못된 설명이 있었다면 지적 부탁드립니다.



---

# References

- [How Amazon ECS manages CPU and memory resources](https://aws.amazon.com/ko/blogs/containers/how-amazon-ecs-manages-cpu-and-memory-resources/)
- [Task definition parameters - container definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html#container_definitions)
- [Amazon ECS on AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Amazon ECS CloudWatch metrics](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-metrics.html)
- [Amazon ECS on AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Container Instance Memory Management](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/memory-management.html)
- [도커 컨테이너는 가상머신인가요? 프로세스인가요?](https://www.44bits.io/ko/post/is-docker-container-a-virtual-machine-or-a-process)