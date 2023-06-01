---
layout  : wiki
title   : Java Lambda Expressions
summary : 
date    : 2023-06-01 09:39:23 +0900
updated : 2023-06-01 14:59:25 +0900
tag     : java
resource: 63/FEBF26-A607-4006-BA3D-F08899AA082F
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/15) 15주차 과제

## Lambda Expressions(람다 표현식) 사용법

### 람다 표현식?

> 식별자 없이 실행가능한 함수

- 메서드를 하나의 식으로 표현하며, 메서드 이름과 return이 나타나지 않아 anonymous function(익명 함수)라고도 한다.
- JDK 1.8에 추가된 기능으로, 객체지향 언어인 동시에 함수형 언어로써의 기능을 제공한다.

### 작성법

```java
// 일반적 메서드 선언
int max(int a, int b) {
    return a > b ? a : b;
}

// 람다 표현식 적용
(int a, int b) -> { a > b ? a : b; }

// 람다 표현식 적용
(int a, int b) -> a > b ? a : b
```

- 메서드에서 이름과 반환 타입을 제거하고, 매개변수 선언부와 몸통('{}') 사이에 '->'를 추가한다.
- 반환 값이 있는 메서드의 경우 return문 대신 식으로 대신할 수 있다.
- 식의 연산 결과가 자동적으로 반환값이 되며, 문장이 아닌 식으로 ';'를 붙이지 않는다.

```java
// 타입 추론
(a, b) -> a > b ? a : b

// 괄호 생략
a -> a * a
int a -> a * a // Error
```

- 선언된 매개변수의 타입은 추론이 가능한 경우 생략 가능하다.
- 람다 표현식에 반환 타입이 없는 이유도 항상 추론이 가능하기 때문이다.
- 또한 선언된 매개변수가 하나인 경우 괄호를 생략할 수 있다.
- 단, 매개변수에 타입이 있으면 괄호 생략이 불가능하다.

## 함수형 인터페이스(@FunctionalInterface)

함수형 인터페이스는 추상 메서드를 하나만 가진 인터페이스로 Single Abstract Method(SAM)라고도 한다.

Java의 모든 메서드는 클래스 내에 포함되어야 한다. 이 때, 람다 표현식은 어느 클래스에 포함되는 것일까?

```java
// 람다 표현식
(int a, int b) -> a > b ? a : b

// 익명 클래스
new Object() {
    int max(int a, int b) {
        return a > b ? a : b;
    }
}
```

람다 표현식으로 정의된 익명 객체의 메서드는 참조 변수가 있어야 객체의 메서드가 호출할 수 있다.

익명 객체의 주소를 d라는 참조변수에 저장해보자.

```java
타입 d = (int a, int b) -> a > b ? a : b;
```

이 때, 참조변수 d의 타입은 참조형으로 클래스 혹은 인터페이스가 올 수 있다.

그리고 람다 표현식과 동등한 메서드가 정의되어 있는 것이어야 참조 변수로 람다 표현식의 메서드를 호출할 수 있다.

그렇다면 max()가 정의된 MyFunction 인터페이스가 정의되어 있다고 가정해보자.

```java
interface MyFunction {
    public abstract int max(int a, int b);
}
```

MyFunction 인터페이스를 구현한 익명 클래스의 객체는 다음과 같이 생성 가능하다.

```java
MyFunction d = new MyFunction() {
    public int max(int a, int b) {
        return a > b ? a : b;
    }
}
```

MyFunction 인터페이스에 정의된 max()는 람다 표현식인 (int a, int b) -> a > b : a : b와 일치한다.

이를 다음과 같이 대체할 수 있다.

```java
MyFunction d = (int a, int b) -> a > b ? a : b; // 익명 객체를 람다식으로 대체
int big = d.max(5, 3); // 익명 객체 메서드 호출 
```

MyFunction 인터페이스를 구현한 익명 객체를 람다 표현식으로 대체할 수 있는 이유는,

람다 표션식도 실제로는 익명 객체이며 MyFunction 인터페이스를 구현한 익명 객체의 메서드 max()와 람다 표현식의 
변수 타입과 갯수 반환값이 일치하기 때문이다.

이처럼 하나의 메서드가 선언된 인터페이스를 정의하여 람다 표현식을 다루는 것은 Java의 규칙을 어기지 않으면서 자연스러운 문법이 되었다.

때문에 인터페이스를 통해 람다 표현식을 다루기로 결정되었으며, 람다 표현식을 다루기 위한 인터페이스를 함수형 인터페이스(Functional Interface)라 부르기로 했다.

```java
@FunctionalInterface
interface MyFunction {
    public abstract int max(int a, int b);
}
```

- 이 때, 함수형 인터페이스에서는 오직 하나의 추상 메서드만 정의되어 있어야 한다는 제약이 있다.
- 이는 람다 표현식과 인터페이스의 메서드가 1:1로 연결될 수 있기 때문이다.
- 하지만, static과 dafault 메서드의 개수에는 제약이 없다.
- 함수형 인터페이스로 구현한 인터페이스라면 @FunctionalInterface를 명시하여 컴파일러가 함수형 인터페이스를 올바르게 정의했는지 확인해 주도록 하자.

### Java 기본 제공 함수형 인터페이스

몇 가지만 살펴보자.

- Java.lang.function 패키지
- Suppliers

```java
public intereface Supplier<T> {
    T get();
}

// 사용
public class SupplierEx {
    public static void main(Stringp[] args) {
        Supplier<String> supplier = () -> "Supplier";
        String s = supplier.get();
        System.out.println(s);
    }
}

// 실행 결과
Supplier
```

    - Supplier<T>는 인자를 받지 않고 T 타입의 객체를 반환한다.

- Counsumer

```java
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

// 사용
public class ConsumerEx {
    public static void main(String[] args) {
        Consumer<String> print = str -> System.out.println("this is " + str + " Interface");
        print.accept("Consumer");
    }
}
// 결과
// this is Consumer Interface
```
    - Consumer<T>는 T타입의 객체를 인자로 받고 반환값은 없다.

## Variable Capture

- 람다 표현식은 특정 상황에서 람다 함수 외부에 선언된 변수에 의해 접근할 수 있다.
- JDK 1.8 이전에는 익명의 내부 클래스가 이를 둘러싼 메서드에 대한 로컬 변수를 캡처할 때 이 문제가 발생했다.
- 때문에, 컴파일러가 이를 확인하도록 로컬 변수앞에 final 키워드를 추가했어야만 했다.
- 변수를 final로 선언하면 컴파일러는 변수가 final이라고 인식할 수 있다.

### Local variable Capture

- Local variable은 조건이 final, effectively final이여야 한다.
    - effectively final은 JDK 1.8에 추가된 syntactic sugar의 일종으로, 초기화된 이후 값이 한 번도 변경되지 않았다는 것이다.
    - effectively final 변수는 final 키워드가 붙어있지 않지만 final 키워드를 명시한 것과 동일하게 컴파일단계에서 처리한다.
- 결론적으로 초기화하고 값이 변경되지 않은 것을 의미하는데, 왜 이런 제약조건이 있는 것일까?
    - 먼저 local variable은 스레드끼리 공유가 되지 않는다.
    - JVM에서 인스턴스 변수는 Heap 영역에 생성되며, 인스턴스 변수는 스레드끼리 공유가 가능하다.
    - 즉 local variable가 stack 영역에 저장되기 때문에, 람다 표현식에서 값을 바로 참조하는 것에 대한 제약이 있어복사된 값을 사용한다.
    - 때문에 멀티 스레드 환경에서 변경사항이 발견되면 동시성에 대한 이슈에 대응하기가 힘들어진다.

```java
Supplier<Integer> increment(int start) {
    return () -> start++;
} // Variable 'start' is already defined in the scope
```

- start라는 local variable을 람다 표현식 내에서 수정하려고 하는 경우이다.
- 멤버 메서드 내부에서 클래스의 객체를 생성하여 사용할 경우, new 키워드를 사용한다.
    - 이는 Heap 영역에 객체를 생성한다는 의미로 자신을 감싸고 있는 멤버 메서드의 실행이 끝난 이후에도 Heap영역에 존재하므로 사용할 수 있지만, 이 멤버 메서드에 정의된 매개변수나 local variable은 Stack에 할당되어 메서드 실행이 끝나면 영역에서 사라져 사용할 수 없게 된다.
    - heap에 생성된 객체가 Stack의 변수를 사용하려고 할 때, 사용 시점에는 Stack에 더 이상 해당 변수가 존재하지 않을 수 있다는 것이다.
- 더 이상 존재하지 않는 변수를 사용하려고 할 때 발생하는 문제를 Java에서는 Variable caputer라고 하는 복사값을 사용해 해결하는 것이다.
- 즉 람다 표현식은 익명 구현 객체처럼 별도의 객체를 생성하거나 컴파일 결과 별도의 클래스를 생성하지 않으며, 람다 표현식 내부에서 사용하는 변수는 Variable caputure가 발생하며 이는 final, effectively final처럼 사용해야 한다는 것이다.
    - 여기서 effectively final은 final 키워드로 선언된 것은 아니지만 값이 한 번만 할당되어 final처럼 쓰이는 것을 의미한다.

## 메서드, 생성자 레퍼런스

### Default Method Reference(메서드 레퍼런스)

- 메서드를 간결하게 지칭할 수 있는 방법으로 람다 표현식이 쓰이는 곳에서 사용 가능하다.
- 일반 함수를 람다 형태로 사용할 수 있도록 해준다.

```java
// Constructor Reference
String::new // ClassName::new
() -> new String()

// Static Method Reference
String::valueOf // ClassName::staticMethodName
(str) -> String.valueOf(str)

// Instance Method Reference
x::toString // instanceName::instanceMethodName
() -> "TheWing".toString()

// Instance Method Reference 
String::toString // ClassName::instanceMethodName
(str) -> str.toString()
```

## 참고자료

- https://five-cosmos-fb9.notion.site/758e363f9fb04872a604999f8af6a1ae
- https://sujl95.tistory.com/76
