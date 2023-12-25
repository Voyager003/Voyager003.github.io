---
layout  : wiki
title   : 스트림에서는 부작용 없는 함수를 사용하라 
summary : 
date    : 2023-12-25 11:35:17 +0900
updated : 2023-12-25 14:30:39 +0900
tag     : java effectivejava
resource: 91/06B59F-D597-461C-B4B3-047AB32A048E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 스트림 패러다임의 핵심

스트림은 함수평 프로그래밍에 기초한 패러다임이다. 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 

각 변환 단계는 가능한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 여기서 순수 함수는 오직 입력만이 결과에 영향을 주는 함수를 말한다. 

또한 다른 가변 상태를 참조하지 않고, 함수 스스로 다른 상태를 변경하지 않는다. 순수함수로 만드려면 중간 단계나 종단 단계에서 스트림 연산에 건네는 함수는 모두 부작용(side effect)이 없어야 한다. 

### 예시로 살펴보기

```java
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
```

이 코드는 텍스트 파일에서 단어별 수를 세서 빈도표를 만드는 일을 한다.

종단 연산인 forEach에서 외부 상태를 수정하는 람다(freq)를 실행하면서 객체의 상태를 변화시키게되는데, 이는 순수 함수의 패러다임에 어긋나는 코드가 되는 것이다.

```java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

forEach 반복 연산을 제거하여 간결하고 명확한 코드로 만들었다.

종단 연산인 forEach는 대놓고 반복적이어서 병렬화할 수 없어 스트림답지 못한 기능이다. 따라서 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 사용하지 말도록 주의하자.

## Collectors 클래스

Collectors 클래스는 스트림의 원소를 쉽게 컬렉션으로 종합하고 스트림 종료를 할 수 있다. 이를 이용하여 빈도(freq)를 계산하는 코드를 개선해보자.

### toList()

```java
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10)
    .collect(toList());
```

sorted에 넘긴 비교자 comparing은 키 추출 함수를 받는 비교자 생성 메서드로 freq:get으로 입력받은 단어를 빈도표에서 찾아 그 빈도를 반환한다. 이어서 가장 흔한 단어가 위로 오도록 비교자를 역순으로 정렬하고 toList()로 리스트에 담는다.

toList는 Collectors의 메서드로 COllectors의 멤버를 정적 임포트하여 스트림 파이프라인의 가독성을 높일 수 있다.

### toMap()

```java
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(
        toMap(Object::toString, e -> e));
```

toMap은 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.(keyMapper, valueMapper)

예시의 경우 Operation 타입의 열거형(enum)을 문자열 키와 매핑하여 Map으로 변환하는 과정을 보여준다.

먼저 Operation 열거형의 모든 값을 스트림으로 변환하고, Object::toString 메소드 참조는 각 Operation 객체를 그 객체의 toString 메소드로 변환한 문자열로 매핑을 한다.

collect(toMap(...))는 스트림의 각 원소를 지정된 키와 값으로 매핑하여 Map으로 수집(collect)한다.

이처럼 toMap()은 각 원소가 고유한 키에 매핑되어 있을 때 적합하다. 원소 다수가 같은 키를 사용한다면 IllegalStateException 예외를 던지고 종료시킬 것이다.

### groupingBy()

groupingBy()는 입력으로 분류 함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아놓은 map을 담은 collector를 반환한다. 

```java
words.collect(groupingBy(word -> alphabetize(word)));
```

알파벳화한 단어를 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성했다. 스트림 collector의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 일이다. 가장 간단한 방법은 toSet()을 넘기는 것이다.

- toSet()

```java
Map<YourKeyType, Set<YourValueType>> mapWithSet = items.stream()
    .collect(Collectors.groupingBy(YourKeyType::getKey, Collectors.toSet()));
```

결과적으로 리스트가 아닌 Set을 값으로 가지는 맵을 만들 수 있다.

다운스트림[^1] collector로 counting()을 건네는 방법을 사용해 각 카테고리를 해당 카테고리에 속하는 원소의 갯수와 매핑한 map을 얻을 수 있다.

```java
Map<String, Long> freq = words.collect(groupingBy(String::toLowerCase, counting()));
```

### partitionBy

partitioningBy는 Java의 스트림 API에서 제공하는 메소드로, 스트림을 두 부분으로 분할하는데 사용되며, 분류 함수 자리에 predicate를 받아 키가 Boolean인 맵을 반환한다. 

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class PartitionByExample {
    public static void main(String[] args) {
    
        // 예시 객체 리스트
        List<YourObject> objects = List.of(/* 객체들 */);

        // Predicate를 사용하여 객체를 분류
        Map<Boolean, List<YourObject>> partitioned = objects.stream()
            .collect(Collectors.partitioningBy(obj -> obj.isSomeCondition()));

        // 조건을 만족하는 객체들
        List<YourObject> truePartition = partitioned.get(true);

        // 조건을 만족하지 않는 객체들
        List<YourObject> falsePartition = partitioned.get(false);

        // 결과 출력
        System.out.println("Objects that satisfy the condition: " + truePartition);
        System.out.println("Objects that do not satisfy the condition: " + falsePartition);
    }
}
```

YourObject는 분류하고자 하는 객체 타입이며, isSomeCondition 메소드는 YourObject 객체가 특정 조건을 만족하는지 여부를 반환하는 메소드이다. partitioningBy는 이 조건에 따라 객체들을 두 그룹으로 나누고, 이를 true와 false 키를 가진 맵으로 반환한다.

### minBy, maxBy

메소드들은 Comparator 인터페이스를 인자로 받아, 주어진 비교 규칙에 따라 스트림의 원소들을 비교한다.

```java
import java.util.Comparator;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class MinMaxByExample {
    public static void main(String[] args) {
        List<YourObject> objects = List.of(/* 객체들 */);

        // 최소값 찾기
        Optional<YourObject> minObject = objects.stream()
            .collect(Collectors.minBy(Comparator.comparing(YourObject::getValue)));

        minObject.ifPresent(obj -> System.out.println("Minimum value object: " + obj));

        // 최대값 찾기
        Optional<YourObject> maxObject = objects.stream()
            .collect(Collectors.maxBy(Comparator.comparing(YourObject::getValue)));

        maxObject.ifPresent(obj -> System.out.println("Maximum value object: " + obj));
    }
}
```

YourObject는 비교하고자 하는 객체 타입이고, getValue 메소드는 비교에 사용될 값을 반환한다. minBy와 maxBy는 Comparator를 사용하여 이 값을 기준으로 객체들을 비교한다.

minBy는 가장 작은 값을 가진 객체를, maxBy는 가장 큰 값을 가진 객체를 찾는다.

결과적으로 Optional<YourObject> 타입으로 반환되며 이는 스트림이 비어 있을 경우 최소값이나 최대값이 존재하지 않을 수 있기 때문이다. ifPresent 메소드를 사용하여 값이 존재하는 경우에만 처리를 수행할 수 있다.

### joining

joining은 Java의 Collectors 클래스에 있는 메서드로, CharSequence 인스턴스(예: 문자열)의 스트림을 하나의 문자열로 결합할 때 사용한다. 스트림의 모든 원소를 하나의 문자열로 연결하며, 선택적으로 구분자(delimiter), 접두사(prefix), 접미사(suffix)를 추가할 수 있다.

```java
import java.util.stream.Stream;
import java.util.stream.Collectors;

public class JoiningExample {
    public static void main(String[] args) {
        Stream<String> stringStream = Stream.of("Java", "Python", "C++", "JavaScript");

        // 단순 결합
        String joinedString = stringStream.collect(Collectors.joining());
        System.out.println("Joined without delimiter: " + joinedString);

        // 구분자를 사용한 결합
        stringStream = Stream.of("Java", "Python", "C++", "JavaScript");
        String joinedWithDelimiter = stringStream.collect(Collectors.joining(", "));
        System.out.println("Joined with delimiter: " + joinedWithDelimiter);

        // 구분자, 접두사, 접미사를 사용한 결합
        stringStream = Stream.of("Java", "Python", "C++", "JavaScript");
        String joinedWithDelimiterAndPrefixSuffix = stringStream.collect(Collectors.joining(", ", "[", "]"));
        System.out.println("Joined with delimiter, prefix, and suffix: " + joinedWithDelimiterAndPrefixSuffix);
    }
}
```

위 코드에서는 세 가지 방법으로 joining을 사용한다.

단순 결합: 원소들을 그대로 연결하는 경우(예: "JavaPythonC++JavaScript".)

구분자를 사용한 결합하는 경우, 원소들 사이에 구분자를 추가합니다. (예: "Java, Python, C++, JavaScript".)

구분자, 접두사, 접미사를 사용한 결합한 경우 원소들 사이에 구분자를 추가하고, 전체 문자열의 시작과 끝에 접두사와 접미사를 추가한다. (예: 예: "[Java, Python, C++, JavaScript]".)

## 참고자료

- 이펙티브자바 3판

## 주석

[^1]: 다운스트림(downstream)은 자바 스트림 API에서 사용되는 용어로, 특히 collect 메소드와 함께 사용되는 Collector 인터페이스의 컨텍스트에서 주로 나타난다. 스트림의 각 원소에 대한 추가적인 처리 단계를 의미한다. 즉, 스트림의 데이터 처리 파이프라인에서 한 단계 더 진행된 처리를 가리킨다. 예를 들어, groupingBy 메소드를 사용할 때, 첫 번째 인자는 그룹화의 기준이 되는 키를 생성하는 함수이고, 선택적인 두 번째 인자인 다운스트림 Collector는 이 그룹화된 원소들에 대해 추가적인 수집 작업을 정의한다. (by ChatGPT)
