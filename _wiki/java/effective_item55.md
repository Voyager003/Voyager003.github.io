---
layout  : wiki
title   : 옵셔널 반환은 신중히 하라 
summary : 
date    : 2024-01-24 09:34:56 +0900
updated : 2024-01-24 11:28:38 +0900
tag     : java effectivejava
resource: EB/8E121F-8C90-417C-ABE3-51491F23F82E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java에서 값을 반환할 수 없는 경우

### 예외 던지기

```java
public static <E extends Comparable<E>> E max(Collection<E> c){
    if (c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");

    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0){
            result = Objects.requireNonNull(e);
        }
    }
    return result;
}
```

위 코드는 컬렉션의 최댓값을 반환하는 메서드로 컬렉션이 비었다면 예외를 던진다.

예외를 던지는 방법은 진짜 예외적인 상황에서만 사용해야 하며, 예외 생성 시 StackTrace 전체를 캡처하므로 비용도 만만치 않다.

### null 반환

null을 반환하는 메서드를 호출할 때는 별도의 null 처리 코드가 필요하다. 

이를 무시하고 반환된 null 값을 어딘가에 저장한다면 NullPointerException이 발생할 수 있다.

### Optional 반환

Java 8에 도입된 Optional<T>는 원소를 최대 1개 가질수 있는 '불변 컬렉션'으로 null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.

보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야하는 경우, T 대신 Optional<T>를 반환하도록 선언하면 된다. 이는 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드를 만든다. 

최댓값이 비었으면 예외를 던지는 코드를 Optional을 사용하도록 수정해보자.

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty()) {
        return Optional.empty(); // 빈 Optional
    }
    
    E result = null;
    for (E e : c) {
        if (result == null || e.compareTo(result) > 0){
        result = Objects.requireNonNull(e);
        }
    }
    return Optional.of(result); // 비지 않은 Optional
}
```

유의할 점은 값이 비지않은 Optional.of(value)에 null을 넣으면 NullPointerException을 던진다는 것이다. Optional을 반환하는 메서드에서는 절대 null을 반환해서는 안된다.

#### stream 종단 연산

stream의 종단 연산 중 상당수가 Optional을 반환한다. 

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

stream의 max 연산을 이용한 경우로 간결하게 표현할 수 있다.

## Optional 선택 기준

Optional은 검사 예외와 취지가 비슷하다. 

반환값이 없을 수도 있음을 API 사용자에게 명확히 알려서, 클라이언트에서는 반드시 이에 대한 대처 코드를 작성하도록 강제한다.

```java
// 기본값을 설정하는 경우
String lastWordInLexicon = max(words).orElse("단어 없음");

// 원하는 예외를 던지는 경우
Toy myToy = max(toys).orElseThrow(TemperTantrumExcpetion::new);
```

메서드가 Optional을 반환한다면 클라이언트는 값을 받지 못한 경우 취할 행동을 선택해야 한다.

기본값을 설정하거나, 상황에 맞는 예외를 던질 수 있다.

예외 설정의 경우, 실제 예외가 아닌 예외 팩토리를 넘겨 실제로 예외가 발생하지 않는 한 예외 생성 비용은 들지 않는다.

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

Optional에 값이 항상 채워져 있다고 확신한다면 get() 메서드로 값을 꺼내 사용하는 선택지도 있다. 이는 잘못 판단한 경우라면 NoSuchElementException을 발생시킬 것이다.

### orElseGet

기본값을 설정하는 비용이 커서 부담이 될 때가 있다.

```java
public T orElseGet(Supplier<? extends T> other)

Return the value if present, otherwise invoke other and return the result of that invocation.
```

Supplier<T>를 인수로 받는 orElseGet은 값이 처음 필요할 때 이를 사용함으로써 초기 설정 비용을 낮출수 있다.

Supplier의 특성 상 해당 값이 실제로 필요한 순간까지 값을 계산하거나 초기화하지 않아 리소스를 효율적으로 활용 가능하다.

### isPresent

```java
isPresent()
Return true if there is a value present, otherwise false.
```

Optional이 채워져 있다면 true를 비어있다면 false를 반환한다. 

### Optional stream()

```java
public Stream<T> stream​()

If a value is present, returns a sequential Stream containing only that value, otherwise returns an empty Stream.
```

Java 9에서 추가된 기능으로 Optional을 stream으로 변환해주는 어댑터이다.

Optional에 값이 있다면 그 값을 원소로 담은 stream을, 없다면 빈 stream으로 변환한다.

## Optional 사용 주의점

주의할 점은 반환 값으로 Optional을 사용한다고 해서 항상 효율적이진 않다.

- Collection, Stream, Array, Optional 같은 컨테이너 타입은 Optional로 감싸지 않기

코드로 표현하면 Optional<List<T>>보다 빈 List<T>를 반환하는 것이 낫다는 것이다.

빈 컨테이너를 그대로 반환하면 Optional 처리가 필요 없어지기 때문이다.

Optional<T>로 선언해야 하는 기본 규칙은 '결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 하는 경우'이다.

- 박싱된 기본 타입을 담은 Optional 반환하지 않기

박싱된 기본 타입을 담은 Optional은 값을 두 번 감싸기 때문에 기본 타입 자체보다 무겁다.

Java는 이를 위해 OptionalInt/Long/Double이라는 전용 Optioanl 클래스를 제공한다.

이 역시 Optional<T>가 제공하는 메서드를 제공하므로 이를 사용하자.

## 정리

Optional 반환에는 성능 저하 이슈가 있으니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다.

Optional을 반환값 이외의 용도(컬렉션의 키, 값, 원소나 배열의 원소 등..)로 쓰는 경우는 드물다.

이를 고려하여 Optional을 사용하도록 하자!

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html - java docs 
- https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/util/Optional.html - java docs
