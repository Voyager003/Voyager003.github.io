---
layout  : wiki
title   : JDK 21의 가상 스레드
summary : 
date    : 2023-12-12 20:12:07 +0900
updated : 2023-12-12 23:36:08 +0900
tag     : 
resource: E3/70B74E-1CAC-4F74-B868-CD4548CD6587
toc     : true
public  : true
parent  : 
latex   : false
---
* TOC
{:toc}

[카카오 기술세미나](https://www.youtube.com/watch?v=JZqF-QKY6GM&ab_channel=kakaotech) 의 JDK 21의 VirtualThread 알아보기를 요약한 글

## 가상 스레드(Thread)와 탄생 배경

> JDK 21에 추가된 경량 스레드(Thread) OS 스레드를 그대로 사용하지 않고 JVM 내부 스케줄링을 통해 수십만~수백만개 동시에 사용할 수 있게 한다.

전통적인 Java의 스레드는 OS 스레드를 Wrapping한 것(플랫폼 스레드)이다. Java 애플리케이션에서 스레드를 사용하면 실제로는 OS 스레드를 사용한 것.

OS 스레드는 생성 갯수가 제한적이고 생성하고 유지하는 비용이 비싸다. 때문에 애플리케이션에서는 플랫폼 스레드를 효율적으로 사용하기 위해 스레드 풀(Thread Pool)을 사용했다.

이러다보니 처리량(Throughput)에 제약이 생김.

기본적인 Web Request 처리방식은 Thread Per Request(하나의 요청 당 하나의 스레드를 처리)를 사용한다. 이 때 처리량을 높이려면 스레드가 많이 필요한데, Java의 전통적 스레드는 OS Thread를 Wrapping 한 구조였기 때문에, 무한정 늘릴 수 없는 OS 스레드에 제약이 있었다. 이는 처리량에 한계에 다다르게 되었다.

이 방식은 무거운 트래픽과 높은 처리량을 감당하는 경우에는 OS 스레드의 한계를 벗어나야만 했다.

![스크린샷 2023-12-12 오후 7 19 49](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/1b65ff27-00dd-4cb3-931f-48a955c3da05)

또한 Blocking I/O 문제도 있다. 스레드에서 I/O 작업을 처리할 때 Blocking이 일어나는데, 작업을 처리하는 시간보다 대기하는 시간이 훨씬 길어 효율적으로 사용할 수 없는 문제가 발생했다.

이런 점을 극복하기 위해 Reactive Programming을 도입했는데 (Webflux), Webflux 스레드를 대기하지 않고 다른 작업을 처리하도록 했다. 

하지만 리액티브 프로그래밍을 작성하고 이해하는 비용이 높고, Reactive하게 동작하는 라이브러리 지원을 필요로 한다. 평소에 동기적 방식에서 비동기적인 방식을 사용하려는데 어려움을 겪었다.

이는 Java의 Design을 살펴봐야 한다. 

Java의 디자인은 Thread 중심으로 구성되어 있다. Exception Stack trace, Debugger, Profiling 모두 스레드 기반인데, Reacitve 할 때 작업이 하나의 스레드가 아닌 여러 스레드를 거쳐 처리되어, 위에서 말한 작업(Excpetion Stack trace..)이 어렵고 컨텍스트 확인이 어려워 디버깅이 어려워지는 사이드 이펙트를 가져왔다.

## 가상 스레드의 구조 

구조를 살펴보기 전에 가상 스레드가 해결하고자 하는 문제를 구분하면 두가지로 다음과 같다.

- 애플리케이션의 높은 처리량(throughput)을 확보 -> Blocking 발생 시 내부 스케줄링을 통해 다른 작업을 처리

- Java 플랫폼의 디자인과 조화를 이루는 코드를 생성 -> 기존 스레드 구조를 그대로 사용

```java
sealed abstract class BaseVirtualThread extends Thread 
		permits VirtualThread, ThreadBuilders.BoundVirtualThread {
		...
}
```

![스크린샷 2023-12-12 오후 7 27 15](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/1d64a59c-08ec-42e5-8d0c-22f4bd1611e1)

가상 스레드는 Reactive 스타일의 높은 처리량과 MVC 스타일의 동기적인 코드 작성이라는 두 장점을 모두 취할 수 있었다.

### 구조 

![스크린샷 2023-12-12 오후 7 28 38](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/b41575e1-ef3d-42e0-a6e5-b0c17f538c6c)


플랫폼 스레드는 애플리케이션이 스레드 풀안의 플랫폼 스레드를 사용하여 OS 스레드와 1:1로 매핑되는 구조를 가진다.

![스크린샷 2023-12-12 오후 7 28 48](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/be46f393-7295-4db9-aee4-828cc1cff5e4)

가상 스레드는 JVM 내부에서 가상 스레드가 따로 존재하여 folk/join 풀에 캐리어 스레드를 두고 사용한다.

플랫폼 스레드의 스레드 풀과 유사하지만, 명시적으로 플랫폼 스레드를 사용하는 것이 아닌 가상 스레드가 이를 결정한다. 

결국 캐리어 스레드와 OS 스레드가 매칭되지만, 실제 애플리케이션에서는 플랫폼 스레드가 아닌 가상 스레드만을 사용하게 된다.

![스크린샷 2023-12-12 오후 7 33 27](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/f233f328-ae7d-4a93-ac65-c5f028ba9454)

가상 스레드를 좀 더 자세하게 살펴보면 가상 스레드가 작업을 할당 받아 캐리어 스레드와 연결되어있다.

이 때 블로킹이 발생되면 블로킹이 발생된다면 캐리어 스레드에서 unmount된다. 이 후, unmount된 캐리어 스레드는 휴식 중일때는 다른 가상 스레드와 mount가 된다.

기존에는 I/O 관련하여 블로킹이 발생하면 플랫폼 스레드가 대기했지만, 가상 스레드를 사용해 블로킹 발생하는 구간에서 mount, unmount를 수행함으로써 다른 가상 스레드 task를 수행할 수 있게 된다.

이는 OS 스레드는 개수의 제한에 있지만, 가상 스레드의 수는 수십~수백만개까지 생성하여 사용할 수 있다.

하지만 가상 스레드의 갯수가 계속 많아지면 이를 관리하여 사용하는 자원도 작게 유지를 해야한다.

사용하는 자원의 차이는 다음과 같다.

![스크린샷 2023-12-12 오후 7 40 45](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/2ec6217e-7ab1-4080-adc4-00ab11ec0498)

## 사용법

![스크린샷 2023-12-12 오후 7 42 32](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/ff7c3190-9d5d-41a1-9404-f72475ecbcfd)

실제 코드를 작성할 때 low level을 직접 핸들링 하는 경우는 없다.

ExceutorService의 newVirtualThreadPerTaskExecutor() 메서드가 추가되었는데, 이는 가상스레드로 동작 유무를 확인할 수 있다.

```
// applicaiton.yml 3.2 이상
spring:
	threads:
		virtual:
			enabled: true
}
```

Spring boot 3.2 이상은 application.yml에서 설정할 수 있다. 

## 유의점 

먼저 Platform Thread 가 가상 스레드로 갈 때 기존 전통적 스레드 풀 관점에서 본다면 차이점이 있다.

스레드라는 전통적 OS의 자원보다는 하나의 Task 별로 할당하는 단위로 생각하는 것이 좋다.

그리고 Thread Local에서 플랫폼 스레드 풀 사용 시, 공유를 위해 스레드 로컬을 사용하던 관습적인 패턴들이 있는데, 가상 스레드는 Heap을 사용하기 때문에 수십~수백만개까지 늘어날 수있는 점을 고려하여 메모리 이슈가 발생하게 된다. 

또한 synchronized 사용시 가상 스레드에 연결된 캐리어 스레드가 블로킹될 수 있어 주의해야한다. 

가상 스레드가 블로킹을 안하게끔 설계되어있긴 하지만, 내부에서 synchronized나 JNI와 같은 native call을 할 때는 캐리어 스레드에서 블로킹을 막을 수 없다. (pinning)

![스크린샷 2023-12-12 오후 7 53 51](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/72e9929b-5300-47ca-83cf-638ff2fbbb6d)

pinning을 회피하기 위해서는 synchnized 작성된 코드를 ReemtrantLock으로 우회하는 방식이 있다.

또는 pinning이 발생 여부를 확인하는 디버깅 구간을 탐지하여 관련 코드를 변경해야 한다.

## 성능 테스트 

![스크린샷 2023-12-12 오후 7 57 49](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/d34fc4db-723c-41d4-aa25-90b9b951f8b3)

![스크린샷 2023-12-12 오후 7 58 08](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/20b27978-dd0f-457d-a8a5-8a76f51a9055)

플랫폼 스레드에서는 톰캣이 요청을 수행하다가 충분한 트래픽을 받지 못하면 기다려야하는 상황이 발생했는데, 가상 스레드는 처리량을 모두 소화하고, DB 커넥션을 기다리다가 타임아웃이 발생하여 예외가 발생했다.

이를 종합해보면 I/O 블로킹이 발생하는 경우 가상 스레드가 더 좋은 처리량을 보여주지만, 톰캣의 서블릿이 가상스레드로 처리량을 뒤로 넘길 때 DB 커넥션을 가져오려다 타임아웃이 발생. 이런경우를 Overwhelming이라 한다.

기존 스레드 풀을 사용하던 이유는 플랫폼 스레드가 비싸기 때문이기도 했지만, 일종의 Throttle 역할도 수행했다. 

제한된 리소스 파일 I/O에 대한 접근이 제한된 갯수만 허용할 때는 세마포어(semaphore)로 사용해 제약이 필요하다. 

## 정리

- 적합한 사용처
    - I/O Blocking이 발생하는 경우 가상스레드가 적합
    - CPU Intensive 작업(동영상 인코딩과 같은 작업)은 I/O intensive가 아닌 CPU intensive 작업으로 적합하지 않다.
    - Spring MVC 기반 Web API 제공 시 편리하게 사용할 수 있다. (높은 throughpout을 위해 webflux를 고려중이라면 대안이될 수 있다)


- 오해 
    - 기존 플랫폼 스레드를 대체하는 것이 목적이 아니다. 서로 반대되는 개념이 아닌 플랫폼 디자인과의 조화 가능한 기술
    - 도입한다고 무조건 처리량이 높아지지 않는다. 단지 기다림(대기)에 대한 개선
    - 가상 스레드 그 자체로 Java의 동시성을 완전히 개선했다고 보기는 어렵다.

- 제약
    - 스레드 풀에 적합하지 않다 task 별로 가상 스레드를 할당해야함
    - 스레드 로컬 사용 시, 메모리 사용이 늘어날 수 있다.

