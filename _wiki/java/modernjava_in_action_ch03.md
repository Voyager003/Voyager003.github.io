---
layout  : wiki
title   : 람다 표현식
summary : 
date    : 2024-06-21 20:47:42 +0900
updated : 2024-06-21 20:52:06 +0900
tag     : java
resource: B6/9E1B04-76F5-4F95-8675-A87142CB162A
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

 [챕터 2](https://voyager003.github.io/wiki/java/modernjava_in_action_ch02/)에서 살펴본 익명 클래스의 문제점은 코드가 깔끔하지 않고 불분명하게 보일 수 있다는 점이었다. 챕터 3에서는 더 깔끔한 코드로 문제점을 해결하는 람다 표현식에 대해 자세히 알아보자.
## 람다 표현식
`람다 표현식`은 메서드로 전달할 수 있는 익명 함수를 하나의 식으로 단순화한 것으로 JDK 1.8에 추가된 기능이다. 익명 클래스와 같이 이름은 없지만, 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트를 가질 수 있다.

기존에 반환타입, 메서드를 선언하는 방식과 어떤 차이가 있는지 알아보자.

```java
// 기존의 방식 
public String ex1() {
	return "hi!";
}

// 람다 표현식 - (매개변수) -> { 실행문 }
() -> "hi!";

// 응용
Comparator<Apple> byWeight = new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
}

Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
람다 표현식은 코드에서 보듯이 세 부분으로 이뤄진다.

- 파라미터 리스트: 코드에서 (Apple a1, Apple a2)가 이에 해당한다.
- 화살표(->):  람다 파라미터 리스트와 바디를 구분한다.
- 람다 바디: 화살표 이후 로직인, a1.getWeight().compareTo(a2.getWeight());가 이에 해당하며, 람다의 반환값에 해당하는 표현식이다.

위 예시는 표현식 스타일(expression style)로 다음처럼 블록 스타일(block style)로 표현할 수도 있다.

```java
(parameters) -> { statements; }

// 두 숫자의 합을 계산하고 결과를 출력하는 블록 표현식으로 나타낸 람다
Function<int[], Integer> sumAndPrint = (int[] numbers) -> {
	int sum = 0;
	for (int number : numbers) {
		sum += number;
	}
    return sum;
}
```
이처럼 코드가 간결해지는 것 외에 람다 표현식의 용도는 무엇일까?
## 함수형 인터페이스(Functional interface)
### 함수형 프로그래밍
함수형 인터페이스에 대해 설명하기 전에 함수형 프로그래밍에 대해 짚고 가보자.

먼저 [위키백과](https://ko.wikipedia.org/wiki/%ED%95%A8%EC%88%98%ED%98%95_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)에서 설명하는 사전적 의미는 다음과 같다.

> 자료 처리를 수학적 함수의 계산으로 취급하고 상태와 가변 데이터를 멀리하는 프로그래밍의 패러다임의 하나이다.
> (중략)
> 명령형 프로그래밍에서는 상태를 바꾸는 것을 강조하는 것과는 달리, 함수형 프로그래밍은 응용을 강조한다. 1930년대에 계산 가능성, 결정문제, 함수 정의와 응용과 재귀를 연구하기 위해 개발된 형식체계인 [람다 대수](https://ko.wikipedia.org/wiki/%EB%9E%8C%EB%8B%A4_%EB%8C%80%EC%88%98)에 근간을 두고 있다.

Clean code의 저자 Robert C.Martin이 정의한 함수형 프로그래밍을 보면 좀 더 와닿을 것이다.

> Functional Programming is programming without assignment satements

대입문이 없는 프로그래밍이 무엇일까? 코드로 살펴보자. (아래 코드는 실제 동작 여부와 무관한 pseudo code이다.)

```jaava
filter(inventory, print(apple));
```

filter 함수는 첫 번째 인자로 inventory를 받고, 두 번째 인자로 출력하라는 함수를 파라미터로 받고 있다. 함수형 프로그래밍은 '무엇'에 초점을 두는 프로그래밍 패러다임이기 때문에 '출력'이라는 작업을 하는 함수를 파라미터로 넘길 수 있는 것이다.

명령형 프로그래밍 패러다임에서 메서드를 호출하면 개발한 함수 내에서 선언된 변수의 메모리에 할당된 값이 바뀌는 등 변화가 발생할 수 있지만, 함수형 프로그래밍은 대입문이 없어 메모리에 한 번 할당된 값은 새로운 값으로 변할 수 없다.

그 외에 함수형 프로그래밍의 특징에 대해 좀 더 자세하게 알아보자.
#### 부수 효과(Side effect)와 순수 함수(Pure Function)
여기서 말하는 `부수 효과`는

- 변수의 값이 변경되거나
- 자료 구조를 제자리에서 수정하거나
- 객체의 필드 값을 설정하거나
- 예외나 오류가 발생하여 실행이 중단되거나
- 콘솔 혹은 파일 입출력이 발생

하는 것을 말한다. 이러한 부수 효과를 제거한 함수들을 `순수 함수`라 부른다.

순수 함수를 사용함으로써 얻을 수 있는 효과는 함수 자체가 독립적으로 부수 효과가 없기 때문에 Thread-safe를 보장받을 수 있으며, 이 Thread-safe를 보장받기 때문에 동기화없이 병렬 처리를 할 수 있다.
#### 1급 객체(First-Class Object)
`1급 객체`라는 개념은 일급 시민(first-class citizen)이라고도 하며 [Robin Popplestone](https://en.wikipedia.org/wiki/Robin_Popplestone)라는 개발자가 명확한 기준을 제시했다.

- 모든 일급 객체는 함수의 실질적인 매개변수가 될 수 있다.
- 모든 일급 객체는 함수의 반환값이 될 수 있다.
- 모든 일급 객체는 할당의 대상이 될 수 있다.
- 모든 일급 객체는 비교 연산( == , equal)을 적용할 수 있다.

이를 코드로 작성해보자. 코드는 Java를 사용했다.

```java
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.Objects;

public class FirstClassCitizenExample {
    public static void main(String[] args) {
        // 1. 함수의 실질적인 매개변수가 될 수 있다.
        Function<Integer, Integer> square = x -> x * x;
        System.out.println(applyFunction(5, square)); // 25

        // 2. 함수의 반환값이 될 수 있다.
        Function<Integer, Integer> increment = x -> x + 1;
        Function<Integer, Integer> incrementThenSquare = composeFunctions(increment, square);
        System.out.println(incrementThenSquare.apply(5)); // 36

        // 3. 할당의 대상이 될 수 있다.
        Function<Integer, Integer> multiplyByTwo = x -> x * 2;
        Function<Integer, Integer> assignedFunction = multiplyByTwo;
        System.out.println(assignedFunction.apply(5)); // 10

        // 4. 비교 연산을 적용할 수 있다.
        Function<Integer, Integer> anotherMultiplyByTwo = x -> x * 2;
        System.out.println(Objects.equals(multiplyByTwo, anotherMultiplyByTwo)); // false (due to lambda identity)
    }

    // 함수의 매개변수로 전달
    public static <T, R> R applyFunction(T value, Function<T, R> function) {
        return function.apply(value);
    }

    // 함수의 반환값으로 사용
    public static <T> Function<T, T> composeFunctions(Function<T, T> first, Function<T, T> second) {
        return first.andThen(second);
    }
}
```

이처럼 함수를 일급 객체로 취급하는 것이 함수형 프로그래밍에서 필수적인 속성이다.
#### 참조 투명성(Referential Transparency)
`참조 투명성`은 **동일한 인자에 대해 항상 동일한 결과를 반환해야 하며**, **참조 투명성을 통해 기존의 값은 변경되지 않고 유지되는 것**이다.

명령형 프로그래밍과 함수형 프로그래밍에서 사용하는 함수의 다른 점 중 하나는 위에서 살펴본 '부수 효과'의 유무이다. 참조가 투명하다는 것은 함수를 실행해도 상태 변화 없이 항상 동일한 결과를 반환하여 동일한 실행 결과를 참조할 수 있다는 것이다.

이처럼 부수 효과를 제거하여 프로그램의 동작을 이해하고 예측을 용이하게 하는 것이 함수형 프로그래밍의 핵심이다.
### 함수형 인터페이스
`함수형 인터페이스`는 **정확히 하나의 추상 메서드를 지정하는 인터페이스**로 챕터 2에서 살펴봤던 Predicate < T >도 이에 해당하며, Java의 API 중 Comparator, Runnable도 함수형 인터페이스에 해당된다.

이 함수형 인터페이스로 어떤 것이 가능할까?

먼저 람다 표현식으로 함수형 인터페이스의 추상 메서드의 구현을 직접 전달하여 전체 표현식을 인터페이스의 인스턴스로 취급할 수 있다.
#### 함수 디스크립터(function descriptor)
함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리키며, 표현식의 시그니처를 서술하는 메서드를`함수 디스크립터`라 한다.

여기서 시그니처는 메서드의 이름과 파라미터 목록을 의미하며, 파라미터 명은 시그니처에 포함되지 않는다.

코드로 살펴보자.

```java
public class SignatureExample {
    // 메서드 시그니처: foo(int, String)
    public void foo(int a, String b) {
        // 구현
    }

    // 메서드 시그니처: foo(double)
    public void foo(double a) {
        // 구현
    }
}
```
위 코드에서 각 foo 메서드는 다른 시그니처를 가진다.
먼저 foo(int, String)는 두 개의 매개변수 int와 String을 가지며, foo(double)는 하나의 매개변수 double을 가지는 것이다.

```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```

API중 하나인 Runnable은 Runnable의 유일한 추상 메서드 run()이 인수와 반환값이 없어 Runnable 인터페이스는 인수와 반환 값이 없는 시그니처라고 할 수 있다.

#### 실행 어라운드 패턴(execute around pattern)

![SCR-20240621-sbzt](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/a35dc9c2-9f09-4ef0-816b-abb556b59fc4)

`실행 어라운드 패턴`은 실제 자원을 처리하는 코드를 정리 두 과정이 둘러싸는 형태를 가지는 패턴이다. 코드로 이해해보자.

```java
public String processFile() throws IOException {
	try (BufferedReader br =
		new BufferedReader(new FileReader("data.txt"))) {
	return br.readLine();
	}
}
```

코드는 data.txt파일에서 한 행을 읽어들이는 코드로, br.readLine()에서 실제 필요 작업을 실행한다.

현재 이 코드는 한 번에 한 줄만 읽을 수 있다. 만약 한 번에 두 줄을 읽거나 가장 자주되는 단어를 반환하려면 어떻게 해야 할까?

기존 설정과 정리 과정은 재사용하고 processFile() 메서드를 다른 동작을 하도록 하면 좋을 것 같다. 챕터 2에서 다뤘던 동작 파라미터화를 적용해보자.

```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
BufferedReader를 파라미터로 받아 String을 반환하도록 하여 processFile() 메서드가 한 번에 두 행을 읽도록 변경한 코드이다.

여기서 함수형 인터페이스 자리에 람다를 사용할 수 있다. 이를 위해 첫 코드의 시그니처와 일치하는 함수형 인터페이스가 필요하다.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}
```
이제 정의한 함수형 인터페이스를 processFIle의 파라미터로 전달하면

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	...
}
```
BufferedReaderProcessor에 정의된 process 메서드의 시그니처와 일치하는 람다를 전달할 수 있다.

이제 람다 코드는 processFIle 내부에서 추상 메서드 구현을 직접 전달하며, 함수형 인터페이스로 전달된 코드와 같은 방식으로 처리되어 processFIle 바디 내에서BufferedReaderProcessor 객체의 process() 메서드를 호출할 수 있게 된다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	try (BufferedReader br = 
		new BufferedReader(new FilerReader("data.txt"))) {
	return p.process(br);
	}
}
```

이제 람다를 이용해 여러 동작을 processFile 메서드로 전달할 수 있다.

한 행, 두 행을 처리하도록 변경해보자

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());

String twoLine = processFile((BufferedReader br) -> br.readLine()) + br.readLine();

```
이처럼 실행 어라운드 패턴을 함수형 인터페이스를 이용해 람다를 전달하면 여러 동작을 간결하게 구현할 수 있다.
#### 예외, 람다, 함수형 인터페이스의 관계
함수형 인터페이스는 Checked Exception(확인된 예외)를 던지는 동작을 허용하지 않는다.

코드로 예외를 처리하는 방식을 알아보자.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedRedaer b) throws IOException;
}

BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
```
첫 번째로 예외를 선언하는 함수형 인터페이스를 정의하는 방법이다.

```java
Function<BufferedReader, String> f = (BufferedReader b) ->
	try {
		return b.readLine();
	} catch(IOException e) {
		throw new RuntimeException(e);
	}
};
```
두 번째는 예외를 명시적으로 처리하는 방법으로, 람다를 try/catch 블록으로 감싸 처리할 수 있다.
## 람다 표현식의 형식검사, 추론, 제약
람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지 정보가 포함되어 있지 않다.
때문에 람다의 실제 형식을 파악할 필요가 있다.
### 형식 검사
람다가 사용되는 컨텍스트를 통해 람다의 형식을 추론할 수 있다.
이 때, 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수와 같은 컨텍스트에서 기대되는 람다 표현식의 형식을 `대상 형식(target type)`이라 부른다.

다음 람다 표현식에서 어떻게 형식을 추론하는지 알아보자.
```java
List<Apple> heavierThan150g = filter(inventory, (Apple apple) -> apple.getWeight()>150);
```

- filter 메서드의 선언을 확인
```java
filter(inventory, (Apple a) -> a.getWeigth() > 150);
```
먼저, 람다가 사용된 컨텍스트를 확인한다.

- filter 메서드는 두 번째 파라미터로 Predicate< Apple > 형식을 기대
```java
filter(List<Apple> inventory, Predicate< Apple > p)
```
대상 형식은 Predicate< Apple >이며, 제네릭은 Apple 인스턴스로 대치된다.

- Predicate< Apple >는 test()라는 하나의 추상 메서드를 정의하는 함수형 인터페이스

Predicate< Apple >의 추상 메서드를 확인한다.

- test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사
```java
boolean test(Apple apple)
```
추상 메서드는 Apple을 파라미터로 받아 boolean을 반환하는 메서드임을 확인한다.

- filter 메서드로 전달된 인수는 이 요구사항을 만족해야 한다.

Apple을 파라미터로 받아 boolean을 반환하는 코드이기 때문에 유효한 코드이다.
### 형식 추론
람다 표현식은 `대상 형식`이라는 특징때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다. 여기서 대상 형식은 컨텍스트를 의미한다.

```java
@FunctionalInterface  
public interface Callable<V> {  
	V call() throws Exception;  
}

@FunctionalInterface  
public interface PrivilegedAction<T> {  
	T run();  
}

Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

Callable과 PrivilegedAction 인터페이스 모두 인수를 받지 않고 제네릭(T)를 반환하지만, 컨텍스트 정보를 통해 컴파일러가 람다 표현식의 타입을 람다 표현식이 할당되거나 전달되는 위치의 기대되는 타입을 바탕으로 추론한다.

### 지역 변수
지금까지 본 람다 표현식은 인수를 자신의 람다 바디 안에서만 사용했다.

하지만 람다 표현식에서는 익명 함수가 하는 것 처럼 자유 변수(free variable)를 활용할 수 있다. 자유 변수는 파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수를 의미한다.

이와 같은 동작을 `람다 캡처링(capturing lambda)`이라 부른다. 코드로 보면 다음과 같다.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```
람다 표현식이 실행될 때 외부변수인 portNunber를 함께 포함하도록 했다.

하지만 이런 자유 변수에는 제약이 있다.
인스턴스 변수와 정적 변수를 자신의 람다 바디에서 참조 할 수 있지만, 그러려면 지역 변수는 명시적으로 final로 선언되어 있거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.

이는 한 번만 할당할 수 있는 지역변수를 캡처할 수 있다는 것이다. 실제로 다음 코드는 에러를 발생시킨다.

```java
 public class Main {  
    public static void main(String[] args) {  
        int portNumber = 1337;  
        Runnable r = () -> System.out.println(portNumber);  
        portNumber = 3125;  
    }  
}

// error java: local variables referenced from a lambda expression must be final or effectively final
```

portNumber에 값을 두 번 할당하므로 컴파일할 수 없는 코드라는 것이다.

이런 제약은 지역 변수와 인스턴스 변수가 어디에 할당되는지에 대한 차이에서 온다고 볼 수 있다.

인스턴스 변수는 메모리의 Heap 영역에 저장되는 반면, 지역 변수는 Stack에 할당된다.
람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면, 변수를 할당한 스레드가 사라져 할당이 해제되었음에도 람다를 실행하는 스레드에서는 해당 변수에 접근하는 일이 일어난다.

따라서 Java에서는 원래 변수에 접근을 허용하는 것이 아닌 자유 지역 변수의 복사본을 제공하는 것이며, 이 복사본의 값이 바뀌지 않아야 하기 때문에 지역 변수에는 한 번만 값을 할당해야 하는 제약이 생긴 것이다.

## 메서드 참조(Method Reference)
`메서드 참조`는 람다 표현식을 더 간결하고 읽기 쉽게 작성할 수 있도록 도와주는 기능이다.
기존의 메서드 정의를 재활용하여 람다처럼 전달할 수 있으며, 케이스에 따라 더 가독성이 좋은 경우가 있다. 코드로 살펴보자.

```java
// 람다 표현식
inventory.sort((Apple a1, Apple a2) -> 
				  a1.getWeigth().compareTo(a2.getWeight()));

// 메서드 참조
inventory.sort(comparing(Apple::getWeight));
```
메서드명 앞에 구분자(::)를 붙이는 방식으로 메서드 참조를 활용한다. 예시의 Apple::getWeight는 Apple 클래스에 정의된 getWeight의 메서드를 참조한 것이다.

### 유형
#### 정적 메서드(static method) 참조
```java
public static int parseInt(String s) throws NumberFormatException {  
    return parseInt(s,10);  
}

// 메서드 참조
Integer::parseInt
```
정적 메서드 중 하나인 Integer.parseInt를 메서드 참조로 나타낸 코드이다.
#### 다양한 형식의 인스턴스 메서드 참조
```java
public int length() {  
    return value.length >> coder();  
}

// 메서드 참조
String::length
```
정적 메서드가 아니더라도 일반 인스턴스 메서드를 참조할 수도 있다.
#### 기존 객체의 인스턴스 메서드 참조
```java
// Transaction.java
public class Transaction {

    private int value;

    public Transaction(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}

// 메서드 참조
public class MethodReferenceExample {
    public static void main(String[] args) {
        // Transaction 객체 생성
        Transaction expensiveTransaction = new Transaction(1000);

        // 메서드 참조를 사용하여 getValue 메서드를 참조
        Supplier<Integer> valueSupplier = expensiveTransaction::getValue;

        // getValue 메서드 호출
        int value = valueSupplier.get();
	    ...
    }
}
```
Transaction 객체를 할당받은 expensiveTransaction 지역변수가 있고, Transaction 객체에는 getValue 메서드가 있다. 이를 메서드 참조로 나타낸 코드이다.

이 때, 메서드 참조는 컨텍스트의 형식과 일치해야 한다.
### 생성자 참조(Constructor Reference)
`생성자 참조`는 기존 생성자를 메서드 참조처럼 사용할 수 있게 해주는 방법으로, 람다 표현식보다 간결하고 직관적인 방법을 제공한다.

```java
// Person.java
public class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}

// 생성자 참조
import java.util.function.Function;
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

public class ConstructorReferenceExample {
    public static void main(String[] args) {
        // Person 객체를 생성하기 위한 Function 인터페이스
        Function<String, Person> personCreator = Person::new;

        // Function 인터페이스를 통해 Person 객체 생성
        Person person = personCreator.apply("ROME");

        // Supplier 인터페이스를 사용하여 리스트 생성
        Supplier<List<String>> listSupplier = ArrayList::new;
        List<String> list = listSupplier.get();

        // 리스트에 값 추가 및 출력
        list.add("Hello");
        list.add("World");
        list.forEach(System.out::println); // Hello World
    }
}
```

이처럼 생성자 참조를 사용하면 인스턴스화하지 않고도 생성자에 접근할 수 있다.

## 참고자료

- 모던 자바 인 액션
- https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html - java docs
- https://nesoy.github.io/articles/2018-05/Functional-Programming - 함수형 프로그래밍
- https://cyberx.tistory.com/55 - 함수형 프로그래밍
- https://jcsoohwancho.github.io/2019-10-18-1%EA%B8%89-%EA%B0%9D%EC%B2%B4(first-class-object)%EC%9D%B4%EB%9E%80/ - 1급 객체란?

