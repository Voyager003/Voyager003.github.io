---
layout  : wiki
title   : 프로그램의 동작을 스레드 스케줄러에 기대지 말라 
summary : 
date    : 2024-05-06 09:41:41 +0900
updated : 2024-05-06 11:24:42 +0900
tag     : java effectivejava
resource: 2F/FE5EEE-F735-4CFD-BF8C-B595B7232632
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## OS 스케줄링 정책 

여러 스레드가 실행 중이라면 OS의 스레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할지 결정한다. 여기서 구체적인 스케줄링 정책은 OS마다 다르기 때문에 잘 작성된 프로그램은 이 스케줄링 정책에 좌지우지돼선 안된다.

'정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다' 

견고하고 이식성 좋은 프로그램을 작성하는 가장 좋은 법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록하여 스케줄러가 고민할 거리를 줄여주는 것이다.

### 적절한 Thread pool

스레드는 당장 처리해야 할 작업이 없다면 실행되서는 안된다. Executor 프레임워크를 예로 들어보자.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {
    public static void main(String[] args) {
        // 스레드 풀 생성
        ExecutorService executor = Executors.newFixedThreadPool(5); // 최대 5개의 스레드를 가지는 풀 생성

        // task
        for (int i = 0; i < 10; i++) {
            Runnable task = new WorkerThread("Task " + i);
            executor.submit(task); // 작업을 스레드 풀에 submit
        }

        // 작업 완료 후 스레드 풀 종료
        executor.shutdown();
    }
}

class WorkerThread implements Runnable {
    private String taskName;

    public WorkerThread(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " Start. Task = " + taskName);

        System.out.println(Thread.currentThread().getName() + " End.");
    }
}
```

이 때, 작업이 너무 짧다면 작업을 분배하는 부담이 오히려 성능을 떨어뜨릴 수 있어 적절한 합의점이 필요하다.

### 바쁜 대기상태(busy waiting)

또한 스레드는 바쁜 대기상태가 되면 안된다. 이는 공유 객체의 상태가 바뀔 때까지 쉬지않고 검사해선 안된다는 것이다. 이 상태는 스레드 스케줄러의 변덕에 취약하며, 프로세서에 큰 부담을 줘 다른 작업이 실행될 기회를 박탈하게 된다.

```java
// CountDownLatch - busy waiting
public class SlowCountDownLatch {
    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }

    public void await() {
        while (true) {
            synchronized(this) {
                if (count == 0)
                    return;
            }
        }
    }
    
    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```

코드를 보면 await() 메서드 내에서 무한 루프를 돌면서 계속해서 상태를 확인하고 있다. 

현재 카운트가 0이 될 때까지 기다리는 메서드인데, 이때 카운트가 0이 아닌 경우에는 계속해서 루프를 돌며 상태를 확인한다. 이러한 방식은 CPU 자원을 계속해서 소모하면서 기다리는 동작을 수행하는데, 이러한 방식을 busy waiting 또는 spinlock이라고 한다.

자원을 효율적으로 사용하지 못하고 CPU를 계속해서 사용하여 시스템의 성능을 저하시킬 수 있어 대기 중인 프로세스가 아무런 일도 하지 않으면서도 시스템 자원을 소모하기 때문에 비효율적이다. 

### Thread.yield

특정 스레드가 다른 스레드와 비교해 CPU 시간을 충분히 얻지 못하는 프로그램을 보더라도 Thread.yield를 써서 해결하지 말자.

yield 메서드는 테스트할 수단도 없으며, 오히려 JVM 성능을 느려지게 할 수도 있다.

## 정리

스레드 우선순위는 Java에서 이식성이 가장 나쁜 특성에 속한다. 스레드 몇 개의 우선 순위를 조율해 반응 속도를 높이는 방법은 이식성이 떨어진다. 서비스 품질을 위해 프로그램을 고치는 용도로 사용해선 안된다!

## 참고자료

- 이펙티브자바 3판

