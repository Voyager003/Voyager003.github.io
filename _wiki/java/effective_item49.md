---
layout  : wiki
title   : 매개변수가 유효한지 검사하라 
summary : 
date    : 2024-01-07 20:28:56 +0900
updated : 2024-01-08 11:22:21 +0900
tag     : java effectivejava
resource: BA/A6E86D-FD58-421F-AB90-2ACF654064CB
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 매개변수 유효성 검증을 하지 않는 경우의 문제점

메서드와 생성자 대부분 입력 매개변수의 값이 특정 조건을 만족하기를 바란다. 예를 들어 인덱스의 값은 음수이면 안되며, 객체 참조는 null이 아니어야한다는 것이다.

이러한 제약은 반드시 문서화가 필요하며 메서드의 몸체가 시작되기 전에 검사해야하며, 그렇지 않으면 몇 가지 문제가 발생한다.

- 메서드가 수행되는 중간에 모호한 예외를 던지며 실패
- 메서드가 제대로 수행됐지만 잘못된 결과를 반환
- 메서드가 제대로 수행됐지만, 어떤 객체의 상태를 이상하게 만들어 알 수 없는 시점에 메서드와 관련없는 오류가 발생

### 예외를 문서화

public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야한다.

이어서 제약을 문서화한다면 그 제약을 어겼을 때 발생하는 예외도 함께 기술해야 한다.

```java
    /**
     * (현재 값 mod m) 값을 반환
     * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder와 다름
     *
     * @param m 계수(양수)
     * @return 현재 값 mod m
     * @throws ArithmeticException m이 0 이하이면 발생
     */
    public BigInteger mod(BigInteger m) {
        if (m.signum() <= 0) {
            throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
        }
        return m;
    }
```

mod() 메서드는 m이 null인 경우, signum() 호출 시, nullPointerException을 던진다.

하지만 이러한 설명이 메서드 설명 어디에도 없다. 이유는, 설명을 개별 메서드가 아닌 BigInteger 클래스 수준에서 기술했기 때문이다. 

클래스 수준의 주석은 그 클래스의 모든 public 메서드에 적용되므로, 각 메서드에 일일이 적는 것 보다 깔끔한 방법이다. 

### java.util.Objects.requireNonNull

```java
// Objects.requireNonNull

public static <T> T requireNonNull(T obj) {
    if (obj == null) {
            throw new NullPointerException();
        return obj;
    }
}

this.startegy = Objects.requireNonNull(startegy, "전략");
```

Java 7에 추가된 메서드로 null 검사를 수동으로 하지않면서 원하는 예외 메시지도 지정하며, 입력을 그대로 반환하여 값을 사용하는 동시에 null 검사를 수행 가능하다.

### Objects 범위 검사

```java
// Objects.checkFromToIndex

public static int checkFromIndexSize
    (int fromIndex, int size, int length)
    
Checks if the sub-range from fromIndex (inclusive) to fromIndex + size (exclusive) is within the bounds of range from 0 (inclusive) to length (exclusive).
The sub-range is defined to be out-of-bounds if any of the following inequalities is true:    
```

checkFromIndexSize, checkFromToIndex, checkIndex 메서드는 Java 9에 추가된 기능으로 범위 검사를 수행한다. 그 중 하나인 checkFromIndexSize의 API를 살펴보면

fromIndex는 0 이상, size는 0 이상, fromIndex + size는 length를 초과해서는 안 된다. 이때 정수 오버플로우를 고려해야 한다.

length는 0 이상이어야 합니다. 이는 앞선 불평등에서 암시됩니다.
반환 값과 예외:

이 메서드는 부분 범위가 전체 범위 내에 있으면 fromIndex를 반환
부분 범위가 범위를 벗어나면 IndexOutOfBoundsException 예외를 던짐

등의 설명을 볼 수 있다.

하지만 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계되어 있다는 점을 고려해야 한다.

### 공개되지 않은 메서드 

공개되지 않은 메서드라면 패키지 제작자인 개발자 본인이 메서드 호출 상황을 통제할 수 있다. 

따라서 유효한 값만이 메서드에 넘겨지리라는 것을 보증 할 수 있꼬, 그렇게 해야만 한다.

```java
    private static void sort(long a[], int offset, int length) {
        assert a != null;
        assert offset >= 0 && offset <= a.length;
        assert length >= 0 && length <= a.length - offset;
        // 계산 수행
    }
```

단언문(assert)를 사용해 매개변수 유효성을 검증한 것이다.

핵심은 이 **단언문이 자신이 단언한 조건이 무조건 참이라고 선언한다는 점**이다.

단언문은 몇 가지 면에서 일반적인 유효성 검사와 다른데, 이는 다음과 같다.

- 실패하면 AssertinoError를 던진다.
- 런타임에 아무런 효과도, 아무런 성능 저하도 없다.

## 정리

item49의 내용을 '매개변수에 제약을 두는 게 좋다'고 해석해선 안된다. 

메서드를 최대한 범용적으로 설계하고, 메서드가 건네받은 값으로 뭔가 제대로 된 일을 할 수 있다면 매개변수 제약은 적을수록 좋다. but 구현하려는 개념 자체가 특정 제약을 내재한 경우도 드물지 않다.

항상 제약들을 문서화하고, 메서드 코드 시작 부분에서 명시적으로 검사하는 습관을 들이도록 하자!

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/util/Objects.html - java docs
 
