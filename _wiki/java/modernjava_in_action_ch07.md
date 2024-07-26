---
layout  : wiki
title   : 병렬 데이터 처리와 성능 
summary : 
date    : 2024-07-26 22:58:40 +0900
updated : 2024-07-26 22:59:16 +0900
tag     : java
resource: 95/1AD2FC-8397-47A0-947B-BA056E9C3653
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 병렬 데이터 처리와 성능
### 병렬 스트림(parallel stream)
`병렬 스트림` 은 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크(덩어리)로 분할한 스트림이다. 이를 사용함으로써 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다.

기본적으로 Java의 모든 스트림 작업은 명시적으로 병렬로 지정되지 않는 한 순차적으로 처리된다. 순차 스트림은 단일 스레드(single thread)를 사용해 파이프라인을 처리하게 된다.

```java
// 순차 스트림
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4); 
listOfNumbers.stream().forEach(number -> 
	System.out.println(number + " " + Thread.currentThread().getName()) 
	);
```
위와 같이 동작하는 스트림을 실행해보면
```
1 main
2 main
3 main
4 main
```
예측 가능한 순차 스트림으로 각 요소는 항상 순서대로 출력이 된다.

이를 병렬 스트림으로 변환하려면 어떻게 할까?\
방법은 순차 스트림에 병렬 메서드(parallel())를 추가하거나 컬렉션의 parallelStream 메서드를 사용하면 된다.

```java
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);
listOfNumbers.parallelStream().forEach(number ->
    System.out.println(number + " " + Thread.currentThread().getName())
);
```
컬렉션의 parallelStream 메서드를 사용해 병렬 스트림으로 변환한 코드이다.

병렬 스트림을 사용했기 때문에 별도의 코어에서 병렬로 실행할 수 있다. 실행 결과를 살펴보면

```java
2 ForkJoinPool.commonPool-worker-1
3 main
4 ForkJoinPool.commonPool-worker-3
1 ForkJoinPool.commonPool-worker-2
```
순차 스트림과 달리 포크조인 풀(ForkJoinPool)이라는 프레임워크를 통해 실행되는 것을 확인할 수 있다.

실제로 코드를 실행할 때마다 다른 결과를 출력하며, 실행 순서를 통제할 수 없다는 것이 특징이다.
### 포크 조인 프레임워크(Fork-join Framework)
Java 7에 추가된`포크조인 프레임워크`는 병렬화 작업을 재귀적으로 작은 작업으로 분할한 뒤 서브태스크 각각의 결과를 합쳐 전체 결과를 만들도록 설계되었다.

병렬 스트림은 내부적으로 `포크조인 풀`을 사용하며, 기본적으로 프로세서 수가 반환하는 값에 상응하는 스레드를 갖는다. 작업들을 분할 가능한만큼 쪼개고, 쪼개진 작업들을 작업자 스레드를 통해 작업 후 합치는 과정으로 결과를 만들어 낸다.

Fork / Join Framework의 핵심은 AbstractExecutorService 클래스를 확장한 `ForkJoinPool`로 다른 종류의 ExecutorService와 다르게 Work-Stealing 메커니즘을 사용한다.

![[Pasted image 20240725102752.png]]

순서대로 살펴보면
-> task를 보내고. (submit)  
-> inbound queue에 task가 들어가고, A와 B 스레드가 가져다가 task를 처리
-> A와 B는 각자 큐가 있으며, 위 그림의 B처럼 큐에 task가 없으면 A의 task를 steal 하여 처리한다.

work-stealing 메커니즘을 사용하기 때문에 CPU 자원이 놀지 않고 최적의 성능을 낼 수 있게 된다. 

스레드 자신의 task queue로 deque를 사용한다. deque는 양 쪽 끝으로 넣고 뺄 수 있는 구조이며, 각 스레드는 deque의 한쪽 끝에서만 일하고 나머지 반대쪽에서는 task를 훔치러 온 다른 스레드가 접근한다.
#### 소스 분할(Splitting Source)
포크조인 프레임워크는 작업자 스레드(worker thread) 간에 데이터를 분할하고 작업이 완료되면 콜백을 처리하는 일을 담당한다.

정수의 합을 병렬로 계산하는 경우로 살펴보자.

```java
List<Integer> listOfNumbers = Arrays.asList(1, 2, 3, 4);
int sum = listOfNumbers.parallelStream().reduce(5, Integer::sum);
assertThat(sum).isNotEqualTo(15);
```
reduce 메서드를 사용해 0부터 시작하는 대신 시작 합계에 5를 더한다.

순차 스트림의 경우는 당연하게도 결과가 15겠지만, reduce 작업은 병렬로 처리되므로 숫자 5는 실제로 모든 작업자 스레드에 합산된다.

![[Pasted image 20240723171950.png]]

실제 결과는 공통 포크조인 풀에 사용되는 스레드 수에 따라 다를 수 있다.

그래서 이를 해결하려면 병렬 스트림 외부에 숫자 5를 추가하는 작업을 해줘야한다.

```java
int sum = listOfNumbers.parallelStream().reduce(0, Integer::sum) + 5;
```
### 성능 비교
for 루프를 사용하는 경우와, 순차적 스트림을 사용하는 경우, 병렬 스트림을 사용하는 경우를 비교해 보자.

```java
// for  루프
public long iterativeSum(long n) {
      long result = 0;
      for (long i = 1L; i<=n; i++) {
        result += i;
      }
      return result;
}

// 순차적 스트림
public long sequentialSum(long n) {
	return Stream.iterate(1L, i -> i 7+ 1)
				 .limit(n)
				 .reduce(0L, Long::sum);
}

// 병렬 스트림
public long parallelSum(long n) {
	return Stream.iterate(1L, i -> i 7+ 1)
				 .limit(n)
				 .parallel()
				 .reduce(0L, Long::sum);
}
```

for 루프를 사용해 방식은 저수준으로 동작할 뿐 아니라 primitive type을 박싱 하거나 언박싱할 필요가 없으므로 더 빠를 것이라 예상할 수 있다. 

![](https://blog.kakaocdn.net/dn/Qn0iU/btskULpKopW/ke5CikpPi6dNzul7PLKvB0/img.png)

![](https://blog.kakaocdn.net/dn/mRpBx/btskUMhSb1I/szd1fl4oxxjfUmoqCs0no1/img.png)

위의 결과가 for 루프, 아래가 순차적 스트림을 사용한 결과이다.

순차적 스트림을 사용하는 버전에 비해 거의 4배가 빠르다는 것을 확인할 수 있다.

병렬 스트림을 사용하는 버전은 어떨까?

![](https://blog.kakaocdn.net/dn/dY2N2U/btskZsbDuGc/37t2DwuMNh6La92rvpnuqk/img.png)

병렬 버전이 순차 버전에 비해 다섯 배나 느린 결과가 나오는데, 원인은 다음과 같이 유추할 수 있다.

- 반복 결과로 박싱 된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야 하며,
- 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기가 어렵다.

두 번째 문제는 `iterate`연산은 이전 연산 결과에 따라 다음 함수의 입력이 달라지기 때문에 청크로 분할하기 어렵다.

reduce() 과정을 시작하는 지점에 전체 수자 리스트가 준비되지 않았으므로 스트림을 병렬로 처리할 수 있도록 청크로 분할할 수 없다.

스트림이 병렬로 처리되도록 지시했고 각각의 합계가 다른 스레드에서 수행되었지만 결국 순차처리 방식과 크게 다른 점이 없으므로 스레드를 할당하는 오버헤드만 증가하게 된 것이다.

이처럼 병렬 프로그래밍을 오용하면 오히려 전체 프로그램의 성능이 더 나빠질 수도 있다.

문제를 해결하려면 `LongStream.rangeClosed`라는 메서드를 통해 해결할 수 있다.

이는 `iterate`에 방식에 비해 다음과 같은 장점을 제공한다.

- 기본형 long을 직접 사용하므로 박싱과 언박싱 오버헤드가 사라지며
- 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다.

```java
public long rangedSum(long n) {
    return LongStream.rangeClosed(1, n)
        .reduce(0L, Long::sum);
}
```

![](https://blog.kakaocdn.net/dn/CAyB1/btskT17foEn/CXhEPK6Ob2izHb8Zqxj7A1/img.png)

기존의 `iterate`를 사용한 버전보다 스트림 처리 속도가 더 빠른데, 이는 특화되지 않은 스트림을 처리할 때는 오토박싱, 언박싱 등의 오버헤드를 수반하기 때문이다.

상황에 따라서는 어떤 알고리즘을 병렬화하는 것보다 적절한 자료구조를 선택하는 것이 더 중요하다는 사실을 단적으로 보여준다.

병렬 스트림을 적용하면 다음과 같다.

```java
public long parallelRangedSum(long n) {
    return LongStream.rangeClosed(1, n)
        .parallel()
        .reduce(0L, Long::sum);
}
```

<img width="502" alt="SCR-20240725-sign" src="https://github.com/user-attachments/assets/7a182e86-2dd2-4d23-85f0-d8a5c3257b25">

순차 실행보다 빠른 성능을 갖는 병렬 리듀싱을 만들었다.

병렬화를 이용하려면 스트림을 재귀적으로 분할해야 하고, 각 서브 스트림을 서로 다른 스레드의 리듀싱 연산으로 할당하고, 이들 결과를 하나의 값으로 합쳐야 한다.

멀티코어 간의 데이터 이동은 우리 생각보다 비싸다. 따라서 코어 간의 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 다른 코어에서 수행하는 것이 바람직하다.

#### 병렬 스트림 사용 지침

- 확신이 서지 않으면 직접 성능을 측정한다.
- 박싱을 주의하라. 오토박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소로, 기본형 특화 스트림을 통해 박싱 동작을 피할 수 있다. (IntStream, LongStream, DoubleStream)
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다. 특히 limit이나 findFirst처럼 요소의 순선에 의존하는 연산을 병렬 스트림에서 수행하려면 비싼 비용을 치러야 한다.
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라.
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다. 이 또한 직접 성능을 측정한다.
- 스트림을 구성하는 자료구조가 적절한지 확인하라. 예를 들어 ArrayList를 LinkedList보다 효율적으로 분할할 수 있다.
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
- 최종 연산의 병합 과정(예를 들면 Collector의 combiner 메서드) 비용을 살펴보자. 병합 과정의 비용이 비싸다면 병렬 스트림으로 얻은 성능의 이익이 서브스트림의 부분 결과를 합치는 과정에서 상쇄될 수 있다.

## 참고자료

- 모던 자바 인 액션
- https://www.baeldung.com/java-when-to-use-parallel-stream - 밸덩
- https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html - 포크 조인 풀 공식 문서
- https://m.blog.naver.com/tmondev/220945933678 - Java8 Parallel Stream 
