---
layout  : wiki
title   : 객체는 인터페이스를 사용해 참조하라  
summary : 
date    : 2024-02-12 09:45:18 +0900
updated : 2024-02-12 10:02:04 +0900
tag     : java effectivejava
resource: 0D/866A2E-28B2-4378-AE08-DE2765C84352
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 인터페이스 사용으로 인한 이점 

적합한 인터페이스가 있다면 매개변수, 반환값, 변수, 필드를 모두 인터페이스 타입으로 선언해보자.

객체의 실제 클래스를 사용해야 할 상황은 오직 '생성자'로 생성할 때뿐이다.

```java
// 좋은 예
Set<Son> sonSet = new LinkedHashSet<>();

// 나쁜 예
LinkedHashSet<Son> sonSet = new LinkedHashSetM<>();
```

위와 같이 인터페이스를 타입으로 사용한다면, 구현 클레스를 교체하고자할 때 새 클래스를 호출해주기만 하면 되기 때문에 훨씬 유연한 코드를 작성할 수 있다.

주의할 점은 원래 클래스가 인터페이스의 일반 규약 외의 특별한 기능을 제공하며, 주변 코드가 이 코드에 기대어 동작한다면 새로운 클래스도 반드시 같은 기능을 제공해야 한다.

```java
Set<Son> sonSet = new LinkedHashSet<>();  
```

예를 들어 첫 번째 선언의 주변 코드가 LinkedHashSet이 따르는 순서 정책을 가정하고 동작하는 상황에 HashSet으로 바꾼다면, HashSet은 반복자의 순회 순서를 보장하지 않기 때문에 문제가 될 수 있다. 이를 염두에 두자.

## 적합한 인터페이스가 없는 경우

- `String` , `BigInteger` 와 같은 값 클래스
- `클래스 기반으로 작성된 프레임워크` 가 제공하는 객체들 
    - ex: java.io 패키지의 OutputStream 등..)
- `인터페이스에 없는 특별한 메서드를 제공` 하는 클래스
    - PriorityQueue는 Queue 인터페이스 없는 비교자(comparator)를 제공
    - 클래스 타입 사용 시, 추가 메서드를 꼭 사용해야하는 경우를 최소화 해야함

결론적으로 클래스 계층구조 중 필요 기능을 만족하는 가장 덜 구체적(상위) 클래스를 타입으로 사용하도록 하자.

## 참고자료

- 이펙티브자바 3판

