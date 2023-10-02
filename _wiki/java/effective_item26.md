---
layout  : wiki
title   : 로 타입(Raw type)은 사용하지 말라 
summary : 
date    : 2023-10-01 10:29:07 +0900
updated : 2023-10-02 12:11:07 +0900
tag     : java effectivejava
resource: 9A/0B9F2E-7744-45E5-BA01-1267375756D4
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 로 타입(Raw Type)

```java
// 제네릭 타입
List<E>

// 로 타입
List
```

클래스 및 인터페이스 선언 타입에 매개변수가 쓰이면 제네릭 클래스 혹은 제네릭 인터페이스라한다. 원소의 타입을 나타내는 타입 매개변수 E를 받아 List<E>로 나타내며 이를 통틀어 제네릭 타입이라 한다.

로 타입은 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.

로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작하는데 제네릭이 등장하기 전(JDK 1.5)에 이전 코드와 호환되도록 하기 위한 방법이라고 할 수 있다.

이 로 타입의 문제점은 다음과 같다.

```java
List numbers = new ArrayList();
numbers.add("one"); // 컴파일 에러 발생 X

String number = (String) numbers.get(0); // ClassCastException 발생
```

로 타입은 컴파일 타임에 타입 정보를 알지 못한다.

컴파일 시에는 문제가 없던 코드가 런타임에 타입 캐스팅을 시도했기 때문에 ClassCastException 예외가 발생하게 된다. 

반면 제네릭 타입의 경우

```java
List<String> numbers = new ArrayList<>();
numbers.add("one");

numbers.add(1); // 컴파일에러 발생
```

컴파일 타임에 numbers에는 String 타입의 인스턴스만 넣어야 한다는 것을 인지하고 있어, 아무런 경고 없이 컴파일되면 코드가 의도대로 동작할 것임을 보장한다.

## 로 타입의 존재 의의

로 타입을 사용하면 제네릭이 안겨주는 안전성(타입)과 표현력 모두 잃게 된다. 그렇다면 절대 써서 안되는 로 타입을 왜 만들어놓은걸까?

그 이유는 마이그레이션 호환성으로, 제네릭이 등장하기 전에 짠 코드가 이미 사용되고 있어 기존 코드를 모두 수용하면서 제네릭을 사용하는 새 코드와 맞물려 돌아가게 해야만 했다. 이 호환성을 위해 로 타입을 지원하고 제네릭 구현에는 소거(item28)를 사용하기로 했다.

### List<Object>

로 타입은 사용해선 안되지만, List<Object>와 같이 임의 객체를 허용하는 매게변수화 타입은 괜찮다. 

그 이유는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달했기 때문이다.

이처럼 로 타입은 타입 안정성을 잃게 된다.

## 비한정적 와일드카드(Unbounded wildcard type)

제네릭 타입을 쓰고싶지만, 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 비한정적 와일드카드를 사용하자.

```java
// 로 타입
static int numElementsInCommon(Set s1, Set s2) {
    ...
}

// 비한정적 와일드카드
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
    ...
}
```

로 타입 컬렉션 Set은 아무 원소나 허용할 수 있어 타입 불변식을 훼손하기 쉬운 반면, 와일드카드로 선언된 Collection(Set)<?>의 경우 null외에 어떤 원소도 넣을 수 없다. 이는 컴파일 시점에 오류를 발생시켜 컬렉션의 타입 불변식을 훼손하지 못하게 막고 컬렉션에서 꺼낼 수 있는 객체의 타입도 알 수 없게 한다.

## 로 타입 규칙 예외

### class 리터럴의 경우

Java 명세는 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다.

List, class, String[].class, int.class는 허용하지만, List<String>.class와 List<?>.class는 허용하지 않는다.

### instanceof 연산자의 경우

Java의 제네릭은 타입 소거([type erasure](https://voyager003.github.io/wiki/java/java_generic/#type-erasure)라는 메커니즘을 사용한다. 컴파일러가 타입 패러미터를 사용하는 대상의 타입을 컴파일러가 정하는 타입으로 대체하게 된다.

따라서 런타임에는 제네릭 타입의 정보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입외의 매개변수화 타입에는 적용할 수 없다. 

로 타입이든 비한정적 와일드카드 타입이든 instanceof는 똑같이 동작한다. 비한정적 와일드카드의 <?> 표현은 역할 없이 코드만 지저분하게 만드므로, 다음과 같이 로 타입을 쓰는 편이 깔끔하다.

```java
if (o instanceof Set) { // 로 타입
    Set<?> s = (Set<?>) o; // 와일드카드 타입
    ...
}
```

## 참고자료

- 이펙티브자바 3판

