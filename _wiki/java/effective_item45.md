---
layout  : wiki
title   : 스트림은 주의해서 사용하라 
summary : 
date    : 2023-12-18 09:44:04 +0900
updated : 2023-12-18 11:33:51 +0900
tag     : java effectivejava
resource: B7/7915FA-39D3-44F7-AFEC-79DB1D964CD6
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Stream의 핵심 개념과 특징

Java 8에 추가된 스트림(Stream) API는 다량의 데이터 처리(순차 혹은 병렬적) 작업을 지원하도록 도입되었는데, 스트림이 제공하는 추상 개념 중 핵심은 두 가지다.

- 데이터 원소의 유한 or 무한 시퀀스(sequence)를 뜻한다.

```java
// 유한 시퀀스
List<String> list = Arrays.asList("Apple", "Banana", "Cherry");
Stream<String> stream = list.stream();
stream.forEach(System.out::println); // 각 요소를 출력

// 무한 시퀀스
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 1);
infiniteStream.limit(10).forEach(System.out::println); // 처음 10개 요소를 출력
```

유한 시퀀스는 한정된 수의 요소를 가진 스트림으로 예시의 경우 리스트의 요소 갯수만큼 요소를 가지므로 유한하다.

반면 무한 시퀀스는 끝이 정해져있지 않은 스트림으로 예시의 경우 각 요소는 이전 요소에 1을 더한 값으로 limit() 메서드로 스트림을 제한하고 있다.

- 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

```java
List<String> words = Arrays.asList("Apple", "Banana", "Cherry", "Date");

long count = words.stream()           // 스트림 소스
                  .filter(word -> word.length() > 5)  // 중간 연산
                  .count();            // 종단 연산
```

words는 스트림의 소스로 stream() 메서드로 리스트를 스트림으로 변환하는 과정이다.

중간 연산인 filter는 길이가 5자 이상인 단어들만 필터링하는데, 스트림을 변형하지만 연산 자체로는 스트림이 소비되지 않는다. 각 중간 연산은 연쇄적으로 연결될 수 있으며 이전 연산의 결과를 다음 단계로 전달하게 된다.

종단 연산인 count는 필터링된 단어들의 수를 카운트하게 된다. 스트림의 처리를 마무리하고 결과를 반환하는 역할을 하는 것이다. 

이처럼 스트림 파이프라인은 데이터 처리의 각 단계를 명확하게 표현하며, 데이터 소스로부터 원하는 결과까지 연속적인 처리 과정을 정의한다.

여기서 알아볼 수 있는 스트림 파이프라인의 특징은 지연 평가(lazy evaluation)된다는 것이다.

평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가는 무한 스트림을 다룰 수 있게 해준다. 

종단 연산이 없다면? 스트림 파이프라인은 아무 일도 하지 않는 명령어가 될 뿐이다. 

스트림은 이처럼 메서드 연쇄를 지원하는 플루언트 API(Fluent API)[^1]로 파이프라인 하나를 구성하는 모든 호출을 연결해 단 하나의 표현식으로 완성할 수 있다.

### 추가적인 특징

- 스트림 파이프라인은 순차적으로 수행된다. 
  
이를 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 되지만, 효과를 볼 수 있는 상황은 많지 않다.

- 어떠한 계산도 가능하다.

어떠한 계산도 가능하지만, 해야한다는 의미는 아니다. 스트림의 제대로 사용한다면 간결해지지만, 읽기 어렵고 유지보수가 힘들어 질 수 있다.

스트림을 언제 써야하는지를 규정하는 규칙은 없지만, 참고할 만한 몇 가지 지침은 있다. item45는 이에 대해 설명한다.

```java
// 스트림을 과하게 사용한 예
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> word.chars().sorted()
                    .collect(StringBuilder::new,
                        (sb, c) -> sb.append((char) c),
                        StringBuilder::append).toString()))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .map(group -> group.size() + ": " + group)
                .forEach(System.out::println);
        }
    }
}
```

코드는 사전 파일에서 단어를 읽어 사용자가 지정한 문턱값보다 원소 수가 많은 애너그램 그룹을 출력하는 기능을 한다. 

사전 파일을 여는 부분을 제외하면, 프로그램 전체가 단 하나의 표현식으로 처리되어 간결하지만, 읽기 어렵다.

```java
// 스트림을 적절하게 사용한 예
public class Anagrams {
    public static void main(String[] args) {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }
}
```

try-with-resource 블록에서 사전 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻도록 수정했다. 

스트림의 변수를 words로 명명하여 스트림 안의 원소가 단어임을 명확히 했으며, 이 때 스트림에서는 중간 연산없이 종단 연산에서 모든 단어를 수집하여 맵으로 모았다. 단어를 수집한 맵은 애너그램끼리 묶어놓은 것으로 스트림을 과하게 사용한 예에서 생성한 맵과 실질적으로 같다. 

### 주의점

- 람다 매개변수 네이밍

람다의 특성상 타입 이름을 자주 생략하므로, 매개변수의 이름을 잘 지어야만 스트림 파이프라인의 가독성이 유지된다.

- 기본 타입인 char용 스트림을 지원하지 않는다.

```java
"Hello world!".chars().forEach(System.out::print); // 721011081081113211911111410810033
```

스트림은 char용 스트림을 따로 지원하지 않는다. 위 코드에서 Hello world!를 출력할 것이라고 기대했지만, 숫자가 출력되었다.

이유는 .chars()가 반환하는 스트림의 원소는 char가 아닌 int 값이기 때문이다. 따라서 정숫값을 출력하는 print 메서드가 호출된 것이다.

올바른 print 메서드를 호출하려면 

```java
"Hello world!".chars().forEach(x -> System.out.print((char) x)); 
```

위와 같이 형변환을 명시적으로 해줘야 한다. 하지만 char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.

이처럼 스트림의 특징을 인지하고, 기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 나아 보일때만 반영하자.

## 람다와 코드 블록 비교

스트림 파이프라인은 되풀이 되는 계산을 함수 객체(람다 혹은 메서드 참조)로 표현하는 반면 반복코드에서는 코드 블록을 사용한다. 

이는 함수 객체로는 할 수 없지만, 코드 블록으로는 할 수 있는 일들이 있기 때문이다. 각각 특징을 살펴보자.

- 지역변수 수정 가능 여부

```java
// 코드 블록
int x = 10;
int y = 20;
{
    int z = 30;
    System.out.println(x + y + z); // 60
}
// z는 이 지점에서 접근 불가능

// 람다
int x = 10;
int y = 20;
Runnable r = () -> {
    // int z = 30; // 람다 내에서 새로운 변수를 선언할 수 있음
    System.out.println(x + y); // 30
    // x++; // 컴파일 에러
};
// r.run(); // 람다 표현식 실행
```

일반 코드블록은 유효한 스코프 내에서는 블록 내에서 값을 변경할 수 있다.

반면 람다 블록의 경우, 주변 스코프의 변수(x, y)에 접근할 수있지만, final(초기화 후에 값을 변경하지 않는 경우) 혹은 사실상 final인 변수에만 접근할 수 있다.

또한 람다 표현식 내에서 외부 변수를 수정하는 것은 허용되지 않으며, 정의된 후에 즉시 실행되지 않고, 명시적으로 호출(r.run)해야 실행이 된다.

- return 문 여부

코드 블록의 경우 return문을 사용해 메서드에서 탈출하거나, break, continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 건너뛸 수 있으며, 메서드 선언에 명시된 검사 예외를 던질 수 있다. 하지만 람다는 어떤 것도 할 수 없다.

### 스트림이 적합한 경우

위 특징을 종합하여 스트림에 적합한 경우를 정리하면 다음과 같다.

- 원소들의 시퀀스를 일관되게 변환하는 경우
- 원소들의 시퀀스를 필터링하는 경우
- 원소들의 시퀀스를 하나의 연산을 사용해 결합하는 경우(더하기, 연결, 최솟값 구하기 등..)
- 컬렉션에 모으는 경우
- 특정 조건을 만족하는 원소를 찾는 경우

### 스트림이 처리하기 어려운 경우

반면 어려운 경우를 정리하면 다음과 같다.

- 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때, 각 단계에서 값들에 동시에 접근하기 어려운 경우

```java
List<Pair<String, Integer>> data = Arrays.asList(
    new Pair<>("Apple", 3),
    new Pair<>("Banana", 5),
    new Pair<>("Cherry", 2)
);

data.stream()
    .map(pair -> pair.getValue() * 2) // 각 과일의 수량을 2배로 증가
    .forEach(System.out::println); // 최종 결과 출력
```

예시 코드는 과일의 이름과 수량을 나타내는 Pair 객체들의 리스트를 스트림으로 처리하는 경우로 map 연산을 통해 과일 수량을 2배로 증가시키는 로직을 적용했다.

여기서 문제되는 경우는 map 연산을 수행한 뒤에, 원본 객체 Pair에 대한 정보는 더 이상 스트림 파이프라인에 접근할 수 없다는 것이다. 이는 일단 한 값을 다른 값에 매핑한 뒤에 원래의 값을 잃는 특성때문이다.

```java
data.stream()
    .map(pair -> new Pair<>(pair.getKey(), pair.getValue() * 2))
    .forEach(pair -> System.out.println(pair.getKey() + ": " + pair.getValue()));
```

위와 같이 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법도 있지만, 스트림을 사용하는 주목적인 간결함에서 벗어나 만족스러운 해법은 아니다.

- 매핑 객체가 필요한 단계가 여러 곳인 경우

```java
List<String> words = Arrays.asList("Apple", "Banana", "Cherry");

Map<String, Integer> wordLengths = words.stream()
    .collect(Collectors.toMap(Function.identity(), String::length));

words.stream()
    .map(word -> word + ": " + wordLengths.get(word))
    .forEach(System.out::println);
```

단어 리스트를 스트림으로 처리해 그 길이를 매핑하는 맵을 생성하는 예시이다.

이후에, 다른 스트림에서 이 map을 사용하여 각 단어와 그 길이를 출력하고 있는데, 여기서 문제 상황이 발생한다.

스트림의 특성 상 데이터를 순차적으로 처리하기 때문에, 각 단계에서의 작업은 이전 단계의 결과에 의존하게 된다. 때문에 스트림 중간 단계에서 생성된 중간 결과물에 여러 단계에서 접근해야 하는 경우 처리가 복잡해질 수 있다.

때문에 전통적인 반복문(for, while)로 접근하거나, 스트림의 각 단계에서 필요한 정보를 포함한 중간 데이터 구조를 생성하고 데이터를 전달하는 방법을 사용할 수 밖에 없다.

## 정리

스트림으로 바꾸는게 가능하더라도 유지보수 측면에서 손해볼 수 있다. 

스트림과 반복 중에 어느 쪽이 더 나은지 확신하기 어렵다면 둘 다 구현해보고 더 나은쪽을 택하자.

## 참고 자료

- 이펙티브자바 3판
- https://ko.wikipedia.org/wiki/%ED%94%8C%EB%A3%A8%EC%96%B8%ED%8A%B8_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4 - 플루언트 인터페이스 설명

## 주석

[^1]: 플루언트 API(Fluent API)는 소프트웨어 개발에서 사용되는 설계 패턴의 일종으로, 코드를 더 읽기 쉽고 자연스럽게 작성할 수 있도록 하는 방식이다. 이 용어는 특히 메소드 체이닝(method chaining)과 관련이 깊은데, 플루언트 API를 사용하면, 여러 메소드 호출을 연쇄적으로 연결하여, 마치 자연 언어와 비슷한 방식으로 코드를 작성할 수 있다.
