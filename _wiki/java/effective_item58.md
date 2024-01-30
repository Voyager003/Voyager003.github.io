---
layout  : wiki
title   : 전통적인 for문보다는 for-each문을 사용하라 
summary : 
date    : 2024-01-30 09:23:13 +0900
updated : 2024-01-30 10:27:47 +0900
tag     : java effectivejava
resource: A5/128FCF-4B79-44B0-A212-59EB51977BFF
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 전통적인 for문

```java 
// 컬렉션 순회
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
}

// 배열 순회
for (int i = 0; i< a.length; i++) {
    ...
}
```

이러한 전통적인 for문에 나타나는 문제점은 다음과 같다.

- 필요한 요소는 원소들 뿐이지만, 반복자(Iterator)와 인덱스 변수(i)가 코드를 너저분하게 만든다는 점
- 쓰이는 요소 종류가 늘어나면 오류가 생길 수 있다는 점
- 잘못된 변수를 사용했을 때 컴파일러가 잡아줄 수 있다는 보장이 없다는 점
- 컬렉션, 배열 등 종류에 따라 코드 형태가 달라질 수 있다는 점

이 점은 for-each문을 사용하면 해결할 수 있다.

## for-each문(향상된 for문: enhanced for statemenet)

```java
for (Element e : elements) {
    ...
}
```

for-each문은 반복자와 인덱스변수를 사용하지 않고, 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있어 어떤 컨테이너를 다루는지 신경쓰지 않고 깔끔한 코드 작성이 가능하다.

여기서 컬렉션을 중첩해 순회해야 한다면 for-each문의 이점은 더 커진다.

```java
// ENUM
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

// 전통적 for문 순회
List<Card> deck = new ArrayList<>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
        deck.add(new Card(i.next(), j.next())); 
```

이 for문의 문제는 바깥 컬렉션(Suits)의 반복자에서 next 메서드의 호출이 너무 많다는 것이다.

i.next()는 Suit 하나 당 한번 씩만 호출되어야 하는데, 안쪽 반복문에서 호출되어 Rank 하나 당 한번 씩 호출되고 있다. 따라서 숫자가 바닥나면 반복문에서 NoSuchElementException를 던지게 된다.

for-each문은 이런 문제를 간단하게 해결한다.

```java
for (Suit suit : suits)
    for (Rank rank : ranks)
        deck.add(new Card(suit, rank));
```

## for-each문의 제약

item58에서 말하는 for-each문을 사용할 수 없는 상황 세 가지이다.

- 파괴적 필터링(destructive filtering)

컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.

Java 8부터는 Collection의 removeIf() 메서드를 사용해 컬렉션으로 명시적으로 순회하는 일을 피할 수 있다.

- 변형(transforming)

리스트, 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자, 배열의 인덱스를 사용해야 한다.

- 병렬 반복(parallel iteration)

여러 컬렉션을 병렬로 순회해야 한다면 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

## 정리

for-each문은 컬렉션, 배열, Iterable 인터페이스를 구현한 객체라면 무엇이든 순회 가능하다.

Iterable을 처음부터 직접 구현하기는 까다롭지만, 이를 구현하면 성능 저하도 없으며 버그를 예방할 수 있다.

## 참고자료

- 이펙티브자바 3판

