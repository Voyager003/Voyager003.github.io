---
layout  : wiki
title   : 스트림 활용 
summary : 
date    : 2024-07-05 20:18:46 +0900
updated : 2024-07-05 20:19:44 +0900
tag     : java
resource: 61/91E502-5108-4889-92F9-A97AB16D7A6B
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 스트림 활용

### 필터링(Filtering)
#### 프레디케이트(Predicate) 필터링
```java
Stream<T> filter(Predicate<? super T> predicate)

// 주어진 predicate와 일치하는 이 스트림의 요소로 구성된 스트림을 반환한다.  
// 이것은 중간 연산이다.
```
boolean을 반환하는 함수인 프레디케이트를 인수로 받아 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.

예시는 모든 채식 요리를 필터링하는 Dish 리스트를 반환하는 코드이다.

```java
List<Dish> vegeMenu = menu.stream()
						  .filter(Dish::isVegetrian)
						  .collect(toList());
```

#### 고유 요소 필터링(distinct)
```java
Stream<T> distinct()

// 이 스트림의 고유 엘리먼트(Object.equals(Object)에 따라)로 구성된 스트림을 반환한다.  
// 정렬된 스트림의 경우, 고유 요소의 선택이 안정적이다(중복된 요소의 경우, 만남 순서에서 먼저 나타나는 요소가 보존됨). 정렬되지 않은 스트림의 경우 안정성이 보장되지 않는다.  
// 이것은 상태 저장 중간 연산이다.
```
distinct는 중복된 요소를 제거하고 고유 요소(distinct element)로 이뤄진 스트림을 반환하는 메서드로, 고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정된다.

예시는 List의 모든 짝수를 선택하고 중복을 필터링하는 코드이다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
	   .filter(i -> i%2==0)
	   .distinct()
	   .forEach(System.out::println); // 2 4
```

### 스트림 슬라이싱
#### 프레디케이트 슬라이싱
##### takeWhile
```java
default Stream<T> takeWhile​(Predicate<? super T> predicate)
```
Java 9에 추가된 메서드로 제네릭 프레디케이트를 인자로 받아 Predicate가 false를 반환할 때까지의 요소를 취한다.

filter 메서드와 유사해보이지만, filter() 메서드가 전체 스트림에 대한 `Predicate`를 판단한다면, takeWhile은 `Predicate`가 **false**를 반환하는 순간 나머지 요소를 전부 버린다는 차이가 있다.

이는 아주 많은 요소를 가진 스트림에서는 이런 부분이 큰 차이를 유발할 수 있으며, 특히 정렬되어 있는 요소에 대해서 큰 장점을 가진다.

다음과 같은 요리 목록을 가지는 List가 있다고 가정해보자.

```java
List<Dish> specialMenu = Arrays.asList(
	new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
	new Dish("prawns", false, 300, Didsh.Type.FISH),
	new Dish("rice", true, 350, Dish.Type.OTHER),
	new Dish("chicken", false, 400, Dish.Type.MEAT),
	new Dish("french fries", true, 530, Dish.Type.OTHER);
)
```
여기서 320 칼로리 이하의 선택하려면 filter를 사용할수도 있겠지만, 320 칼로리보다 크거나 같은 요리가 나왔을 경우 중단하고 싶다면?

```java
List<Dish> slicedMenu1 = menu.stream()
							 .takeWhile(dish -> dish.getCalories() < 320)
							 .collect(toList());
```

takeWhile을 사용하면 반복 작업을 중단할 수 있다. 이는 많은 요소를 포함하는 큰 스트림에서 큰 차이를 보인다.
##### dropWhile
```java
default Stream<T> dropWhile​(Predicate<? super T> predicate)
```
Java 9에 추가된 메서드로  제네릭 프레디케이트를 파라미터로 받아서 해당 프레디케이트가 **false**를 반환할 때까지의 요소를 모두 버리고 나머지 요소를 반환한다.

takeWhile과는 정반대로 작업을 수행하며 프레디케이트가 **false**가 되는 지점에서 작업을 중단하고 남은 모든 요소를 반환한다.

다음 예시는 320칼로리보다 큰 요소를 탐색하는 코드이다.

```java
List<Dish> slicedMenu2 = specialMenu.stream()
									.dropWhile(dish - > dish.getCaloiries() < 320)
									.collect(toList()); 

// rice, cheicken, french fries
```

#### 스트림 축소
```java
Stream<T> limit​(long maxSize)

// 이 스트림의 요소로 구성된 스트림을 반환하며, 길이가 maxSize보다 길지 않도록 잘라낸다.  
// 이것은 short circuiting stateful 중간 연산이다.
```
주어진 값 이하의 크기를 갖는 새로운 스트림을 반환한다.

예시는 300칼로리 이상의 세 요리를 선택하여 리스트를 만드는 스트림이다.

```java
List<Dish> dishes = specialMenu.stream()
							   .filter(dish -> dish.getCalories() > 300)
							   .limit(3) // 3개로 제한
							   .collect(toList()); 

// rice, chicken, french fries
```
#### 요소 건너뛰기
```java
Stream<T> skip​(long n)

// 스트림의 처음 n개의 요소를 버린 후 이 스트림의 나머지 요소로 구성된 스트림을 반환한다. 이 스트림에 n개보다 적은 수의 요소가 포함되어 있으면 빈 스트림이 반환된다.
// 이것은 상태 저장 중간 연산이다.
```

n개 이하의 요소를 포함하는 스트림에 skip|(n)을 호출하면 빈 스트림이 반환된다.

예시는 300칼로리 이상의 처음 두요리를 건너뛴 뒤 300칼로리가 넘는 나머지 요리를 반환한다.

```java
List<Dish> dishes = menu.stream()
						.filter(dish -> dish.getCalories() > 300)
						.skip(2) // 2
						.collect(toList()); 

// french fries
```
### 매핑(Mapping)
#### 스트림 각 요소에 함수 적용하기
```java
<R> Stream<R> map​(Function<? super T,? extends R> mapper)

// 이 스트림의 요소에 주어진 함수를 적용한 결과로 구성된 스트림을 반환한다.  
// 이것은 중간 연산이다.
```

함수를 인수로 받아 그 함수를 적용하는 메서드이다. 인수로 제공된 함수는 각 요소에 적용되며, 함수를 적용한 결과가 새로운 요소로 매핑된다.

예시는 Dish::getName을 map 메서드의 인수로 전달하여 스트림 요리명을 추출하는 코드이다.

```java
List<String> dishNames = menu.stream()
							 .(Dish::getName)
							 .collect(toList());


```
이 때, getName은 String을 반환하므로 map 메서드의 출력 스트림은 Stream < String > 형태를 가진다.

#### 스트림 평면화
map을 응용하여 List에서 고유 문자로 이뤄진 List를 반환하도록 해보자.

```java
String[] word = ["Hello", "World"];

words.stream()
	 .map(word -> word.split(""))
	 .distinct()
	 .collect(toList());

// H, e, l, o, W, r, d
```
이 때, map으로 전달한 람다는 각 단어의 String[], 즉 문자열 배열을 반환한다. 문자열 스트림을 반환하도록 Stream < String > 이 되도록하려면 어떻게 해야할까?

##### Arrays.stream
먼저 배열 스트림 대신 문자열 스트림이 필요하다.
```java
String[] arrayOfWords = {"Goodbye", "World"};
Stream<String> streamOfwords = Arrays.stream(arrayOfWords);

words.stream()
	 .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
	 .map(Arrays::stream) // 각 배열을 별도의 스트림으로 생성
	 .distinct()
	 .collect(toList());
```
하지만 이 코드도 스트림 리스트(List< Stream< String >> )이 만들어 지면서 문제가 해결되지 않았다.

##### flatMap
이를 해결하려면 각 단어를 개별 문자열로 이뤄진 배열로 만든 뒤, 각 배열을 별도의 스트림으로 만들어야 한다.

```java
List<String> uniqueCharacters = 
	words.stream()
	 .map(word -> word.split("")) // 각 단어를 개별 문자열 배열로 변환
	 .flatMap(Arrays::stream) // 생성 스트림을 하나의 스트림으로 평면화
	 .distinct()
	 .collect(toList());
```

flatMap은 각 배열을 스트림이 아닌 스트림의 컨텐츠로 매핑한다. 스트림의 각 값을 다른 스트림으로 만든 뒤 모든 스트림을 하나의 스트림으로 연결한 것이다.

### 검색과 매칭
#### anyMatch
```java
boolean anyMatch​(Predicate<? super T> predicate)

// 이 스트림의 요소가 제공된 술어와 일치하는지 여부를 반환한다. 결과 결정에 필요하지 않은 경우 모든 요소에 대해 술어를 평가하지 않을 수 있다. 스트림이 비어 있으면 거짓이 반환되고 술어는 평가되지 않는다.  
// 이것은 short-circuiting 연산이다.
```
프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는 지 확인할 때 사용한다.

예시는 menu에 채식메뉴가 있는지 확인하는 코드이다.

```java
if(menu.stream().anyMatch(Dish::isVegetarian)) {
	...
}
```

참고로 boolean을 반환하므로 최종 연산이다.

#### allMatch
```java
boolean allMatch​(Predicate<? super T> predicate)

// 이 스트림의 모든 요소가 제공된 술어와 일치하는지 여부를 반환한다. 결과 결정에 필요하지 않은 경우 모든 요소에 대해 술어를 평가하지 않을 수 있다. 스트림이 비어 있으면 true가 반환되고 술어는 평가되지 않는다.  
// 이것은 short-circuiting 연산이다.
```
allMatch는 스트림의 모든 요소가 주어진 프레디케이트와 일치하는지 검사한다.

예시는 메뉴가 1000칼로리 이하인지 확인하는 코드이다.

```java
boolean isHealthy = menu.stream()
						.allMatch(dish -> dish.getCalories < 1000);
```

#### noneMatch
```java
boolean noneMatch​(Predicate<? super T> predicate)

// 이 스트림의 요소가 제공된 술어와 일치하는 요소가 없는지 여부를 반환한다. 결과 결정에 필요하지 않은 경우 모든 요소에 대해 술어를 평가하지 않을 수 있다. 스트림이 비어 있으면 true가 반환되고 술어는 평가되지 않는다.
// 이것은 short-circuiting 연산이다.
```
allMatch와 반대 연산으로, 주어진 프레디케이트와 일치하는 요소가 없는지 확인한다.

예시는 위의 allMatch 코드를 noneMatch 메서드로 동일하게 작동시키는 코드이다.

```java
boolean isHealthy = menu.stream()
						.noneMatch(dish -> dish.getCalories >= 1000);
```

#### 요소 검색
##### findAny
```java
Optional<T> findAny​()

// 스트림의 일부 요소를 설명하는 선택사항을 반환하거나 스트림이 비어 있는 경우 빈 선택사항을 반환한다.  
// 이것은 short-circuiting 연산이다.
  
// 이 작업의 동작은 명시적으로 비결정적이며 스트림의 모든 요소를 자유롭게 선택할 수 있다. 이는 병렬 작업에서 성능을 극대화하기 위한 것으로, 동일한 소스에 대해 여러 번 호출하면 동일한 결과가 반환되지 않을 수 있다는 단점이 있다. (안정적인 결과를 원한다면 findFirst()를 대신 사용하라.)
```
findAny는 현재 스트림에서 임의 요소를 반환한다.
\
예시는 무작위 채식 메뉴를 선택하는 코드이다.

```java
Optional<Dish> dish = menu.stream()
						  .filter(Dish::isVegetarian)
						  .findAny();
```
##### findFirst
```java
Optional<T> findFirst​()

// 이 스트림의 첫 번째 요소를 설명하는 선택사항을 반환하거나 스트림이 비어 있는 경우 빈 선택사항을 반환한다. 스트림에 인카운터 순서가 없는 경우 아무 요소나 반환될 수 있다.  
// 이것은 short-circuiting 연산이다.
```
List나 정렬된 연속 데이터로부터 생성된 스트림처럼 일부 스트림에는 논리적 아이템 순서가 정해져있을 수 있다. 이러한 스트림에서 첫 번째 요소를 찾으려면? 코드로 알아보자.

예시는 숫자 List에서 3으로 나눠떨어지는 첫 번째 제곱값을 반환하는 코드이다.

```java
List<Integer> someNum = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree() =
	someNum.stream()
		   .map(n -> n * n)
		   .filter(n -> n % 3 == 0)
		   .findFirst(); // 9
```
참고로 병렬 실행에서는 첫 번째 요소를 찾기 어렵기 때문에, 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.
### 리듀싱(Reducing)
```java
T reduce​(T identity, BinaryOperator<T> accumulator)

// 제공된 ID 값과 연관 누적 함수를 사용하여 이 스트림의 요소에 대해 감소를 수행하고 감소된 값을 반환한다.
```

`리듀싱 연산`은 모든 스트림 요소를 처리하여 값으로 도출하는 것이다. 함수형 프로그래밍 용어로는 종이를 작은 조각이 될 때까지 접는 것과 비슷하다는 의미로 '폴드'라고 불리기도 한다.
#### 요소의 합 구하기
```java
// 기존의 for-each 루프 방법
int sum = 0;
for (int x : numbers) {
	sum += x;
}
```

for-each 루프를 이용하는 방법에서는 sum 변수의 초깃값인 0과 요소를 조합하는 연산 + 두 파라미터를 사용하여 List에서 하나의 숫자가 남을 때까지 reduce를 반복한다.

이를 reduce를 사용하여 나타낸다면?

```java
int sum = numbers.stream().reduce(0, (a, b)-> a + b);
```

reduce로 람다인 (a, b) -> a * b를 넘겨 모든 요소에 곱셈을 적용할 수 있다.

##### 초깃값을 받지 않는 reduce
```java
Optional<Integer> sum = numbers.stream().reduce((a, b)->(a+b));
```
초깃값을 받지 않도록 오버로드된 reduce는 Optional을 반환한다.

이유는 스트림을 아무 요소도 없는 상황이라고 가정한다면 초기값이 없어 reduce를 반환할 수 없다. 따라서 합계가 없음을 가리킬 수 있도록 Optional로 감싼 결과를 반환한다.

#### 최댓값 및 최솟값 구하기
```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```
스트림에서는 초깃값과 스트림의 두 요소를 하나의 값으로 만드는 람다를 인수로 받는다.

reduce 연산이 스트림의 모든 요소를 소비할 때까지 람다를 반복 수행하면서 최댓값과 최솟값을 만들어낸다.

#### reduce의 특징
기존에 단계적 반복으로 합계를 구하는 것과 reduce를 사용한 방법은 무슨 차이가 있을까?

##### 병렬화
먼저 reduce는 내부 반복이 추상화되어 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 이는 반복적인 합계에서는 sum 변수를 공유하기 때문에 병렬화가 어렵다.
여기서 강제적으로 동기화시키더라도 병렬화로 얻어야 할 이득이 스레드 간의 소모적 경쟁때문에 상쇄되어 버린다.
##### stateless/stateful operation(내부상태를 갖지않는 연산/내부상태를 갖는 연산)
map, filter 등 메서드는 입력 스트림에서 각 요소를 받아 0 혹은 결과를 출력 스트림으로 보낸다. 이는 사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 갖지 않는 가정하에 보통 내부 상태를 갖지 않는 연산이라 한다.

reduce, sum, max와 같은 연산은 결과를 누적할 내부 상태가 필요하다. 내부 상태는 위 예시에서 살펴본 int 혹은 double을 내부 상태로 사용했다. 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정(bounded)되어 있다.

반면 sorted, distinct 같은 연산은 스트림의 요소를 정렬, 중복 제거하려면 과거의 이력을 알아야 한다. 어떤 요소를 출력스트림으로 추가한다고 하면 모든 요소가 버퍼에 추가되어 있어야 한다는 것이다.
이 때 연산을 수행하는 데 필요한 저장소 크기는 정해져있지 않기 때문에 데이터 스트림의 크기가 크거나 무한이라면 문제가 생길수 있다. 이러한 연산을 stateful operation이라 한다.
### 숫자형 스트림
#### 기본형 특화 스트림
JDK 8에서는 세 가지 기본 특화 스트림을 제공한다. 박싱(Boxing) 비용을 피할 수 있도록 각 요소에 특화된 IntStream, DoubleStream, LongStream이 그 스트림 API이다.

각 인터페이스는 sum, max 등 숫자 관련 리듀싱 연산 수행 메서드를 제공하며, 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공한다.

특화 스트림은 `오직 박싱 과정에서 일어나는 효율성`과 관련이 있다는 것이 포인트이다!
##### 숫자 스트림으로 매핑
스트림을 특화 스트림으로 변환 시 mapToInt, mapToDouble, mapToLong을 주로 사용한다. map 스트림과 같은 기능을 수행하지만 제네릭 Stream 대신 특화된 스트림을 반환한다.
```java
int calories = menu.stream()
				   .mapToInt(Dish::getCalories)
				   .sum();
```
예시의 mapToInt는 각 요리에서 모든 칼로리 형식(Integer 형식)을 추출하여 IntStream을 반환한다. 이 후, IntStream 인터페이스가 제공하는 sum을 이용해 칼로리 합을 계산한다. 이 때 스트림이 비어있다면 sum은 0을 반환한다.
##### 객체 스트림으로 복원하기
숫자 스트림을 만들고 원상태인 특화되지 않은 스트림으로 복원하고 싶다면?

```java
// 스트림 -> 숫자 스트림
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);

// 숫자 스트림 -> 스트림
Stream<Integer> stream = intStream.boxed();
```
boxed 메서드를 이용해 특화 스트림을 일반 스트림으로 다시 변환할 수 있다. 이는 일반 스트림으로 박싱할 숫자 범위의 값을 다룰 때 boxed를 활용할 수 있다.

#### 숫자 범위
특정 범위의 숫자를 이용해야 하는 경우 IntStream과 LongStream에서는 ragne와 rangeClosed라는 두 정적 메서드를 제공한다.

```java
// 1부터 100까지의 범위
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
							.filter(n -> n % 2 == 0);
```
rangeClosed는 인자로 받은 범위까지의 숫자를 만들어낸다. 이후 filter 메서드로 짝수만을 필터링했다. filter를 호출하더라도 실제로 아무 계산도 이뤄지지 않는다.

### 스트림 만들기
#### .of()
```java
static <T> Stream<T> of​(T t)

// 단일 요소를 포함하는 순차적 스트림을 반환한다.
```
임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해 스트림을 만드는 방식이다. 예시는 스트림의 모든 문자열을 대문자로 변환하고 문자열을 하나씩 출력하는 것이다.
```java
Stream<String> stream = Stream.of("modern", "java", "in", "action");

stream.map(String::toUpperCase).forEach(System.out::println);
```
#### ofNullable
```java
static <T> Stream<T> ofNullable​(T t)

// null이 아닌 경우 단일 요소를 포함하는 순차적 스트림을 반환하고, 그렇지 않으면 빈 스트림을 반환한다.
```
JDK 9에 추가된 기능으로 null이 될 수 있는 개체를 스트림으로 만들 수 있다.

때로 null이 될 수 있는 객체를 스트림으로 만들어야할 수 있다. 예를 들어 System.getProperty는 제공된 키에 대응하는 속성이 없다면 null을 반환해야 한다는 조건이 있다고 해보자.
```java
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```
명시적으로 null을 반환하지 않고 스트림으로 null이 도리수 있는 객체를 스트림으로 만들 수 있다.
#### Arrays.stream
배열을 인수로 받는 정적 메서드 Arrays.stream을 이용해 스트림을 만들 수 있다.

```java
int[] nums = {1,2,3,4,5};
int sum = Arrays.stream(nums).sum(); // 15
```
#### File 스트림
파일을 처리하는 등 입출력 연산에 사용하는 NIO API도 스트림 API를 활용할 수 있다.

다음은 파일에서 고유한 단어 수를 찾는 프로그램이다.

```java
long uniqueWords = 0;

try(Stream<String> lines =
	   Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
		   uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
		   .distinct()
		   .count();
	   }
	   ) catch(IOException e) {
		...   
	}
```
Files.lines로 파일의 각 행 요소를 반환하는 스트림을 얻을 수 있다. 이 후 각 행의 단어를 flatMap으로 스트림을 하나로 평면화한 뒤, 중복제거 및 카운팅을 실행하는 코드이다.

#### 함수 무한스트림(infinite stream)
스트림 API는 함수에서 스트림을 만들 수 있는 정적 메서드 Stream.iterate와 Stream.generate를 제공한다. 이 함수를 통해 만든 스트림은 요청할 때마다 주어진 함수를 이용해 값을 만들며 무제한으로 값을 계산할 수 있다.
##### iterate
```java
static <T> Stream<T> iterate(T seed,
                             UnaryOperator<T> f)

// 함수 f를 초기 요소 시드에 반복적으로 적용하여 생성된 무한 순차 순서 스트림을 반환하고 시드, f(시드), f(f(f(시드))) 등으로 구성된 스트림을 생성한다. 
// 첫 번째 요소(위치 0) 스트림에는 제공된 시드가 있다. n > 0인 경우, 위치 n의 요소는 n - 1 위치의 요소에 함수 f를 적용한 결과가 된다.
```
iterate는 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 무한 스트림을 만든다. 이러한 스트림을 언바운드 스트림이라고 표현한다.

예시는 0부터 시작하여 2씩 증가하는 언바운드 스트림이다.

```java
Stream.iterate(0, n -> n + 2)
        .limit(10) // 언바운드 스트림 제한
        .forEach(System.out::println); // 0, 2, 4...
```
이 때 무한한 값을 출력하지 않도록 limit()과 함께 사용한다.

```java
IntStream.iterate(0, n -> n + 4)
        .takeWhile(n -> n < 100)
        .forEach(System.out::println);
```
내부적으로 프레디케이트를 지원하기 때문에 특정 조건에 맞는 숫자 생성도 가능하다.

이 때, takeWhile을 사용할 수 있는데, filter는 언제 중단해야 할지 알 수 없기 때문에 filter로는 대체되지 않는다는 특징이 있다.
##### generate
```java
static <T> Stream<T> generate(Supplier<T> s)

// 제공된 Supplier에 의해 각 요소가 생성되는 무한 순차 비순차 스트림을 반환한다.
// 이는 상수 스트림, 임의 요소 스트림 등을 생성하는 데 적하다.
```

generate는 iterate와 달리 생산된 값에 대한 연속성을 보장하지 않는다.

```java
// 0~1 사이 임의의 숫자 5개 생성
Stream.generate(Math::random)
         .limit(5)
         .forEach(System.out::println);
```
generate 내부에서는 Supplier를 받는다. 
이 때, 박싱 연산을 피하기 위해서는 기본형 특화 스트림을 사용하면 된다.

```java
DoubleStream.generate(Math::random)
            .limit(5)
            .forEach(System.out::println);
```
generate는 인자로 supplier를 받는데, supplier를 통해서 어떠한 조작을 통해 스트림에 존재하는 다음 값을 생성할 때 상태를 변경할 수도 있게 된다.

```java
IntSupplier fib = new IntSupplier() {
    private int previous = 0;
    private int current = 1;
    public int getAsInt() {
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
    }
};

IntStream.generate(fib).limit(10).forEach(System.out::println);
```

Supplier 내부에서 'previous', 'current' 변수를 선언한 다음에, 다음 요소들을 계산할 때 해당 변수의 값을 사용하는 것을 볼 수 있다. 이는 객체가 생성되었을 때 기존의 요소가 영향을 주기 때문에 **가변 상태 객체**가 된다.

만약, 병렬로 처리한다면 previous와 current의 값이 순차적으로 증가하는 형태가 아니게 될 것이다. 그래서 **병렬 상태에서는** 위의 코드처럼 상태를 가지고 있게 만드는 것이 아닌 **어떠한 상태에도 의존하지 않도록 코드를 작성**해야 한다.

반면에 iterate의 경우 상태에 대한 조작은 불가능하기 때문에 비교적 병렬 실행에 안전하다고 볼 수 있다. 

## 참고자료

- 모던 자바 인 액션
- https://docs.oracle.com/javase%2F9%2Fdocs%2Fapi%2F%2F/java/util/stream/Stream.html- java Stream docs

