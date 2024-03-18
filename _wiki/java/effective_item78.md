---
layout  : wiki
title   : 공유 중인 가변 데이터는 동기화해 사용하라 
summary : 
date    : 2024-03-18 09:43:19 +0900
updated : 2024-03-18 14:52:37 +0900
tag     : java effectivejava
resource: 83/F3F576-415A-4729-98B1-648601AD888C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 동기화(synchronization)와 경쟁 상태(race condition)

동기화는 `시스템을 동시에 작동시키기 위해 여러 사건들을 조화시키는 것` 이라는 사전적 의미를 가진다. 

사전적 의미와 크게 다르지는 않지만 컴퓨터 과학에서 동기화는 `프로세스 또는 스레드들이 수행 시점을 조절하여 서로 알고 있는 정보를 일치하는 상태`를 의미한다.

CPU를 최대한 활용하기 위해 하나의 프로세스 내에서 둘 이상의 스레드를 동시에 작업을 수행하는 일이 빈번해졌는데, 이 멀티 스레드 환경에서는 두 개 이상의 스레드가 동시에 변경 가능한 공유 데이터를 업데이트를 시도할 때 경쟁 상태가 발생한다. 

### 동기화를 하는 이유

```java
@Getter @Setter
public class SynchronizedMethods {

    private int sum = 0;

    public void calculate() {
        setSum(getSum() + 1);
    }
}
```

간단하게 합계를 계산하는 calculate() 메서드를 실행하는 경쟁 상태를 고려해보자.

```java
@Test
public void givenMultiThread_whenNonSyncMethod() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    SynchronizedMethods summation = new SynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(summation::calculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, summation.getSum());
}
```

스레드 풀과 작업(task) 할당을 위한 API인 ExecutorService를 사용해 세 개의 스레드로 구성된 스레드 풀을 생성 한 뒤에, 0부터 999까지의 범위를 생성하고 스레드 풀에 작업을 할당하고 1초 동안 스레드 풀이 종료될 때까지 기다리도록 했다.

반복문을 모두 마치고 getSum() 메서드를 실행한 뒤 기대 값은 1000이지만, 내 컴퓨터에서는

```java
java.lang.AssertionError: expected:<1000> but was:<959>
```

959라는 결과를 볼 수 있었다. 이유는 멀티 스레드 환경에서 스레드가 동시에 calculate 메서드를 호출하여 sum 변수를 수정했기 때문에 발생한 것.

### synchronized 키워드

Java에서는 이런 문제를 해결하기 위해 synchronized 키워드로 해당 메서드 혹은 블록을 한 번에 하나의 스레드가 수행하도록 보장하도록 돕는다.

>> synchronized statement는 실행 중인 스레드를 대신하여 mutual-exclusion lock을 획득하고 블록을 실행한 다음 lock을 해제한다. 실행 중인 스레드가 lock을 소유하는 동안 다른 스레드는 lock을 획득할 수 없다. [^1]

lock을 건 메서드는 객체의 상태를 확인하고, 필요하면 수정하여 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. 

## 스레드 간 통신

동기화는 배타적 수행뿐만 아니라 한 스레드가 만든 변화를 다른 스레드에서 확인하는 `스레드 간 통신`을 돕는다.

일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 lock의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

[Java docs](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.7) 에 따르면 비휘발성(non-volatile) long과 double의 변수를 읽고 쓰는 동작은 single write는 32비트 절반에 각각 하나씩 두번의 separate write로 처리된다. 이를 제외한 변수를 읽고 쓰는 동작은 원자적(atomic)이다.

이는 여러 스레드가 같은 변수를 동기화없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 의미이다.

여기서 주의할 점은 성능을 높이려고 원자적 데이터를 읽고, 쓸 때 동기화하지 않겠다는 발상이다.

이는 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 [Java의 메모리 모델](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4) 때문이다. 코드로 살펴보자.

```java
public class ThreadEx {
    private static boolean stopRequested;
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i= 0;
            while (!stopRequested) {
                i++;
            });
            backgroundThread.start();
            
            TimeUnit.SECONDS.sleep(1);
            stopRequested = true
        }
    }
}
```

메인 스레드에서 1초 후에 stopRequested 필드를 true로 설정하고 반복문을 탈출하여 종료될 것이라고 예상했지만 종료되지 않고 계속 수행되었다.

원인은 동기화를 하지 않아서 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤 보게 될지 보증할 수 없기 때문이다.

여기서 stopRequested 필드를 동기화해 접근한다면?

```java
public class ThreadEx {
    private static boolean stopRequested;
    
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }
    
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i= 0;
            while (!stopRequested()) {
                i++;
            });
            backgroundThread.start();
            
            TimeUnit.SECONDS.sleep(1);
            requestStop();
        }
    }
}
```

requestStop(쓰기 메서드)와 stopRequested(읽기 메서드)를 두 개 모두 synchronized 키워드로 선언하여 동기화했다. 

여기에서 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있는데, synchronized 키워드로 인한 동기화 덕분에 lock의 보호 아래 수행된 수정사항을 다른 스레드에서 최종 결과를 볼 수 있게 된다. 즉, `스레드 사이의 안정적인 통신을 도운 것`이다.

주의할 점은 `쓰기, 읽기 메서드 모두 동기화 되지 않으면 동작을 보장하지 않는다`는 것.

### volatile 필드

위의 예시에서 볼 수 있는 반복문에서 매번 동기화하는 비용은 그렇게 크지는 않지만 item78은 더 빠른 대안을 제시한다. volatile 필드이다.

>> Java는 스레드가 공유 변수(shared variables)에 액세스할 수 있도록 허용한다. 일반적으로 공유 변수가 일관되고 안정적으로 업데이트되도록 하기 위해 스레드는 일반적으로 공유 변수에 대해 상호 배제를 강제하는 lock을 획득하여 해당 변수를 독점적으로 사용하도록 해야 한다. [^2]

volatile는 동기화의 효과 중 스레드 간의 통신만 지원하며 배타적 실행을 지원하지 않는다.

```java
public class ThreadEx {
    private static volatile boolean stopRequested; // volatile 추가
     
    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i= 0;
            while (!stopRequested()) {
                i++;
            });
            backgroundThread.start();
            
            TimeUnit.SECONDS.sleep(1);
            stopRequested = true;
        }
    }
}
```

volatile 한정자는 동기화의 기능 중 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다. 기대한 것과 같이 스레드가 정상적으로 종료된다.

여기서 volatile를 사용할 때 주의할 점이 있다.

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

generateSerialNumber 메서드는 매번 고유한 값을 반환할 의도로 만들어 졌다.

volatile 한정자로 인해 동기화하지 않더라도 불변식을 보호할 수 있어 보이지만, 동기화없이 의도한대로 동작하지 않는다.

원인은 증가 연산자(++)로 코드 상으로는 하나지만 실제로는 nextSerialNumber 필드에 두 번 접근하게 된다. 따라서 값을 읽고, 새로운 값을 저장하게 된다. 

여기서 두 번째 스레드가 이 사이에 접근하여 값을 읽게 된다면 첫 번째 스레드와 같은 값을 돌려받게 된다. (안전 실패: safety failure)

generateSerialNumber 메서드에 synchronized 한정자를 붙이게 되면 동기화되어 이 문제를 해결할 수 있다. 다른 스레드에서 동시에 호출하더라도 lock이 걸려 있어 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다.

여기서 메서드에 synchronized 한정자를 명시하려면 nextSerialNumber 필드의 volatile을 제거해야 한다는 점을 알아두자.

### java.util.concurrent.atomic 

lock 없이(lock-free)도 thread-safe한 프로그래밍을 지원하는 클래스를 담은 패키지이다. 

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
} 
```

내부적으로 CAS(Compare And Swap) 연산을 사용하게 되는데, 다음과 같이 동작한다.

- 연산할 메모리 위치(M)
- 변수의 기존 예상 값(A)
- 설정해야 하는 새 값(B)

M의 값을 B로 원자적으로 업데이트하지만, M의 기존 값이 A와 일치하는 경우에만 업데이트하고 그렇지 않은 경우에는 아무 조치도 취하지 않는다. 

여러 스레드가 CAS 연산을 통해 동일한 값을 업데이트하려고 시도하면 그 중 하나가 승리하여 값을 업데이트한다. 그러나 lock의 경우와 달리 다른 스레드는 일시 중단되지 않고 단순히 값을 업데이트하지 못했다는 알림만 받게 된다. 그러면 스레드는 다른 작업을 계속 진행할 수 있으며 context-switch가 완전히 방지된다.

## 정리

item78을 요약하면 `동기화는 배타적 실행 뿐만 아니라 안정적인 통신에 필수 적` 이라는 것!

가변 데이터는 단일 스레드에만 사용하도록 하고, 이 정책을 받아들였다면 이를 문서에 남겨 유지보수 과정에도 지켜지도록 하자.

## 참고자료

- 이펙티브자바 3판
- https://ko.wikipedia.org/wiki/%EB%8F%99%EA%B8%B0%ED%99%94 - wiki의 동기화 설명
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html -executorservice api
- https://docs.oracle.com/javase/specs/jls/se17/html/jls-14.html#jls-14.19 - the synchronized statement
- https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.3.1.4 - volatile 키워드
- https://www.baeldung.com/java-atomic-variables - java atomic

## 주석

[^1]: A synchronized statement acquires a mutual-exclusion lock on behalf of the executing thread, executes a block, then releases the lock. While the executing thread owns the lock, no other thread may acquire the lock.

[^2]: The Java programming language allows threads to access shared variables. As a rule, to ensure that shared variables are consistently and reliably updated, a thread should ensure that it has exclusive use of such variables by obtaining a lock that, conventionally, enforces mutual exclusion for those shared variables.
