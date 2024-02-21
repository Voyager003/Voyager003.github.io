---
layout  : wiki
title   : 예외는 진짜 예외 상황에만 사용하라 
summary : 
date    : 2024-02-21 09:37:26 +0900
updated : 2024-02-21 13:37:04 +0900
tag     : java effectivejava
resource: F5/4406D1-5E9F-48B9-B02F-DA1BE5F92262
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java의 예외 처리의 잘못된 예시

```java
try {
    int i = 0;
    while(true) {
        range[i++].climb();
    } 
} catch (ArrayINdexOutOfBoundsException e) {
}
```

이 코드는 배열 원소를 순회하면서 무한루프를 돌다가 배열의 끝 인덱스에 도달하여 ArrayINdexOutOfBoundsException이 발생하면 예외를 발생시켜 종료되는 프로그램이다.

JVM이 배열에 접근할 때마다 경계를 넘지 않는지 검사하는데, 일반적인 반복문에서도 배열 경계에 도달하게되면 종료한다. 따라서 반복문에도 예외를 명시하면 같은 일이 중복되므로 하나를 생략한 것인데, 이는 다음 이유에서 잘못된 추론이다.

- 예외는 예외 상황에서만 사용할 용도로 설계되었으므로 JVM 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야할 동기가 약하다.

- 코드를 try-catch 블록 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다.

- 배열을 순회하는 표준 관용구(for (Mountain m : range) ..)는 앞서 걱정한 중복 검사를 수행하지 않고 JVM이 최적화하여 없앤다.


## 예외 처리의 목적

논점은 예외는 오직 예외 상황에서만 사용하고, 일상적인 제어 흐름용으로 사용해선 안된다는 것.

이는 API 설계에도 적용되며, 잘 설계된 API라면 클라이언트가 정상적인 제어 흐름에서 예외를 사용할 일이 없도록 해야 한다.

특정 상태에서만 호출할 수 있는 '상태 의존적' 메서드를 제공하는 클래스는 '상태 검사' 메서드도 함께 재공해야 한다. Iterator 인터페이스의 next와 hasNext가 각각 상태 의존적 메서드, 상태 검사 메서드에 해당한다.

```java
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
    Foo foo = i.next();
    ...
}
```

이 코드는 상태 검사 메서드를 사용하여 예외를 처리하는 방법이다.

Iterator의 hasNext() 메서드를 사용하여 컬렉션에서 다음 요소가 있는지 먼저 확인한 후에 next() 메서드를 호출하는데, 이는 예외 처리보다는 명시적인 상태 검사를 통해 예외를 방지하려고 시도한다.

상태 검사 메서드를 사용하면 예외가 발생하는 상황이 아니라면 예외 처리에 대한 오해를 줄이면서 좀 더 직관적이고 예외 처리 시의 오버헤드가 없어 성능이 더 빠를 수 있다.

반면 이를 클라이언트에서 처리한다면 다음과 같다.

```java
try{
  Iterator<Foo> i = collection.iterator();
  while(true) {
    Foo foo = i.next();
    ...
  }
} catch(NoSuchElementException e) {
}
```

try-catch 블록을 사용하여 NoSuchElementException 예외를 직접적으로 처리하는데, 루프는 예외가 발생할 때까지 계속 실행되며, 예외가 발생하면 catch 블록이 실행되어 예외를 처리하게 된다.

이 방법은 반복문에 예외를 사용하여 속도가 느리며, 발생한 버그를 숨기기도 한다.

### 다른 선택지

올바르지 않은 상태일 때 빈 Optional 혹은 null 같은 특수한 값을 반환하는 방법도 있다.

item69에서는 상태 검사 메서드, Optional, 특정 값 중 하나를 선택하는 지침을 제시한다.

- 외부 동기화 없이 여러 스레드가 동시에 접근할 수 있거나 외부 요인으로 상태가 변할 수 있는 경우

상태 검사 메서드와 상태 의존적 메서드 호출 사이의 객체 상태가 변할 수 있어 Optional이나 특정 값을 사용한다.

```java
public class SharedData {
    private volatile boolean dataReady = false; // 상태를 나타내는 플래그
    
    // 상태 검사 메서드
    public boolean isDataReady() {
        return dataReady;
    }
    
    public void setDataReady(boolean ready) {
        dataReady = ready;
    }
}

public class Main {
    public static void main(String[] args) {
        SharedData sharedData = new SharedData();
        
        // 여러 스레드가 동시에 데이터의 상태를 확인
        boolean ready = sharedData.isDataReady();
        System.out.println("Data is ready: " + ready); // 예측하기 어려움
    }
}
```

상태 검사 메서드를 여러 스레드가 동시에 호출하면 데이터의 상태가 변경될 수 있어 일관성 보장이 어려울 수 있다.

- 성능이 중요한 상황에서 상태 검사 메서드가 상태 의존적 메서드의 작업 일부를 중복 수행하는 경우

Optional, 특정 값을 사용한다.

- 그 외

다른 모든 경우에는 다음 이유에서 상태 검사 메서드 방식이 좀 더 나은 선택이다.

먼저 가독성이 좋으며, 잘못 사용했을 때 발견하기 쉽다. 상태 검사 메서드 호출을 잊었다면 상태 의존적 메서드가 예외를 던져 버그를 드러낼 수 있다. 이는 특정 값을 사용하면 검사하지 않고 지나쳐도 발견하기 어렵다. 

## 참고자료

- 이펙티브자바 3판

