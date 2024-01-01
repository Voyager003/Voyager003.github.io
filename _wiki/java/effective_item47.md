---
layout  : wiki
title   : 반환 타입으로는 스트림보다 컬렉션이 낫다 
summary : 
date    : 2024-01-01 14:55:36 +0900
updated : 2024-01-01 18:52:10 +0900
tag     : java
resource: F8/0EEA9D-59A7-4B06-A73A-42A2F6E4581D
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 스트림에서의 반복(iteration)

스트림(Stream) 자체는 전통적인 for-each 반복문을 직접적으로 지원하지 않는다.

이는 스트림이 Iterable 인터페이스를 확장(extend)하지 않기 때문이다.

이 문제를 명확하게 해결해줄 우회로는 없다.

```java
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // 프로세스 처리
} // 컴파일 에러: method reference not expected here for (ProcessHandle ph : ^) 
```

위와 같이 메서드 참조를 하게되면 for-each 구문이 메서드 참조를 직접적으로 반복 가능한 형태로 인식이 불가능하기 때문이다. (Iterable을 인터페이스를 구현한 객체가 아니기 때문) 

메서드 참조 문제를 해결하기위해 우회한 방법은 다음과 같다.

```java
for(ProcessHandle ph : (iterableOf<ProcessHandle>)ProcessHandle.allProcesses::iterator){
    // 프로세스 처리
}
```

이 코드는 작동은 하지만 형변환으로 인해 난잡하고 직관성이 떨어진다. 문제점을 어댑터[^1]와 타입 추론으로 개선해보자.

```java
// 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}

// 어댑터를 적용한 코드
for(ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) { 
    // 프로세스 처리
}
```

이 때 API가 Iterable만 반환하면 스트림 파이프라인에서 처리하는 상황에서 문제가 된다. Iterable<E>를 Stream<E>로 중개하는 방법은 어떨까?

```java
public static <E> Stream<E> streamOf(Iterable<E> iterable) { 
    return StreamSupport.stream(iterable.spliterator(), false);
}
```

객체 시퀀스를 반환하는 메서드를 작성할 때, 메서드가 오직 스트림 파이프라인에서만 쓰이는 것을 알고 있다면 스트림을 반환하도록 만들자.

하지만 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다. 

따라서 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 것이 일반적으로 최선이다. 

유의할 점은 반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 ArrayList나 HashSet과 같은 표준 구현체를 반환하는 게 최선일 수 있지만, 단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.

## 전용 컬렉션 구현

반환할 시퀀스가 크지만 표현을 간결하게 할 수 있다면 전용 컬렉션 구현을 검토하자.

item47은 주어진 집합의 멱집합[^2]을 반환하는 상황을 예시로 들고 있다.

이 경우 원소가 n개라면 멱집합의 갯수는 2^n개가 된다. 이를 AbstractList를 이용해 전용 컬렉션을 구현할 수 있다.

```java
public class PowerSet {
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if(src.size() > 30) {
            throw new IllegalArgumentException("집합에 원소가 너무 많습니다.(원소 최대 30개) : " + s);
        }

        return new AbastractList<Set<E>>() {
        
        @Override 
        public int size() {
            return 1 << src.size();
      }

        @Override 
        public boolean contains(Object o){
            return o instanceof Set && src.containsAll((Set)o);
      }

        @Override 
        public Set<E> get(int index){
            Set<E> result = new HashSet<>();
            for(int i = 0; index !=0; i++, index >>= 1){
                if((index & 1) == 1){
                    result.add(src.get(i));
            }
        }
        return result;
    }
    };
  }
}
```

각 원소의 인덱스를 비트 벡터로 사용해 인덱스의 n번 째 비트 값은 멱집합의 해당 원소가 원래 집합의 n번째 원소를 포함하는지 여부를 알려준다. 결과적으로 0부터 2^n - 1까지의 이진수와 원소 n개인 집합의 멱집합과 매핑된다.

item47의 예시의 경우 전용 컬렉션을 구현하는 것이 스트림을 사용하는 것보다 1.4배 빠르고, 어댑터 형식은 스트림보다 약 2.3배 느리다는 것이 확인이 되었다.

## 참고자료

- 이펙티브자바 3판
- https://www.baeldung.com/java-iterable-to-stream

## 주석

[^1]: 클라이언트가 요구하는 인터페이스와 호환되지 않는 인터페이스를 가져 함께 동작할 수 없는 클래스를 함께 동작할 수 있게 해주는 패턴 : https://invincibletyphoon.tistory.com/20

[^2]: 한 집합의 모든 부분 집합을 원소로 하는 집합.


