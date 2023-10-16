---
layout  : wiki
title   : 이왕이면 제네릭 메서드로 만들라 
summary : 
date    : 2023-10-16 11:02:13 +0900
updated : 2023-10-16 15:13:44 +0900
tag     : java effectivejava
resource: B0/BB4E4A-D637-4109-9A2A-3BA38A7473A3
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 제네릭 메서드(Generic method)

[클래스](https://voyager003.github.io/wiki/java/effective_item29/) 와 마찬가지로 메서드도 제네릭으로 만들 수 있다.

```java
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1); // raw type
  result.addAll(s2);
  return result;
}
```

위 코드는 로 타입(rwa type)을 사용한 두 Set의 합을 반환하는 메서드를 지는데, 컴파일은 되지만, unchecked 경고가 발생한다.

경고를 없애려면 타입 안전하게 만들어야 하는데, 제네릭을 사용해 개선해보자.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

 메서드 선언에서 원소 타입(<E>)을 타입 매개변수로 지정하고, 이를 메서드 내에서 타입 매개변수를 사용하도록 바꿨다. 
 
 이제 경고 없이 컴파일되고, union 메소드에 String 객체만 포함된 Set와 Integer 객체만 포함된 Set을 전달하려고 하면 컴파일러가 오류를 발생시킨다. 따라서 잘못된 타입의 객체가 메서드로 전달되어 런타임에 예외가 발생하는 것을 사전에 방지하는 타입 안정성을 얻을 수 있게 되었다.
 
## 제네릭 싱글톤 팩토리 패턴(Generic singleton factory pattern)

불변 객체를 여러 타입으로 활용할 수 있게 만들어야 하는 경우가 있다.

```java
// Collections.emptySet
@SuppressWarnings("unchecked")
public static final <T> Set<T> emptySet() {
  return (Set<T>) EMPTY_SET;
}
```

예시로 Collections의 emptySet() 메서드는 제네릭을 이용해 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주게 된다.

제네릭은 런타임에 타입 정보가 소거되어 객체를 어떤 타입으로든 매개변수화 할 수 있는 데, 이를 위해 요청한 타입 매개변수에 맞게 그 객체의 타입을 바꿔주는 정적 팩토리 메서드를 만든 것이다.

이 패턴을 제네릭 싱글톤 팩토리 패턴이라하며 코드에 유연성을 더해준다.

제네릭 싱글톤의 특징을 이용해 제네릭 항등 함수를 담은 클래스를 만들어보자.

```java
private static UnaryOperator<Object> IDNTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
private static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDNTITY_FN;
}
```

제네릭이 실체화된다면 항등 함수를 타입별로 하나씩 만들어야 했지만, 타입 소거를 했기 때문에 런타임에서는 그 정보를 알지 못하는 상태로 제네릭 싱글톤 하나로 이를 해결할 수 있다.

이 때 IDENTITY_FN를 UnaryOperator로 형변환을 시도하면 T가 어떤 타입이든 UnaryOperator<Object>는 UnaryOperator<T>가 아니기 때문에 경고가 발생한다. 

하지만 항등 함수는 입력 값을 수정없이 그대로 반환하는 특별한 함수이므로, T에 어떤 타입이 오건 안전한 타입이라고 알리기 위해 @SupperessWarnings 애노테이션으로 해당 경고를 없애주었다.

이처럼 제네릭 싱글톤은 하나의 메서드로 여러 타입에 대응할 수 있는 유연함을 준다.

### 재귀적 타입 한정(recursive type bound)

드물지만 자기 자신이 들어간 표현식을 사용해 타입 매개변수의 허용 범위를 한정할 수 있다. 이를 재귀적 타입 한정이라 하며 다음과 같이 사용한다.

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

주로 타입의 자연적 순서를 정하는 Complarable과 함께 쓰인다.

타입 매개변수 T는 Comparable<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다. 

Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬, 검색, 비교하는 용도로 사용되며, 해당 용도로 사용하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 하는데 이 제약을 코드로 표현해야하는데

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

'모든 타입 E는 자신과 비교할 수 있다'라는 의미로 상호 비교 가능하다는 의미를 명확하게 표현했다.

## 마무리 

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다. 읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브 자바

