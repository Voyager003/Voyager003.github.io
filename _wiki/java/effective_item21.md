---
layout  : wiki
title   : 인터페이스는 구현하는 쪽을 생각해 설계하라 
summary : 
date    : 2023-09-06 19:57:40 +0900
updated : 2023-09-06 21:00:02 +0900
tag     : java effectivejava
resource: A2/B7D95A-75CE-421D-B1DD-AD61C01BD94F
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 인터페이스의 디폴트 메서드(Default method)

Java 8 이전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가할 방법이 없었지만, 디폴트 메서드가 등장하면서 가능해졌다. ([디폴트 메서드](https://voyager003.github.io/wiki/java/java_interface/#%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%9D%98-default%EA%B8%B0%EB%B3%B8-%EB%A9%94%EC%84%9C%EB%93%9Cjava-8)) 

디폴트 메서드 선언 시에 인터페이스를 구현한 후 디폴트 메서드를 정의하지 않은 모든 클래스에서 디폴트 구현이 쓰이게 된다. 기존 인터페이스에 메서드를 추가하는 길이 열렸지만 매끄럽게 연동되리라는 보장이 없고, 디폴트 메서드는 구현 클래스에 대해 합의없이 무작정 삽입될 뿐이다.

Java 라이브러리의 디폴트 메서드는 코드 품질이 높고 범용적이라 대부분 상황에서 잘 작동한다. 하지만 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하긴 어렵다.

Item21은 Collection 인터페이스의 removeIf를 예시로 드는데 살펴보자.

```java
default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```

이 메서드는 boolean 함수인 predicate가 true를 반환하는 모든 원소를 제거한다. 반복자를 이용해 순회하면서 각 원소의 인수로 넣어 predicate를 호출하고, predicate가 truef르 반환하면 반복자의 remove메서드를 호출하여 그 원소를 제거한다.

이보다 범용적으로 구현하기도 어렵겠지만, 그렇다고 모든 Collection 구현체와 잘 어우러지는 것은 아니다. 예시로 아파치 커먼즈 라이브러리의 클래스인 SynchronizedCollection 살펴보자.

```java
@Override
public boolean removeIf(Predicate<? super E> filter) {
    synchronized(lock) {
        return decorated().removeIf(filter);
    }
}
```

아파치 라이브러리의 4.4버전에서는 removeIf()를 위와 같이 재정의하고 있다. 4.4 이전에는 removeIf가 재정의되어있지 않아서 디폴트 구현을 물려받아 모든 메서드 호출을 알아서 동기화하지 못했다. 

동기화에 대해 아무것도 모르기 때문에 Lock 객체를 사용할 수 없으며, SynchronizedCollection 인스턴스를 여러 스레드가 공유하는 환경에서 removeIf를 호출하면 ConcurrentModifcationException이 발생하거나 다른 예상치 못한 결과로 이어질 수 있다. 

## 인터페이스 설계 시 주의점

디폴트 메서드는 컴파일에 성공하더라도 기존 구현체에 Runtime Error를 일으킬 수 있다.

따라서 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니면 피하고 추가하려는 디폴트 메서드가 기존 구현체들과 충돌하지는 않는지 고려해야한다.

반면, 새 인터페이스를 만드는 경우는 표준적인 메서드 구현을 활용하게 할 수 있게끔해주는 유용한 수단이다. (item20)

한편으로는, 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을 명시해야한다. 이는 클라이언트를 망가뜨리게 된다. 

새로운 인터페이스는 release 전에 반드시 테스트를 거치고, release 후에도 결함을 수정하는게 가능한 경우도 있지만, 이 가능성에 기대서는 안된다. 

## 참고자료

- 이펙티브자바 3판


