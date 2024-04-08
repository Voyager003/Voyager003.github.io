---
layout  : wiki
title   : 스레드보다는 실행자, 태스크, 스트림을 애용하라 
summary : 
date    : 2024-04-08 09:47:10 +0900
updated : 2024-04-08 11:09:11 +0900
tag     : java effectivejava
resource: 86/4744D6-3476-42F0-99C6-9CCE5A47153C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java Executor(실행자) 프레임워크

java.util.concurrent 패키지에는 Executor Framework(실행자 프레임워크)라는 인터페이스 기반의 태스크 실행 기능을 담고 있다.

이는 스레드 풀과 같은 구성 요소를 직접 개발하는 대신 유틸을 사용하여 여러 이점을 가져다 준다. (중략) [^1]

기능 중 하나로 작업 큐를 단 한줄로 생성할 수 있다.

```java
// 작업 큐 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// Executor에 실행할 작업을 넘기기
exec.execute(runnable);

// Executor 종료: 이 작업이 실패하면 VM 자체가 종료되지 않을 것
exec.shutdown();
```

## Executor 주요 기능

- get

특정 태스크가 완료되기를 기다린다.

- invokeAny && invokeAll

작업(task) 모음 중 아무것 중 하나(invokeAny) 혹은 모든 작업(invokeAll)가 완료되기를 기다린다.

- awaitTermination

ExecutorService 종료하기를 기다린다.

- ExecutorCompletionService

완료된 작업들의 결과를 차례로 받는다.

- ScheduledThreadPoolExecutor

작업을 특정 시간 혹은 주기적으로 실행하게 한다.

### 포크 조인(fork-join)

포크 조인 태스크는 포크 조인 풀이라는 특별한 ExecutorService가 실행해주는데, ForkJoinTask의 인스턴스는 작은 하위 태스크로 나눠, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하여 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수 있다. 이는 모든 스레드를 활용해 CPU를 최대 활용하면서 높은 처리량과 낮은 지연 시간을 달성한다.

## 정리

작업 큐를 손수 만드는 일을 삼가고 스레드를 직접 다루는 작업도 일반적으로 삼가야 한다.

스레드를 직접 다루게되면 스레드가 작업 단위와 수행 메커니즘 역할을 모두 수행하기 때문이다. 반대로 Executor는 작업 단위와 실행 메커니즘이 분리된다.

Executor에서 작업 단위를 나타내는 추상 개념은 태스크(task)로 크게 Runnable, Callable이 있으며, 태스크를 수행하는 일반적인 매커니즘이 바로 ExecutorService이다. 핵심은 Executor 프레임워크가 작업 수행을 담당해준다는 것!

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/1.5.0/docs/guide/concurrency/overview.html - concurrency utilities in oracle
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html - Executor docs

## 주석

[^1]: Using the Concurrency Utilities, instead of developing components such as thread pools yourself, offers a number of advantages: 
