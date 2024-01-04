---
layout  : wiki
title   : 스트림 병렬화는 주의해서 적용하라 
summary : 
date    : 2024-01-04 12:49:42 +0900
updated : 2024-01-04 13:52:10 +0900
tag     : java effectivejava
resource: 27/F8F8FA-9C7F-49CA-8536-1AC83962AE1A
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java의 동시성 프로그래밍 기술 

Java 5의 util.concurrent 라이브러리와 실행자 프레임워크, Java 7의 folk-join 그리고 Java 8에서는 parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림, Java 21에서 도입된 가상 스레드를 지원하는 등 Java에서 동시성 프로그램을 작성하기 쉽도록 여러 기능이 추가되었다.

동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 신경써야해서 올바르고 빠르게 작성하는 일은 여전히 어려운 작업이다. 이는 병렬 스트림 파이프라인 프로그래밍에서도 다르지 않다.

```java
public statid void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

메르센 소수를 생성하는 프로그램인데, 이를 실행하면 즉각 소수를 찍기 시작하면서 12.5초만에 완료된다. 여기서 속도를 높이기 위해 스트림 파이프라인의 parallel()을 호출한다면 성능은 어떻게 변할까?

결과는 아무것도 출력하지 못하고 CPU를 점유한 상태가 무한히 계속된다.(liveness failure)

이 문제의 원인은 **스트림 라이브러리가 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문**이다.

파이프라인 병렬화가 작업을 CPU의 코어 수만큼 병렬로 수행한다고 해보자. 

메르센 소수를 구하는 코드에서 쿼드 코어 시스템에서 수행하면 19번째 계산까지 마치고 마지막 20번 째 연산이 수행되는 시점에 CPU 코어 3개가 한가한 상태이다. 따라서 21, 22, 23번 째 메르센 소수를 찾는 작업이 병렬로 시작되는데, 20번 째 계산이 끝나더라도 이 계산들은 끝나지 않는다. 각각 20번 째 계산보다 2배, 4배, 8배의 시간이 더 필요하기 때문이다.

이처럼 파이프라인은 자동 병렬화 알고리즘이 제 기능을 못하도록 마비시키기때문에 스트림 파이프라인을 마구잡이로 병렬화하면 안된다. 성능 저하로 이어지기 때문.

## 병렬화의 효과가 좋은 경우

스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다. 

위 예시의 공통점은 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 일을 다수의 스레드에 분배하기에 좋다는 특징이 있다. 

또한 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference)이 뛰어나다는 것, 즉 이웃한 원소의 참조값들이 메모리에 연속하여 저장되어 있다는 것이다. 반대로 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져있을 수 있는데, 이는 참조 지역성이 나빠진다. 

참조 지역성이 나쁘다는 것은 스레드가 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 허비한다는 것이다. 이처럼 중요한 요소로 작용한다.

### 스트림의 종단 연산

종단 연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다. 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행의 효과는 제한될 수 밖에 없다.

종단 연산 중에 병렬화에 가장 적합한 것은 축소(reduction)으로 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다. 정리해보면 

- Stream의 reduce 메서드
- min, max, count, sum같이 완성된 형태로 제공되는 메서드
- anyMatch, allMatch, noneMatch와 같이 조건에 맞으면 반환되는 메서드

가 있다. 반면 가변 축소(mutable reduction)를 수행하는 스트림의 collect는 컬렉션들을 합치는 부담이 크기 때문에 병렬화에 적합하지 않다.

### 스트림 병렬화 주의점

스트림을 잘못 병렬화하면 성능뿐만 아니라 결과 자체가 잘못되거나 예상치 못한 동작(safety failure)이 발생할 수 있다. 

Stream 명세는 이 때 사용되는 함수 객체에 관한 엄중한 규약을 정의했다. [^1]

예시로 스트림 reduce 연산에 건네지는 accumulator(누적자)와 combiner(결합자) 함수는 반드시 결합법칙을 만족하고, 간섭받지 않고 상태를 갖지 않아야 한다고 설명한다.

요구사항을 지키지 못하는 상태라도 파이프라인을 순차적으로 수행한다면 올바른 결과를 얻을 수도 있겠지만 병렬로 수행하면 실패로 이어진다.

병렬화는 오직 성능 최적화 수단임을 기억해야 한다. 다른 최적화와 마찬가지로 변경 전후로 반드시 성능을 테스트해 병렬화를 사용할 가치가 있는지 확인해야 한다. 

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html#filter-java.util.function.Predicate- - stream docs

## 주석

[^1]: Most stream operations accept parameters that describe user-specified behavior, such as the lambda expression w -> w.getWeight() passed to mapToInt in the example above. To preserve correct behavior, these behavioral parameters: ...
