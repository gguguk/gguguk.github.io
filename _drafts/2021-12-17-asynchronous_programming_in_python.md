---
title: 파이썬 비동기(asynchronous) 프로그래밍
author: Gukwon Koo
categories: [CS, Basic]
tags: [asynchronous, python, process, thread]
pin: false
math: true
comments: false
date: 2021-12-11 14:46:00 +0900
---

파이썬의 비동기 프로그래밍을 이해하기 위해 필요한 지식을 정리합니다.

<br>

## 동기(synchronous)와 비동기(asynchronous) 프로그래밍

---

- 동기 프로그래밍: 작업을 순차적으로 처리하는 것
- 비동기 프로그래밍: 작업을 순차적으로 처리하지 않고 대기시간(waiting time)을 최소화 하는 방법

<br>

## 프로세스와 스레드

---

- 프로세스(process)
  - 메모리상에서 실행 중인 프로그램(디스크에 저장된 코드 덩어리)
  - 프로세스 내부에는 하나 이상의 스레드로 이루어져 있음
  - 운영체제는 실제로는 스레드 단위로 스케줄링을 함
- 스레드(thread) -> 일반적으로 소프트웨어 스레드
  - 프로세스 내부의 작업의 단위
  - 운영체제가 스케줄링하는 단위

<br>

## 스케줄링

---

- 운영체제가 한정된 자원인 CPU를 여러 프로세스의 스레드가 최대한 효율적으로 활용할 수 있도록 할당하는 작업을 말함
- 이를 통해 멀티 프로그래밍이 가능함

<br>

## 멀티 프로세싱 vs 멀티 스레딩 vs 코루틴

---

모두 concurrency를 달성하기 위한 방법

- cpu-bound
  - 프로세스에서 수행하는 작업이 대부분이 CPU 연산의 비중이 클 때
  - 멀티 프로세싱
    - 멀티 스레딩에 비해 context switching 비용이 큼
      - 프로세스끼리는 code, data, heap, stack 영역의 메모리를 공유하지 않음
  - 멀티 스레딩
    - 멀티 프로세싱에 비해 context switching 비용이 작음
      - 스레드끼리는 code, data, heap 메모리 영역을 공유함(stack 메모리 영역은 공유하지 않음)
    - 그러나 race condition 등의 문제가 발생할 수 있음
    - 파이썬 GIL 때문에 사용이 제한적임
- I/O-bound
  - 프로세스에서 수행되는 작업이 I/O(network I/O, 하드웨어 I/O)의 비중이 클 때
  - 코루틴
    - 단일 스레드에서 여러 중단점과 진입점을 왔다 갔다 하면서 작업을 수행
    - I/O 대기 시간을 최소화 하는 목적
    - I/O 작업을 하나의 코루틴에서 수행하고 응답이 올 때까지 다른 코루틴을 실행하고... 이러한 행위를 반복

<br>

## GIL

---

- 파이썬에서 한번에 하나의 스레드만 작업을 수행할 수 있음
- 즉 동시에 여러 스레드가 실행 될 수 없음

<br>

## Future 모듈

---

...

## Reference

- [asyncio : 단일 스레드 기반의 Nonblocking 비동기 코루틴 완전 정복](https://soooprmx.com/python-asycnio-%EC%97%90%EC%84%9C-%EB%9F%B0%EB%A3%A8%ED%94%84%EB%A5%BC-%EA%B8%B0%EB%B0%98%EC%9C%BC%EB%A1%9C-non-blocking-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0/?amp#fnref-6882-1)
- [파이썬의 새로운 병렬처리 API – Concurrent.futures](https://soooprmx.com/concurrent-futures/?amp)
- [스레드(Thread)와 스케줄링(Scheduling)](https://woo-dev.tistory.com/148)
- [Speed Up Your Python Program With Concurrency](https://realpython.com/python-concurrency/#how-to-speed-up-an-io-bound-program)