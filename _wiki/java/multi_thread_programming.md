---
layout  : wiki
title   : Java Multi-Thread Programming
summary : 
date    : 2023-05-12 19:42:30 +0900
updated : 2023-05-13 20:38:46 +0900
tag     : java
resource: A5/A38377-6BF2-4200-9D35-6F712C45A65B
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/10) 10주차 과제

## Thread 클래스와 Runnable 인터페이스

![](https://i.stack.imgur.com/vLRdp.gif)

Thread를 생성하는 방법은 크게 두 가지가 있다.

### Thread class

```java
package java.lang;

class Thread implements Runnable {
    private static native void registerNatives();
    static {
        registerNatives();
    }
        ...
}
```

- Thread 클래스는 Runnable 인터페이스를 구현한 클래스이다.
- java.lang 패키지에 포함되어있다.

### Runnable interface

```java
package java.lang;

@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

- Runnable은 단일 추상 메서드인 run()을 가진다.
- Thread가 실행할 코드를 정의한다.
- Runnable은 인터페이스로 다른 클래스를 상속할 수 있어 재사용에 용이하다.

### Thread 실행

```java
// Runnable
public class RunnableSample implements Runnable{
    @Override
    public void run() {
        System.out.println("This is RunnableSample's run() method.");
    }
}

// Thread
public class ThreadSample extends Thread{
    @Override
    public void run() {
        System.out.println("This is ThreadSample's run() method.");
    }
}

// Main 함수
public class RunThreads {
    public static void main(String[] args) {
        runBasic();
    }

    public static void runBasic() {
        RunnableSample runnable = new RunnableSample();
        new Thread(runnable).start();
        ThreadSample thread = new ThreadSample();
        thread.start();
        System.out.println("RunThreads.runBasic() method is ended.");
    }
}

// 결과
This is RunnableSample's run() method. 
RunThreads.runBasic() method is ended.
This is ThreadSample's run() method.
```

- 먼저 Runnable 인터페이스로 선언한 start()가 실행된 뒤 끝날때 까지 기다리지 않고, Thread 클래스의 start()를 실행한다.
- 새로운 Thread를 시작하여 run()가 종료될 때까지 기다리지 않고, 다음 코드를 실행한다.

### sleep()

```java
public class Main {
  public static void main(String[] args) {
    try {
      System.out.println("Sleeping...");
      Thread.sleep(1000);
      System.out.println("Awake!");
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

- Thread 클래스의 sleep()은 현재 실행 중인 Thread를 지정된 시간(millie seconds)동안 일시 중지한다.
- InterruptedException 예외를 던질 수 있다고 선언되어 있어 try-catch 블록으로 묶어서 사용한다.

## Thread states

<img width="640" alt="스크린샷 2023-05-12 오후 10 04 28" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/67b5ff59-fe3b-439d-8c9b-41f8f50a7f1e">

- NEW : Thread 인스턴스가 생성되었지만, 아직 시작되지 않은 상태
- RUNNABLE : Thread가 실행 중인 상태
- BLOACKED : Thread가 실행 중지 상태이며, monitor lock이 풀리기 기다리는 상태
- WAITING : Thread가 대기 중인 상태
- TIMED_WAITING : 특정 시간만큼 Thread가 대기중인 상태
- TERMINATED : Thread가 종료된 상태

### 관련 메서드

- void checkAccess() : 현재 Thread가 SecurityManager에 의해 Thread를 수정할 수 있는 권한이 있는지 확인한다.
  - 권한이 없다면 SecurityException 예외를 발생
- boolean isAlive() : Thread가 실행 중인지 확인한다.
  - 해당 Thread의 run()의 종료 여부를 확인한다.
- boolean isInterrupted() : run()이 정상적으로 종료되지 않고, interrupt() 호출을 통해 종료되었는지 확인한다.
- static booelan interrupted() : Thread 중단 여부를 확인한다. Thread의 중단된 상태는 메서드에 영향받지 않는다.
- static int activeCount() : 현재 Thread가 속한 Thread 그룹의 활성화 된 Thread 개수를 반환한다.
- static Thread currentThread() : 현재 실행 중인 Thread 인스턴스를 반환한다.
- static void dumpStack() : console에 현재 Thread stack 정보를 출력한다.

### Object 클래스의 메서드

- void wait() : 다른 Thread가 Object 객체에 대한 notify(), notifyAll()를 호출할 때 까지 현재 Thread가 대기하고 있도록 한다.
  - wait() 호출 시, 상태는 WAITING이 되며, notify() 메서드를 호출하여 Thread를 깨워 WAITNIG 상태를 풀 수 있다.
- void wait(long timeout) : 매개 변수에 지정한 시간만큼 대기한다. 대기한 뒤, 매개 변수 시간을 넘어가면 현재 Thread가 다시 깨어난다.
- void wait(long timeout, int nanos) : 보다 상세한 대기시간 기록을 위한 millie/s+nano/s 
- void notify() : Object 객체에 대한 대기 중인 Thread 중 하나를 깨운다.
- void notifyAll() : Object 객체에 대한 대기 중인 모든 Thread를 깨운다.

## Thread 우선순위(priority)

- Thread.MIN_PRIORITY : Thread의 최소 우선순위
- Thread.NORM_PRIORITY : Thread의 기본 우선순위
- Thread.MAX_PRIORITY : Thread의 최대 우선순위

각 Thread는 우선순위에 관한 자신만의 필드를 가진다. 

- getPrioity(), setPriority()를 통해 Thread 우선순위를 반환 혹은 변경할 수 있다. 
- Thread 우선 순위가 가질 수 있는 범위는 1부터 10까지로, 높을 수록 우선순위가 높다.
- 이 때, 비례적인 절댓값이 아닌, 상대적인 값이다.

### 예시

```java
// ThreadWithRunnable.java
class ThreadWithRunnable implements Runnable {

  public void run() {
    for (int i = 0; i < 5; i++) {
      System.out.println(Thread.currentThread().getName()); // 현재 실행 중인 Thread의 이름 반환
      try {
        Thread.sleep(10);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
}

// Thread02.java
public class Thread02 {

  public static void main(String[] args){
    Thread thread1 = new Thread(new ThreadWithRunnable());
    Thread thread2 = new Thread(new ThreadWithRunnable());
    thread2.setPriority(10);
    thread1.start(); // Thread1 실행
    thread2.start(); // Thread2 실행
    System.out.println(thread1.getPriority());
    System.out.println(thread2.getPriority());
  }
}

// 결과
5
10
Thread-1
Thread-0
Thread-1
Thread-0
Thread-1
Thread-0
Thread-1
Thread-0
Thread-1
Thread-0
```

- main() 메서드를 실행하는 Thread의 우선순위는 5이다.
- setPriority()를 사용해 thread2의 우선순위를 10으로 지정했다.
- thread2가 우선순위가 상대적으로 더 높기때문에, 실행 queue에 포함되어 좀 더 많은 작업 시간을 할당받았다. 

## Main Thread

```java
public class MainMethod {
  public static void main(String[] args) {

  }
}
```

- Java 코드를 실행하기 위한 메서드로 Main Thread이다.
- public static void main(String[] args)은 Main Thread로 Main Thread의 시작점을 선언하는 것이다.
- 따로 Thread를 실행하지 않고 main()만 실행하는 것을 Single-Thread Application이라 한다.
- Main Thread에서 Thread를 생성하여 실행하는 것은 Multi-Thread Application이라 한다.

### Daemon Thread

- Background에서 실행되는 Thread로 Main Thread가 종료되면, Daemon Thread는 강제로 종료된다.
  - Background : 프로그램이나 시스템에서 사용자의 주요 작업과는 별도로 동작하거나 실행되는 영역으로, 사용자가 보고있는 화면에는 나타나지 않고 컴퓨터 내부에서 작업이 실행되고 있는 상태이다.
- Main Thread를 보조하는 Thread로, Background에서 지속적으로 실행되어야 하는 작업에 사용된다.

```java
public class DaemonThreadExample {
    public static void main(String[] args) {
        Thread daemonThread = new Thread(new MyRunnable());
        daemonThread.setDaemon(true); // Daemon 스레드로 설정
        daemonThread.start();

        // 주 스레드 작업 수행
        System.out.println("Main thread is running");

        try {
            Thread.sleep(3000); // 주 스레드를 3초 동안 대기시킴
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 주 스레드 종료
        System.out.println("Main thread is exiting");
    }

    static class MyRunnable implements Runnable {
        @Override
        public void run() {
            while (true) {
                System.out.println("Daemon thread is running");
                try {
                    Thread.sleep(1000); // 1초마다 작업 실행
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

- DaemonThread.setDaemon(true)를 사용하여 daemonThread를 Daemon Thread로 설정했다.
- daemonThread.start()를 호출하여 스레드를 실행시킨 뒤, main thread에서는 "Main thread is running"을 출력한 후 3초 동안 대기한 뒤 "Main thread is exiting"을 출력하여 종료된다.
- Daemon Thread인 MyRunnable은 background에서 1초마다 "Daemon thread is running"을 출력하는 작업을 수행하고, main thread가 종료되면 함께 종료되는 것을 확인할 수 있다.
- 이 때, JVM이 모든 Non-Daemon Thread가 종료되면 프로그램을 종료시키기 때문에, 자원 정리등의 작업을 수행하지 않는 것이 좋다.
- 

## 동기화(Synchronize)

- 여러 Thread가 하나의 resource에 접근할 때, 사용하려는 Thread를 제외한 나머지 Thread에서 접근하지 못하게 막는 것
- 이를 만족하면 Thread-safe하다고 한다.
- Java의 동기화는 크게 3가지로 분류된다.

### Synchronized 키워드

- 예약어로, 변수명이나 클래스 명으로 사용이 불가능하다.
- 메서드 자체를 synchronized로 선언하는 법과 메서드 내 특정 부분을 synchronized로 선언하는 법(synchronized statements)이 있다.

```java
public class Counter {
  private int count = 0;

  public synchronized void increment() {
    count++;
  }

  public synchronized int getCount() {
    return count;
  }
}

public class Main {
  public static void main(String[] args) throws InterruptedException {
    Counter counter = new Counter();

    Thread t1 = new Thread(() -> {
      for (int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });

    Thread t2 = new Thread(() -> {
      for (int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });

    t1.start();
    t2.start();

    t1.join();
    t2.join();

    System.out.println("Count: " + counter.getCount());
  }
}
```

- Counter 클래스는 공유 변수 count를 가지고 있으며, increment()는 count의 값을 1증가시키는 작업을 수행한다.
- synchronized 키워드로 인해 한 번에 하나의 thread만 increment() 메서드에 진입하여 count를 증가시킬 수 있다.
- 이로써 여러 thread가 동시에 count 값을 수정하는 충돌이 발생하지 않는다.
- Main 클래스에서는 두 thread(t1, t2)를 생성하여 동시에 Counter 인스턴스의 increment()를 호출한다.
- 각 thread는 1000번씩 increment()를 호출하므로, count의 값은 2000이 되어야 한다.

```java
// 적용하지 않은 경우
public class Counter {
  private int count = 0;

  public void increment() {
    count++;
  }

  public int getCount() {
    return count;
  }
}

public class Main {
  public static void main(String[] args) throws InterruptedException {
    Counter counter = new Counter();

    Thread t1 = new Thread(() -> {
      for (int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });

    Thread t2 = new Thread(() -> {
      for (int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });

    t1.start();
    t2.start();

    t1.join();
    t2.join();

    System.out.println("Count: " + counter.getCount());
  }
}
```

- synchronized 키워드가 없는 경우이다.
- 이 때, 여러 thread가 동시에 increment()에 접근할 수 있다.
- 따라서 thread 간 경합조건(race condition)이 발생할 수 있고, count의 값이 유지되지 않을 수 있다.

### Atomic Class

- java.util.concurrent.atomic package의 atomic 클래스는 Atomic type은 Wrapping 클래스의 일종이다. 
- primitive, reference type 두 종류의 변수에 모두 적용 가능하다.
- CAS(Compare-And-Sweep) 알고리즘을 사용해 lock 없이 동기화 처리가 가능하다.
  - CAS는 메모리 위치 내용을 주어진 값과 비교하고, 동일한 경우에만 해당 메모리 위치의 내용을 새로 주어진 값으로 수정하는 알고리즘이다.

#### 주요 클래스 및 메서드

- 클래스 : AtomicBoolean, Integer, Long, IntegerArray, DoubleArray가 있다.
- 메서드 
  - get() : 현재 값을 반환한다.
  - set(newValue) : 현재 값을 설정한다.
  - CompareAndSet(expect, update) : 현재 값이 expect와 같으면 update로 설정하고 true를 반환한다. 다르면 false를 반환한다.
  - addAndGet(delta) : 현재 값에 delta를 더한 값을 반환한다.
  - getAndAdd(delta) : 현재 값을 반환하고, delta를 더한다.

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {
  private AtomicInteger count = new AtomicInteger(0);

  public void increment() {
    count.incrementAndGet();
  }

  public int getCount() {
    return count.get();
  }
}

public class Main {
  public static void main(String[] args) throws InterruptedException {
    AtomicCounter counter = new AtomicCounter();

    Thread t1 = new Thread(() -> {
      for (int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });

    Thread t2 = new Thread(() -> {
      for (int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });

    t1.start();
    t2.start();

    t1.join();
    t2.join();

    System.out.println("Count: " + counter.getCount());
  }
}
```

- AtomicCounter 클래스는 AtomicInteger를 사용하여 count 값을 증가시키는 기능을 제공한다.
- AtomicInteger 인스턴스의 초기값을 0으로 생성하고 increment()는 메서드를 사용해 카운터 값을 1씩 증가시킨다.
- 이 때, thread-safe한 atomic(원자적)연산을 수행하여 여러 thread가 동시에 접근해도 안전하게 값을 증가시킬 수 있다.
- 각 thread는 1000번 씩, increment()를 호출하므로, 2000이라는 값이 되어야한다.

### Volatile 키워드

- volatile 키워드는 변수를 메인 메모리에 저장하도록 지시하는 키워드이다.
- 변수를 CPU 캐시가 아닌 메인 메모리에 직접 읽고 쓸 수 있다.
- 만약 Multi Thread 환경에서 Thread가 변수 값을 읽어올 때 각 CPU 캐시에 저장된 값이 다르기 때문에 변수 값 불일치 문제가 발생하는데 이를 해결하기 위함이다.
- 또한 Multi Thread 환경에서 변수의 가시성과 일관성을 보장한다.
  - 가시성 : volatile 변수는 변수의 변경 내용이 다른 thread에 즉시 알려야하는 것을 보장

```java
public class SharedData {
    private volatile int counter = 0;

    public void increment() {
        counter++;
    }

    public int getCounter() {
        return counter;
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        SharedData sharedData = new SharedData();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                sharedData.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                sharedData.increment();
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("Counter: " + sharedData.getCounter());
    }
}
```

- SharedData 객체의 increment()를 호출하면, 각 thread는 1000번 씩 호출하여 counter 값은 총 2000이 되어야 한다.
- counter 변수가 volatile로 선언되었기 때문에, 한 thread에서 counter 값을 변경하면 다른 thread에서는 변경된 값을 즉시 읽을 수 있다. (가시성)
- CPU 캐시보다 Main Memory 비용이 더 크기 때문에, 변수 값 일치를 보장해야 하는 경우에 volatile을 고려하는 것이 좋다.

## DeadLock(데드락, 교착상태)

- Multi Thread 환경에서 발생하는 동기화 문제로, 두 개 이상의 thread가 서로 소유한 자원을 기다리며 무한히 대기하는 상황이다.

```java
public class DeadlockExample {
    private static Object resource1 = new Object();
    private static Object resource2 = new Object();

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            synchronized (resource1) {
                System.out.println("Thread 1: Locked resource 1");

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (resource2) {
                    System.out.println("Thread 1: Locked resource 2");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (resource2) {
                System.out.println("Thread 2: Locked resource 2");

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (resource1) {
                    System.out.println("Thread 2: Locked resource 1");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

- t1 thread는 resouce1을 점유한 상태에서 resource2를 기다리고, t2는 그와 반대인 상태이다.
  - 여기서 thread가 resouce를 점유한다는 것은 해당 reousrce에 대한 access 권한을 갖고, 다른 thread들은 접근할 수 없는 상태를 의미한다.
- 두 thread는 서로 점유한 resource를 기다리고 있는 상태로 deadlock 상태가 되며, 아무런 작업도 수행하지 못한다.
- deadlock을 해결하기 위한 접근 방법을 몇 가지 확인해보자.
  - resource를 동시에 모두 요청하도록 변경한다. 
  - 점유 resource를 기다리는 대신, 점유하지 않은 resource를 찾아 작업을 수행하도록 한다.(tryLock())
  - resource의 우선순위를 정의하고, 순환 대기 조건을 방지한다.

## 참고자료
- https://sujl95.tistory.com/63 
- https://stackoverflow.com/questions/541487/implements-runnable-vs-extends-thread-in-java
