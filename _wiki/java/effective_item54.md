---
layout  : wiki
title   : null이 아닌, 빈 컬렉션이나 배열을 반환하라 
summary : 
date    : 2024-01-23 09:58:14 +0900
updated : 2024-01-23 10:24:11 +0900
tag     : java effectivejava
resource: 7A/1E529D-4773-44A4-81A4-C322323A4E88
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## null을 반환하는 경우

```java
private final List<Cheese> cheesesInStock = ...;

/**
* @return 매장 안의 모든 치즈 목록 반환
* 단, 재고가 없다면 null반환
*/
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

삼항 연산자로 재고가 하나도 없다면(컬렉션이 비었다면) null을 반환하고, 그렇지 않다면 cheesesInStock을 반환한다. 

이 코드처럼 null을 반환한다면 null을 처리하는 코드를 추가로 작성해야 한다.

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON)) {
    ... // 예외 처리
}
```

컬렉션 혹은 배열과 같은 컨테이너가 비었을 경우 null을 반환하는 메서드는 항상 방어 코드를 넣어줘야 한다.

방어 코드를 넣지 않는다면 실제로 객체가 0개일 가능성이 거의 없는 상황에서는 수년 뒤에야 오류가 발생하기도 하고, 한편 null을 반환 하려면 반환하는 쪽에서도 이 상황을 특별 취급해줘야 해서 코드가 더 복잡해진다.

### null을 반환하는 것이 낫다?

빈 컨테이너를 할당하는 데도 비용이 들기 때문에 null을 반환하는 것이 낫다고 생각할 수 있지만 두 가지 면에서 틀린 주장이다.

먼저 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못된다.

두 번째로 빈 컬렉션과 배열은 새로 할당하지 않고도 반환이 가능하다. 

```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

가능성은 작지만 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어뜨릴 수 있다. 이는 매번 같은 빈 '불변'컬렉션을 반환하도록 하면된다.

불변객체는 자유롭게 공유해도 안전하기 때문이다.

Collections.emptyList, emptySet, emptyMap이 그 예시로, 이 경우 최적화가 필요하다고 판단되면 수정 전,후의 성능을 측정해 실제 성능이 개선되는지 확인할 필요가 있다.

배열의 경우도 마찬가지다.

```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```

길이가 0인 배열을 반환하면 된다. 보통의 경우 정확한 길이의 배열을 반환하면 된다.

이 방식이 성능을 떨어뜨릴 것 같다면, 길이 0짜리 배열을 미리 선언해두고 매 번 그 배열을 반환해보자. 

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

getCheese 메서드에서는 항상 EMPTY_CHEESE_ARRAY를 인수로 넘겨 toArray를 호출하는데, 이는 cheesesInStock가 비었다면 언제나 EMPTY_CHEESE_ARRAY를 반환하게 된다. 

이 때 주의할 점은 단순히 성능 개선이 목적이라면 

```java
...
return cheeseInStock.toArray(new Cheese[cheeseInStock.size()]);
```

toArray에 넘기는 배열을 미리 할당하는 것은 오히려 성능이 떨어진다는 결과도 있다는 점을 유의하자.

## 참고자료

- 이펙티브자바 3판
 
