---
layout  : wiki
title   : 동작 파라미터화 코드 전달하기 
summary : 
date    : 2024-06-14 21:38:40 +0900
updated : 2024-06-14 21:39:21 +0900
tag     : java
resource: 78/8AC8D6-3F4F-4E7E-B6C2-3DFCC2050F32
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 동작 파라미터화 코드 전달하기

`동작 파라미터화(behavior parameterization)`는 어떻게 실행할 것인지 결정하지 않은 코드, 즉 빈번한 요구사항 변경을 처리하는 방법을 제공하는 소프트웨어 개발 패턴이다.

코드로 살펴보자. 빨간색, 녹색 사과가 담긴 inventory에서 녹색 사과만 선별하는 작업이다.

```java
// Color.java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        // inventory에 담긴 사과가 녹색이라면 result에 추가
        if (GREEN.equals(apple.getColor())) {
            result.add(apple);
        }
        return result;
    }
}
```
이 코드의 문제점은 무엇일까?

녹색 사과가 아니라 빨간 사과를 선별해야한다면 코드를 어떻게 바꿀 수 있을까?
별도의 filter 메서드를 만든다면 빨간 사과를 바꿀 수 있을 것 같다.

```java
public static List<Apple> filterRedApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>();
	for (Apple apple : inventory) {
		// inventory에 담긴 사과가 빨강이라면 result에 추가
		if(RED.equals(apple.getColor())) {
			result.add(apple);
		}
	}
	return result;
}
```
하지만 빨강, 녹색 이외의 다른 색이 추가된다면 그 때마다 filter 메서드를 만들어야 할까? 이는 중복된 작업일 것이다.

그렇다면 사과의 색깔별로 filter 메서드를 만드는 반복 작업을 하지않고 filter를 구현할 수 있을까? 색을 파라미터화 할 수 있도록 메서드에 파라미터를 추가해보자.

```java
public static List<Apple> filterRedApples(List<Apple> inventory, Color color) {
	for (Apple apple : inventory) {
		if(apple.getColor().equlas(color)) {
			result.add(apple);
		}
	}
	return result;
}

// 파라미터로 건낸 색깔을 가진 사과를 선별
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```
Color ENUM을 파라미터로 받은 코드이다. 이제 파라미터로 Color ENUM을 넘겨 원하는 사과의 색을 선별할 수 있다.

하지만 여기서 색 뿐만아니라 사과를 무게에 따라 구분한다고 하면 어떨까? 색과 동일하게 무게 파라미터를 건네면 되지 않을까?

```java
public static List<Apple> filterRedApples(List<Apple> inventory, Color color, int weight) // weight를 추가 {
	for (Apple apple : inventory) {
		if(apple.getWeight() > weight) {
			result.add(apple);
		}
	}
	return result;
}
```

하지만 이렇게 파라미터를 계속 추가하여 필터링 조건을 적용하는 부분은 사과의 색을 선별하는 코드와 대부분 중복된다. 이를 동작 파라미터화를 이용해 유연성을 도모해보자.

### 동작 파라미터화 적용

파라미터로 조건을 추가하는 방법이 아니라, 사과의 어떤 속성에 기초하여 boolean값을 반환하는 방법이 있는데, 여기서 true 또는 false를 반환하는 함수를 `predicate(프레디케이트)`라한다.

사과 선택 조건을 결정하는 인터페이스를 정의해보자.

```java
// ApplePredicate.java
public interface ApplePredicate {
	boolean test (Apple apple);
}
```

이제 선택 조건을 결정하도록 인터페이스를 구현한다.

```java
// AppleHeavyWeightPredicate.java
public class AppleHeavyWeightPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return apple.getWeight() > 150;
	}
}

// AppleGreenColorPredicate.java
public class AppleGreenColorPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return GREEN.equals(apple.getColor());
	}
}
```

ApplePrediate 인터페이스는 사과를 선택하는 조건을 캡슐화했다. 따라서 ApplePredicate 구현 클래스들은 인터페이스를 통해서만 접근되기 때문에, 사용자나 클라이언트 코드는 필터링 로직의 세부사항을 알 필요없이 test 메서드를 사용할 수 있다.

이로써 ApplePredicate를 구현하는 클래스들이 내부 로직을 변경하더라도 인터페이스를 사용하는 클라이언트의 코드는 영향을 받지 않게되며, 한 번 정의한 ApplePredicate 구현체들은 다양한 곳에서 재사용가능하다.

이처럼 '전략'이라고 불리는 각 알고리즘을 캡슐화하는 알고리즘군을 정의하고 런타임에 알고리즘을 선택하는 기법을 `전략 패턴(strategy pattern)`이라 한다.

위에서 살펴본 예제는 ApplePredicate가 알고리즘군이며, AppleHeavyWeightPredicate와 AppleGreenColorPredicate가 전략에 해당하는 것이다.

이제 filterApples에서 Predicate 객체를 받아 사과의 조건을 검사하도록 고쳐보자.

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate pd) // ApplePredicate를 파라미터로 추가 {
	for (Apple apple : inventory) {
		if(pd.test(apple)) {
			result.add(apple);
		}
	}
	return result;
}
```

이제 filterApples에서는 '전략'을 파라미터로 받아 내부에서 다양한 동작을 수행한다.
Predicate 객체로 사과의 선별 조건을 캡슐화하여 filterApples에서는 필터링 세부사항을 클라이언트에게 알리지 않는다.

이로써 내부 로직을 변경하더라도 외부에서 이 객체를 사용하는 코드에는 영향을 미치지않게되며, 한 메서드(test)가 다른 동작을 수행하도록 재활용할 수 있다.

### 코드 간소화

지금까지 동작을 추상화하여 변화하는 사과 선별 조건에 대응할 수 있는 코드를 작성했다.

하지만 filterApples 메서드로 새로운 전략을 전달할 때 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 뒤에 인스턴스화 해야하는 번거로운 작업이 계속된다.

이를 개선해보자.

#### 익명 클래스(Anonymous class)

`익명 클래스`는 클래스를 선언하고 동시에 인스턴스화 할 수 있다. 이름이 없다는 점을 제외하면 로컬 클래스와 같으며, 문서에서는 로컬 클래스를 한 번만 사용해야 하는 경우 이를 사용하라고 설명한다.

익명클래스 사용 전에 익명 클래스의 속성을 살펴보자.

- 생성자 문제

익명 클래스는 인스턴스 초기화 블록과 상위 클래스의 생성자를 통해 초기화되기 때문에 생성자없이 객체를 생성할 수 있다.

- 정적 멤버(static member)

익명 클래스는 상수를 제외하고 정적 멤버를 가질 수 없다.
```java
new Runnable() { 
	static final int x = 0; 
	static int y = 0; // compilation error!
	
	@Override 
	public void run() {...} 
};
```
위 코드를 컴파일하면

```
The field y cannot be declared static in a non-static inner type, unless initialized with a constant expression
```

위와 같은 에러를 확인할 수 있다. 직역해보면 필드 y는 상수 표현식으로 초기화하지 않는 한 non-static(정적이 아닌) inner type으로 선언할 수 없다는 것이다.

- 변수의 범위

익명 클래스는 클래스를 선언한 블록 범위에 있는 지역 변수를 캡처한다.

```java
int count = 1;
Runnable action = new Runnable() {
    @Override
    public void run() {
        System.out.println("Runnable with captured variables: " + count);
    }           
};
```

코드를 보면 지역 변수인 count와 action은 동일한 블록에 정의되어 있다. 이는 클래스 선언 내에서 count에 접근할 수 있다.

지역 변수를 사용하려면 사실상 final 키워드를 명시한 변수여야 한다. JDK 1.8부터는 final 키워드를 사용해 변수를 선언할 필요는 없지만 그럼에도 해당 변수는 final이어야 한다.

```
[ERROR] local variables referenced from an inner class must be final or effectively final
```

그렇지 않으면 위와 같은 컴파일 오류를 볼 수 있을 것이다.

이제 익명 클래스를 이용하여 사과를 선별하는 필터링 예제를 구현해보자.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
	public boolean test(Apple apple) {
		return RED.equals(apple.getColor());
	}
}
```

filterApples() 메서드의 동작을 직접 인스턴스화하여 파라미터화 했다.

하지만 여전히 파라미터로 전달된 ApplePredicate()와 그 안의 test() 메서드가 반복되어 가독성이 좋지 않으며, 새로운 동작을 정의하는 test 메서드를 구현해야 한다는 점은 여전히 변하지 않는다.

이렇게 일시적으로 사용된다는 것은 나중에 재사용이 되지 않는다는 것인데, 재사용할 필요를 없는 객체는 확장성이 좋지 않다는 것이다.

좀 더 좋은 방법은 없을까?

### 람다 표현식(Lambda expressions)

JDK 1.8에 추가된 `람다 표현식`은 기능을 메서드 인수로 처리하거나 코드를 데이터로 처리할 수 있는 기능으로 함수를 하나의 식으로 표현한 것이다.

람다 표현식도 메서드의 이름이 필요없기 때문에 익명 함수의 한 종류라고 볼 수 있다.

사과 필터링 예제를 람다 표현식으로 구현하면 다음과 같다.

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

익명 클래스의 장황함을 없애고 훨씬 간결해졌다. 이를 리스트 형식으로 추상화해보자.

```java
public interface Predicate<T> {
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> pd) {
	List<T> result = new ArrayList<>();
	for(T e : list) {
		if(pd.test(e)) {
			result.add(e);
		}
	}
	return result;
}
```

Predicate 인터페이스는 단일 메서드 test를 가지며, 주어진 타입 T의 객체를 테스트하며, test 메서드는 주어진 조건을 만족하는지 여부를 결정하는 역할을 한다.

이어서 **filter 메서드**는 제네릭 타입을 사용하여 다양한 타입의 리스트를 조건에 맞게 필터링하도록 변경했다.

이제 람다 표현식을 사용해 빨간 사과를 고르면 다음 코드로 정리할 수 있다.

```java
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

이처럼 람다 표현식은 코드에 유연성과 간결함을 가져다준다.
### Java API에 적용해보기

#### Comparator

`java.util.Comparator`는 일부 개체 컬렉션에 전체 순서를 적용하는 비교기능이다.

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

타입 파라미터로 제네릭을 받으며 비교할 객체의 타입을 지정한다.
compare() 메서드는 두 객체를 비교하여 순서를 결정하게 되는데, 이 때 반환값이 음수라면 첫 번째 객체(o1)가 두 번째 객체(o2)보다 작은 것며, 0이라면 두 객체가 같음, 양수라면 첫 번째 객체가 두 번째 객체보다 큰 것이다.

이를 이용해 Comparator를 구현하여 익명 클래스로 사과를 정렬해보자.

```java
inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
});
```

요구사항이 바뀌더라도 요구사항에 맞는 Comparator를 구현하여 sort() 메서드에 Comparator를 파라미터로 전달하기만 하면된다.

람다 표현식으로 좀 더 간결하게 만들어보자.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```

이처럼 람다 표현식은 인터페이스를 상속받아 여러 클래스를 구현해야 하는 수고를 덜어준다.

## 참고자료

- 모던 자바 인 액션
- https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html - Java tutorial docs
- https://docs.oracle.com/javase%2Ftutorial%2F/java/javaOO/lambdaexpressions.html - Lambda Expressions docs
 
