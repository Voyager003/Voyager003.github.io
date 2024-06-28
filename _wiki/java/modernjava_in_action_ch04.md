---
layout  : wiki
title   : 스트림 소개
summary : 
date    : 2024-06-28 20:28:57 +0900
updated : 2024-06-28 20:29:58 +0900
tag     : java
resource: C7/3708DE-3494-4B1D-B2C1-892940762D83
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 스트림 소개

### 스트림(stream)이란?

`스트림`은 JDK 1.8에 추가된 기능으로 선언형으로 컬렉션(Collection) 데이터를 처리할 수 있는 기능이다. 람다와 마찬가지로 함수형 프로그래밍에 기초한 패러다임으로 계산을 일련의 변환으로 재구성하는 것이 스트림 패러다임의 핵심이다.

먼저 코드로 간단하게 살펴보자.

```java
List<Dish> lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu) {
	if(dish.getCalories() < 400) {
		lowCaloricDishes.add(dish);
	}
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    public int compare(Dish d1, Dish d2) { 
        return Integer.compare(d1.getCalories(), d2.getCalories());
    }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName());
}
```

예시 코드는 400칼로리를 넘지 않는 요리명을 반환하고, 칼로리를 기준으로 요리를 정렬하는 코드이다.
이 때, 코드의 lowCaloricDishes 변수는 컨테이너 역할만 하는 중간 변수로 '가비지 변수'라 한다. 이 코드를 스트림을 이용해 개선해보자.

```java
List<String> lowCaloricDishesName =
	menu.stream()
		.filter(d -> d.getCalories() < 400) // 400칼로리이하로 필터링
		.sorted(comparing(Dish::getCalories)) // 정렬
		.map(dish::getName) // 추출
		.collect(toList()); // 리스트에 저장
```

스트림 사용 이전에는 if문과 for 루프문 등 제어블록을 사용해 동작을 지정하고 직접 컬렉션에 자료를 추가했지만, 스트림을 사용한 코드는 동작 파라미터화, 람다 표현식을 조합하여 같은 동작을 한 줄로 쉽게 구현할 수 있는 것을 볼 수 있다.

또한 filter, sorted, map, collet 등 여러 연산을 파이프라인으로 연결하면서 가독성과 명확성을 유지한채 원하는 데이터를 뽑아낼 수 있다. 이 후에 알아보겠지만, 데이터 처리 과정을 병렬화하면서 스레드와 lock을 걱정할 필요없다.

#### 스트림의 정의

스트림은 '데이터 처리 연산'을 지원하도록 '소스'에서 추출된 '연속된 요소(sequence of elements)'로 정의할 수 있다. 정의를 하나씩 살펴보자.

'데이터 처리 연산'은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 DB와 비슷한 연산(SQL 쿼리)을 지원한다. 이 연산은 순차적 혹은 병렬로 실행할 수 있다.

'소스'는 스트림이 소비하는 컬렉션, 배열, I/O 자원 등 데이터 제공 소스를 의미한다. 소스를 소비한 컬렉션은 정렬된 컬렉션으로 스트림 생성 시 이 정렬이 그대로 유지된다.

컬렉션과 마찬가지로 스트림은 는 특정 요소 형식으로 이뤄진 '연속된 값 집합'의 인터페이스를 제공한다. '컬렉션'이 시간 및 공간 복잡성과 관련된 요소를 저장하거나 접근 연산이 주를 이룬다면, '스트림'은 표현 계산식이 주를 이룬다. 즉, 스트림의 주제는 '계산'이다.

```java
List<String> lowCaloricDishesName =
	menu.stream()
		.filter(d -> d.getCalories() < 400)
		.sorted(comparing(Dish::getCalories))
		.map(dish::getName)
		.collect(toList());
```

<img width="665" alt="SCR-20240627-rshk" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/55fe842d-4f95-4858-a3e1-d0afcffba69a">



또한 스트림 연산끼리 연결하여 파이프라인을 구성할 수 있도록 스트림 자신을 반환할 수 있다.

파이프라인은 소스(요리 리스트)에 적용하는 질의이며, collect를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환한다.

### 특징

Java의 컬렉션과 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다는 공통점을 가진다. 반면 차이점은 데이터를 '언제' 계산하느냐이다.

#### 계산 시점
```java
public class CollectionExample {
    public static void main(String[] args) {
        // 컬렉션에 요소를 추가
        List<Integer> numbers = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            numbers.add(i);
        }

        // 컬렉션에서 요소를 필터링
        List<Integer> evenNumbers = new ArrayList<>();
        for (Integer number : numbers) {
            if (number % 2 == 0) {
                evenNumbers.add(number);
            }
        }
    }
}
```

컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조이기 때문에 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다.

위 코드에서 numbers 리스트는 미리 모든 숫자를 저장하고, 필터링할 때도 모든 요소가 이미 메모리에 로드된 상태이다.

```java
public class StreamExample {
    public static void main(String[] args) {
        // 스트림 생성
        List<Integer> evenNumbers = 
        IntStream.range(0, 10) // 0부터 9까지의 숫자 스트림 생성
                .filter(number -> number % 2 == 0) // 짝수 필터링
	            .boxed() // Integer 스트림으로 변환                              .collect(Collectors.toList()); // 결과를 리스트로 수집
    }
}
```

반면 스트림은 이론적으로 요청 시에만 요소를 계산하는 고정된 자료구조이다. 사용자가 요청하는 값만 스트림에서 추출하는 것이 핵심이다.

위 코드에서 스트림은 range() 메서드로 생성되며, 필터링은 요청 시에만 수행되며, 스트림의 요소들은 collect 메서드를 호출할 때까지 실제로 계산되지 않는다.

#### 스트림 API는 일회용이다.

반복자와 같이 스트림은 한 번만 탐색 가능하다. 탐색된 스트림 요소는 즉시 소비되며, 한 번 탐색한 요소를 탐색하려면 초기 데이터 소스에서 새 스트림을 만들어야 한다.

```java
List<String> title = Arrays.asList("Java8", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println); // title의 단어 출력
s.forEach(System.out::println); // 에러 발생

// Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
```

실제로 이미 소비된 스트림을 다시 출력하려고 한다면, 스트림이 이미 조작되었거나 닫혔다는 에러를 확인할 수 있다.

#### 원본의 데이터를 변경하지 않는다.

```java
public class StreamImmutabilityExample {
    public static void main(String[] args) {
        // 원본 데이터 리스트
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Edward");

        // 스트림을 사용하여 길이가 4보다 큰 이름만 필터링
        List<String> filteredNames = names.stream()
                                          .filter(name -> name.length() > 4)
                                          .collect(Collectors.toList());

        // 필터링 결과 출력
        System.out.println("Filtered names: " + filteredNames);

        // 원본 데이터 출력
        System.out.println("Original names: " + names);
    }
}

// Filtered names: [Alice, Charlie, David, Edward]
// Original names: [Alice, Bob, Charlie, David, Edward]
```

스트림은 원본의 데이터를 조회하여 원본의 데이터가 아닌 별도의 요소들로 스트림을 생성한다.

원본의 데이터로부터 읽기만 할 뿐이며, 정렬 및 필터링 등의 작업은 별도의 스트림 요소에서 처리되는 것이다.

#### 외부 반복과 내부 반복

```java
// 컬렉션의 경우
List<String> names = new ArrayList<>();
for(Dish dish : menu) {
	names.add(dish.getName()); // 직접 요소를 추가
}
```

컬렉션 인터페이스에서는 for-each를 사용해 사용자가 직접 요소를 반복해야 해야한다. 이를 외부 반복(external iteration)이라 한다.

```java
// 스트림의 경우
List<String> names = menu.stream()
						 .map(Dish::getName)
						 .collect(toList());
```

반면 스트림은 반복을 알아서 처리하고, 처리한 결과 스트림값을 어딘가 저장하는 내부 반복(internal iteration)을 사용한다. 함수에 어떤 작업을 수행할 지 지정하면 모든 것이 처리된다.

내부 반복이 좋은 이유는 작업을 투명하게 병렬로 처리하거나 다양한 순서로 처리할 수 있다는 점이다. 컬렉션에서는 달성하기 힘든 조건이다.

### 스트림 연산

스트림 인터페이스(java.util.stream.Stream)는 많은 연산을 정의한다. 이 연산은 크게 두 가지로 구분할 수 있는데, `중간 연산(intermediate operation)`과 `최종 연산(terminal operation)`으로 구분된다.

#### 중간 연산

```java
List<String> names = menu.stream().
					.filter(dish -> dish.getCalories() > 300)
					.map(Dish::getName)
					.limit(3)
					.collect(toList());
```

코드에서 중간 연산은 filter, map, limit에 해당한다.
이 중간 연산은 단말 연산을 스트림 파이프라인에 실행하기 전까지 아무 연산도 수행하지 않는다. 이는 중간 연산을 합친 뒤 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하기 때문이다.

스트림 파이프라인에서 무슨 일이 일어나는지 콘솔에 출력해보자.

```java
List<String> names  =
	menu.stream()
	.filter(dish -> {
		System.out.println("filtering:" + dish.getName());
		return dish.getCalories() > 300;
	})
	.map(dish -> {
		System.out.println("mapping:" + dish.getName());
		return dish.getName();
	})
	.limit(3)
	.collect(toList());

System.out.println("terminal op:" + names);

// filtering:pork
// mapping:pork
// filtering:beef
// mapping:beef
// filtering:chicken
// mapping:chicken
// terminal op: [pork, beef, chicken]
```
최종 연산인 collect를 호출하기 전에는 menu들이 연산되지 않은 상태로 남아있는 것을 확인할 수 있다.

#### 최종 연산

최종 연산은 스트림 파이프라인에서 결과를 도출한다. 위 코드에서는
 ```java
 .collect(toList());
 ```
collect가 이에 해당한다.

보통은 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다. 코드로 살펴보자.

```java
menu.stream()
	.forEach(System.out::printlnm);
```

스트림 파이프라인에서 forEach는 소스의 각 menu에 람다를 반환하는 최종 연산이다. 이는 menu에서 만든 스트림의 모든 요소를 출력하게 된다.

forEach외에도 스트림의 요소 개수를 반환하는 count나 스트림을 reduce하여 List, Map, Integer 형식 등 컬렉션을 만드는 collect가 이 최종 연산에 해당한다.

### 스트림 사용처

#### 스트림이 적합한 경우

위 특징을 종합하여 스트림에 적합한 경우를 정리하면 다음과 같다.

- 원소들의 시퀀스를 일관되게 변환하는 경우
- 원소들의 시퀀스를 필터링하는 경우
- 원소들의 시퀀스를 하나의 연산을 사용해 결합하는 경우(더하기, 연결, 최솟값 구하기 등..)
- 컬렉션에 모으는 경우
- 특정 조건을 만족하는 원소를 찾는 경우

#### 스트림이 처리하기 어려운 경우

##### 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때, 각 단계에서 값들에 동시에 접근하기 어려운 경우

```
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
```
data.stream()
    .map(pair -> new Pair<>(pair.getKey(), pair.getValue() * 2))
    .forEach(pair -> System.out.println(pair.getKey() + ": " + pair.getValue()));
```

위와 같이 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법도 있지만, 스트림을 사용하는 주목적인 간결함에서 벗어나 만족스러운 해법은 아니다.

##### 매핑 객체가 필요한 단계가 여러 곳인 경우

```
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

## 참고자료
- 모던 자바 인 액션
- 이펙티브 자바 3판 item45 - 스트림은 주의해서 사용하라
