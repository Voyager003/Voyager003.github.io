---
layout  : wiki
title   : wait와 nofify보다는 동시성 유틸리티를 애용하라 
summary : 
date    : 2024-04-15 10:01:36 +0900
updated : 2024-04-15 22:00:01 +0900
tag     : java effectivejava
resource: 19/CE1637-BB0F-475E-B45A-46F02CA10ACD
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java Thread wait notify

멀티 스레드 환경에서는 여러 스레드가 동일한 리소스에 접근하여 변경하려고 시도할 수 있다. 이 때 보호되지 않은 리소스에 접근하게 되면 데이터의 일관성이 깨지게 된다.

여러 스레드의 작업을 조정하는 도구 중 하나는 Object 클래스의 wait, notify 메서드이다.

각각 스레드를 일시 중단(wait)하거나 중단된 스레드를 깨우는(notify) 역할을 한다.

하지만 이 wait, notify는 올바르게 사용하기 까다로우며 하드코딩 해야하는 일이 생기기 때문에 item81은 고수준 동시성 유틸리티 사용을 제안한다.

## java.util.concurrent

util.concurrent의 유틸리티는 세 범주로 나눌 수 있다.

실행자 프레임워크(ExecutorService), 동시성 컬렉션(concurrent collection), 동기화 장치(synchronizer)이다. item81은 동시성 컬렉션과 동기화 장치에 대해 다룬다.

### 동시성 컬렉션

동시성 컬렉션은 List, Queue, Map같은 표준 컬렉션 인터페이스에 동시성을 가미하여 구현한 컬력센이다. 높은 동시성에 도달하기 위해 동기화를 내부에서 수행한다. 

때문에 **동시성을 무력화하는 것은 불가능하며, 외부에서 lock을 추가로 사용하면 오히려 속도가 느려진다.**

추가적으로 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어서 호출하는 일도 역시 불가능하다. 때문에 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다. 

ConcurrentHashMap의 API를 살펴보자.

```java
// Correcto java 17.0.7 기준 ConcurrentHashMap API

public V put(K key, V value) {
    return putVal(key, value, false);
}    
    
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 첫 분기
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
            ...
```

첫 분기에는 빈 bucket에 node를 삽입하는 경우 casTab(Compare and swap)을 이용해 lock을 걸지 않고 새로운 node를 삽입한다.

```java
...

// 두 번째 분기
 else {
    V oldVal = null;
    synchronized (f) {
            if (tabAt(tab, i) == f) {
                if (fh >= 0) {
                    binCount = 1;
                    for (Node<K,V> e = f;; ++binCount
                    K ek;
                        if (e.hash == hash &&
                            ((ek = e.key) == key ||
                             (ek != null && key.equals(ek)))) {                                                             oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value);
                                    break;
                                }
                            }
                        }
                ...
```

두 번째 분기에서는 Bucket에 node가 이미존재하는 경우 synchronized를 이용해 다른 스레드가 접근하지 못하게 lock을 걸어 다른 스레드가 같은 hash bucket에 접근 할 수 없다.

이처럼 상태 의존적 수정 메서드를 사용해 thread-safe한 HashMap을 구현할 수 있다.

### 동기화 장치

동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율하도록 돕는다.

CountDownLatch, Semaphore, CyclicBarrier, Exchanger, Phaser.. 가 있다.

다음 코드는 CountDownLatch를 사용해 일정 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 코드이다.

```java
public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비가 됐음을 알림
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 대기
                    start.await();
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // 타이머에게 작업을 마쳤음을 알림
                    done.countDown();
                }
            });
        }

        ready.await(); // 모든 작업자가 준비될 때까지 대기
        long startNanos = System.nanoTime();
        start.countDown(); // 작업자들을 깨움
        done.await(); // 모든 작업자가 일을 끝마치기를 대기
        return System.nanoTime() - startNanos;
    }
}
```

코드는 카운트다운 래치를 3개를 사용하는데

- ready 래치는 작업자 스레드들이 준비가 완료되었음을 타이머 스레드에 통지하며, 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 대기
- 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start, countDown을 호출하여 기다리던 작업자 스레드를 깨움
- done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열리며, 타이머 스레드는 done 래치가 열리자마자 깨어나 종료 시각을 기록하게 된다. 
  
이 때 time 메서드에 넘겨진executor는 concurrency 매개변수로 지정한 값만큼의 스레드를 생성할 수 있어야 한다. 

그렇지 않다면 메서드 수행이 끝나지 않는데 이를 `스레드 기아 교착 상태(thread starvation deadlock)`라고 한다. 

추가적으로 시간을 잴 때는 시스템 시간과 무관한 System.nanoTime을 사용하는 것이 더 정밀하여 시간 보정에 영향받지 않아 권장된다. 

## wait, notify 사용 시 주의점

새로운 코드라면 동시성 유틸리티를 사용하면 되지만 레거시 코드를 다루는 경우에는 wait와 notify를 다뤄야할 때가 있다.

```java
synchronized (obj) {
    while (조건이 충족되지 않았다) {
        obj.wait(); // 락을 놓고, 깨어나면 다시 잡는다.
    }

    ... // 조건이 충족됐을 때의 동작을 수행
}
```

이 때 주의점으로 lock 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 여역 안에서 호출해야 한다.

반드시 대기 반복문(wait loop) 관용구를 사용하고 반복문 밖에서 호출하는 것을 금하라!

반복문은 wait 호출 전후로 조건이 만족하는지를 검사하는 역할을 한다. 

대기 전에 조건을 검사하여 조건이 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치이며, 만약 조건이 이미 충족되었지만 스레드가 notify 또는 notifyAll 메서드로 먼저 호출한 후 대기 상태로 빠지면, 그 스레드를 다시 깨우지 못할 수 가능성이 있다.

반면, 대기 후에 조건을 검사하여 조건을 충족하지 않았을 때 다시 대기하게 하는 것은 잘못된 값을 계산하는 안전 실패를 막기 위한 조치이다. 그런데 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황이 있다. 상황은 다음과 같다. 

- notify를 호출하여 대기 중인 스레드가 깨어나는 사이에 다른 스레드가 락을 거는 경우
- 조건이 만족되지 않았지만 실수 혹은 악의적으로 notify를 호출하는 경우
- 대기 중인 스레드 중 일부만 조건을 충족해도 notifyAll로 모든 스레드를 깨우는 경우
- 대기 중인 스레드가 드물게 notify 없이 깨어나는 경우. 허위 각성(spurious wakeup)이라고 한다.

일반적으로 notify보다는 notifyAll을 사용하는 것이 안전하며, wait는 항상 while문 내부에서 호출하도록 해야 한다.

## 참고자료

- 이펙티브자바 3판
- https://www.baeldung.com/java-wait-notify - baeldung wait-notify

