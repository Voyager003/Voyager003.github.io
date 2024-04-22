---
layout  : wiki
title   : 스레드 안전성 수준을 문서화하라 
summary : 
date    : 2024-04-22 10:45:01 +0900
updated : 2024-04-22 11:09:39 +0900
tag     : java effectivejava
resource: 96/6DE3E1-CFBC-4EBD-9C2F-1A86B2483C01
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 스레드 안전성 수준 문서화

한 메서드를 여러 스레드가 동시에 호출할 때, 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약이다. 

API 문서에 synchronized 한정자가 명시된 메서드는 thread-safe해보이지만 다음 몇가지 면에서 틀렸다.

- javadocs이 기본옵션에서 생성한 API 문서에는 synchronized 한정자가 포함되지 않는다.

메서드 선언에 synchronized 한저앚를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다. 따라서 이것만으로 thread-safe하다고 믿기 어렵다.

- 스레드 안전성의 수준을 고려

멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다. 다음 목록은 스레드 안전성이 높은 순으로 나열한 것이다.

    - 불변(immutable): 불변 클래스의 인스턴스는 상수와 같아 외부 동기화가 필요없다. (String, Long, BigInteger 등..)
    - 무조건적 스레드 안전(unconditionally thread-safe): 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다. (AtomicLong, ConcurrentHashMap 등..)
    - 조건부 스레드 안전(conditionally thread-safe): 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다. Collections.synchronized 래퍼 메서드가 반환한 컬렉션들이 이에 속한다.
    - 스레드 안전하지 않음(not thread-safe): 이 클래스의 인스턴스는 수정될 수 있다. 또한 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다. (ArrayList, HashMap 등..)
    - 스레드 적대적(thread-hostile): 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다. 일반적으로 정적 데이터를 아무 동기화없이 수정한다. 스레드 적대적으로 밝혀진 클래스나 메서드는 일반적으로 문제를 고쳐 재배포하거나 deprecated API로 지정한다.

## 문서화 주의점

클래스의 스레드 안전성은 클래스의 문서화 주석에 기재한다. Collections.synchronizedMap이 좋은 예이다.

```java
// Collections.synchronizedMap API

synchronizedMap이 반환한 map의 컬렉션 뷰를 순회하려면 그맵을 lock으로 사용해 수동으로 동기화 하라
```

독특한 특성의 메서드라면 해당 메서드의 주석에 기재한다.

```java
Map<K, V> m = Collections.synchronizedMap(new HashMap<>());
Set<K> s = m.keySet(); // 동기화 블록 밖에 있어도 된다.

...

synchronized(m) { // s가 아닌 m을 사용해 동기화 해야함
    for (K key : s) 
        key.f();
}

이대로 따르지 않으면 동작을 예측할 수 없다.
```

### 비공개 lock 객체

클래스 외부에서 사용할 수 있는 lock을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다. 하지만 이는 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없게 된다. 때문에 ConcurrentHashMap 같은 동시성 컬렉션과는 함께 사용하지 못한다.

또한 클라이언트가 공개된 lock을 오래 쥐고 놓지 않는 서비스 거부 공격(denial-of-service attack)을 수행할 수 있다. 이를 막으려면 synchronized 메서드 대신 비공개 lock 객체를 사용해야 한다. 

```java
private final Object lock = new Object();

public void foo() {
    synchronized(lock) {
        ...
    }
}
```

비공개 lock 객체는 클래스 바깥에선 보리수 없어 클라이언트가 그 객체의 동기화에 관여할 수 없다. 

이 비공개 lock 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용할 수 있다. 조건부 스레드 안전 클래스에서는 특정 호출 순서에 필요한 lock이 무엇인지를 클라이언트에게 알려줘야 하므로 관용구를 사용할 수 없다. 

## 참고자료

- 이펙티브자바 3판

