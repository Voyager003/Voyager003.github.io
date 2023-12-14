---
layout  : wiki
title   : 표준 함수형 인터페이스를 사용하라 
summary : 
date    : 2023-12-14 11:40:33 +0900
updated : 2023-12-14 14:34:10 +0900
tag     : java effectivejava
resource: E4/C9CAC8-38DB-4BD9-97F3-AE46090C9C08
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 람다 도입으로 인한 변화

Java에서 람다를 지원하면서 API를 작성하는 사례도 크게 바뀌었는데, 예컨대 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다.

이 방법을 대체하는 현대적 해법은 함수 객체를 받는 정적 팩토리 혹은 생성자를 제공하는 방법으로 대체하는 것이다. 이 때, 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 하며 함수형 매개변수 타입을 올바르게 선택해야 한다.

item44가 예시로 든 LinkedHashMap의 protected 메서드인 removeEldestEntry()를 재정의하면 캐시로 사용할 수 있다.

```java
// 기존 메서드
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}

// 재정의한 메서드
@Override
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```

기존의 Map에 새로운 key를 추가하는 put()메서드는 이 메서드를 호출해 true가 반환되면 Map에서 가장 오래된 원소를 제거한다. 

하지만 재정의한 메서드는 Map에 원소가 100개가 될 때까지 확장하다가, 그 이상이 되면 새로운 key가 추가될 때마다 가장 오래된 원소를 하나씩 제거한다. 

이는 잘 동작하지만 람다를 사용하면 개선할 수 있다.

### 개선

```java
@Override
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
```

removeEldestEntry()는 size()를 호출하여 Map의 원소의 갯수를 알아내는데, 이것은 removeEldestEntry()가 인스턴스 메서드라서 가능한 방식이다.

인스턴스 메서드라는 것은 특정 인스턴스(위 상황의 경우 LinkedHashMap)에 속한 메서드를 의미하며, 객체의 인스턴스에 대해 호출되어 그 인스턴스의 상태(필드, 속성)에 접근하고 수정할 수 있다는 것이다.

하지만 생성자에 넘기는 함수 객체는 팩토리나 생성자를 호출할 때는 존재하지 않기 때문에(Map의 인스턴스가 완전히 생성되지 않아) Map의 인스턴스 메서드가 아니다.

따라서 Map은 자기 자신도 함수 객체어 건네줘야 하는데, 이를 반영한 함수형 인터페이스는 다음과 같다.

```java
@FunctionInterface 
interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K,V> map, Map.Entry<K, V> eldest);
}
```

하지만 Java 표준 라이브러리인 java.util.function 패키지에서 다양한 용도의 표준 함수형 인터페이스가 이를 제공하여 굳이 사용할 이유가 없다.

따라서 item44는 필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준형 함수형 인터페이스를 활용하는 것을 권한다.

## 표준 함수형 인터페이스

java.util.function 캐피지에는 총 43개의 인터페이스가 있다. (Java 21까지 변동 없음)

기본 인터페이스 6개만 기억한다면 나머지를 충분히 유추해낼 수 있다. 

- Operator<T> 인터페이스 
  
Operator 인터페이스는 인수가 1개인 Unary와 Binary로 나뉘고, 반환값과 인수의 타입이 같은 함수를 뜻한다.

함수 시그니처는 T apply(T t), T apply(T t1, T t2)로 주로 변형(transformations) 또는 결합(combinations) 작업에 사용된다.

- Predicate<T> 인터페이스

함수 시그니처는 boolean test(T t)로 하나의 인자를 받아들이고, 그 인자에 대한 불리언(boolean) 조건을 평가하여 true 또는 false를 반환한다. 

주로 필터링(filtering) 또는 매칭(matching) 조건을 정의하는 데 사용된다.

- Function<T, R> 인터페이스

시그니처는 R apply(T t)로 하나의 인자를 받고, 그 인자를 다른 타입으로 변환하여 반환한다. 

어떤 객체를 다른 타입의 객체로 변환하거나, 값을 계산하는 데 사용되며 데이터 변환(transformation) 작업에 주로 사용된다.

- Supplier<T> 인터페이스 
  
시그니처는 T get()로 인자 없이 호출되며, 특정 타입의 객체를 반환한다. 

주로 요구에 의해 새로운 객체를 생성하거나 제공할 때 사용된다.

- Consumer<T> 인터페이스

시그니처는 void accpet(T t)로 하나의 인자를 받아들이고, 그 인자에 대한 작업을 수행하지만 아무 것도 반환하지 않는다. 

주로 인자로 받은 데이터에 대해 작업을 수행하는 데 사용되며, 예를 들어 데이터 처리, 출력, 변경과 같은 작업에서 사용된다.

이 표준형 인터페이스 대부분은 대부분 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자. 이유는 계산량이 많을 때 성능이 처참하게 느려질 수 있기 때문이다. 

## 코드를 직접 작성해야 하는 경우

대부분 상황에서는 직접 작성하는 것보다 표준 함수형 인터페이스를 사용하는 편이 낫다는 것을 알았다. 그렇다면 직접 작성해야 할 상황은 언제일까?

바로 표준 인터페이스 중 필요한 용도에 맞는 게 없는 경우이다.

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

@FunctionalInterface
public interface ToIntBiFunction<T, U> {
    int applyAsInt(T t, U u);
}
```

Comparator<T> 인터페이스를 보면 구조적으로 BiFunction<T, U>와 동일하다.

하지만 직접 작성한 인터페이스인 ToIntBiFunction은 이름부터 그 용도를 자세히 설명해주며, 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있고 비교자들을 변환하고 조합해주는 디폴트 메서드를 담고 있다.

종합하면 Comparator가 독자적 인터페이스로 남아야 하는 이유가 세 가지로

- 자주 사용되고, 이름 자체가 용도를 명확히 설명하는 경우
- 반드시 지켜야할 규약이 있는 경우
- 유용한 디폴트 메서드를 제공할 수 있는 경우

가 된다.

3가지 중 하나 이상을 만족한다면 전용 함수형 인터페이스를 구현을 고민해보자.

## @FunctionalInterface

@FunctionalInterface 애노테이션은 @Override와 동일하게 프로그래머의 의도를 명시하는 것으로 크게 세가지 목적이 있다.

- 해당 인터페이스가 람다용으로 설계되었음을 알린다.
- 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일이 가능하다.- 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막는다.

직접 만든 함수형 인터페이스에 항상 @FunctionalInterface 애노테이션을 명시하자.

### 주의점 

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중정의하면 안된다.

클라이언트에게 불필요한 모호함을 주는데, 모호함으로 인한 다음과 같은  문제가 발생할 수 있다.

```
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callback<T> task);
    Future<?> submit(Runnable task);
}
```

ExecutorService 인터페이스는 Callable<T>와 Runnable을 각각 인수로 하여 다중정의했다.

올바른 메서드임을 알려주기 위해서는 submit() 메서드를 사용할 때마다 형변환을 해줘야하는 상황이 발생한다. 

이런 문제를 피하는 방법은 서로 다른 함수형 인터페이스를 같은 위치의 인수로 사용하는 다중정의를 피하는 것이다. 

## 참고자료

- 이펙티브자바 3판
