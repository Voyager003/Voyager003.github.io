---
layout  : wiki
title   : 박싱된 기본 타입보다는 기본 타입을 사용하라 
summary : 
date    : 2024-02-05 09:52:50 +0900
updated : 2024-02-05 11:23:32 +0900
tag     : java effectivejava
resource: DF/E7B1A1-E62F-4C5A-8417-18D4FA70CBA5
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java의 데이터 타입

[Java의 데이터 타입](https://voyager003.github.io/wiki/java/java_type/#primitive-type%EA%B3%BC-reference-type) 은 기본(primitive) 타입과 참조(reference) 타입으로 나눌 수 있다.

각 기본 타입에는 대응하는 참조 타입이 하나씩 있으며, 이를 박싱된 기본 타입(boxed primitive type)이라 한다. (Int -> Integer)

기본 타입과 박싱된 기본 타입의 주된 차이는 크게 세 가지다.

- 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 이에 더불어 식별성(identity)를 가진다.
    - 박싱된 기본 타입의 두 인스턴스는 값이 같아도 서로 다르다고 식별될 수 있다.
- 기본 타입의 값은 언제나 유효하지만, 박싱된 기본 타입은 null을 가질 수 있다.
- 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

코드로 살펴보자.

```java
Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1); 
```

Integer의 값을 오름차순으로 정렬하는 비교자이다. 여기에서

```java
naturalOrder.compare(new Integer(42), new Integer(42)); // 1 
```

위 값을 출력해보면? 두 인스턴스의 값이 42로 같기 때문에 0을 출력할 것으로 기대했지만 실제로는 1을 출력한다. (첫 번째 Integer가 두 번째 Integer보다 큼)

여기서 두 '객체 참조'의 식별성을 검사하게 되는데, 값은 같더라도 i와 j가 서로 다른 인스턴스이기 때문에 비교의 결과가 false가 되는 것이다. 

이처럼 같은 객체를 비교하는 것이 아니라면 박싱된 기본 타입에 == 연산자를 사용한다면 예상치 못한 결과를 가져온다.

그에 반해 기본 타입 정수로 저장한 뒤에 비교를 수행한다면 

```java

Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed, int j = jBoxed; // 오토박싱
    return (i < j) ? -1 : (i == j ? 0 : 1);
};
```

오류의 원인인 식별성 검사가 이뤄지지 않는다.

### null 문제

```java
public class Unbelievable {
    static Integer i;
    
    public static void main(String[] args) {
        if (i == 42) {
            System.out.println("Unbelievable");
        }
    }
}
```

이 코드에서 i == 42를 검사할 때 NullPointerException를 던진다. 

i가 int가 아닌 Integer이기 때문에 다른 참조 타입 필드와 마찬가지로 i의 초깃값이 null이기 때문이다. 

기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 자동으로 풀리게 된다. 

이를 해결하려면 i를 기본 타입인 int로 선언해주면 된다.

### 시간과 메모리 효율

```java
Long sum = 0L

for (long i = 0; i <= Integer.MAX_VALUE; i++){
    sum += 1;
}
System.out.println(sum);
```

지역변수 sum을 박싱된 기본타입으로 선언한 경우이다. 오류없이 컴파일 되지만 박싱, 언박싱이 반복해서 발생하여 성능이 느려진다.

item61에서 다룬 세 문제의 원인은 하나로, 기본 타입과 박싱된 기본 타입의 차이를 무시한 것이다.

### 박싱된 기본 타입을 사용하는 경우

그렇다면 박싱된 기본타입을 언제 사용해야 할까?

- 컬렉션의 원소, 키, 값으로 사용할 경우
- 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로 사용하는 경우(컬렉션은 기본 타입을 담을 수 없음)
- 리플렉션을 통해 메서드를 호출하는 경우 

## 참고자료

- 이펙티브자바 3판



