---
layout  : wiki
title   : ordinal 메서드 대신 인스턴스 필드를 사용하라 
summary : 
date    : 2023-11-13 09:33:55 +0900
updated : 2023-11-13 10:16:22 +0900
tag     : java effectviejava
resource: 52/87D4D6-017A-4261-A06A-08829851A416
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## ordinal() 메서드

```
public final int ordinal()

Returns the ordinal of this enumeration constant (its position in its enum declaration, where the initial constant is assigned an ordinal of zero). Most programmers will have no use for this method. It is designed for use by sophisticated enum-based data structures, such as EnumSet and EnumMap.

Returns:
the ordinal of this enumeration constant
```

enum은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal()를 제공한다. 그렇다면 열거 타입 상수와 연결된 정수값이 필요하여 ordinal() 메서드를 사용해보는 것은 어떨까?

```java
public enum Esemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1;
    }
}
```

위 코드는 합주단의 종류를 1명부터 10명까지 정의한 열거 타입이다.

ordinal() 메서드를 사용한 nubmerOfMusicians() 메서드는 합주단의 인원수를 반환한다. 코드는 동작하지만 이 코드에는 문제점이 있다.

먼저 상수의 선언 순서를 바꾸게되면 각 합주단의 종류가 반환하는 값이 달라져 numberOfMusicians는 제대로 동작하지 않으며, 8중주(OCTET) 상수가 이미 있어 똑같이 8명이서 연주하는 복4중주는 추가할 수 없다. 

또한 값을 중간에 비워둘 수 도 없어 12명이 연주하는 3중 4중주를 추가한다고 한다면 11명으로 구성된 합주단의 연주를 일컫는 이름이 없어 dummy 상수를 추가해야하는 실용성이 떨어지는 문제점을 발견할 수있다.

## 해결법

어찌보면 당연한 문제지만 위에 따른 문제점은 인스턴스 필드에 저장하여 해결할 수 있다. 

``java
public enum Esemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5), SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8), NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Esemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

인스턴스 필드에 정수로 합창단의 인원수를 명시하면 훨씬 보기쉽고 문제도 없다.

API 문서에 명시된 것처럼 ordinal() 메서드는 EnumSet과 EnumMap과 같이 열거 타입기반의 범용 자료구조에 쓸 목적으로 설계된 메서드로 해당 목적이 아니라면 인스턴스 필드를 선언하여 사용하자.

## 마무리 

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html#ordinal-- - enum docs
