---
layout  : wiki
title   : Java Interface 
summary : 
date    : 2023-05-02 12:02:40 +0900
updated : 2023-05-02 21:01:41 +0900
tag     : java
resource: 9E/66F084-39BF-40B9-A486-08D8718879DE
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/8) 8주차 과제

## Interface(인터페이스) 정의 및 구현

> 클래스 작성 시, 기본이 되는 틀을 제공하면서, 다른 클래스 사이의 중간 매개 역할까지 담당하는 추상 클래스이다.

굉장히 추상적인 설명이기 때문에, 구현을 통해 살펴보자.

### 정의

```java
// 형식
public interface interfaceName extends [부모 인터페이스 ...]{
    ...
}
```

- 인터페이스 선언은 예약어로 class 대신 'interface' 키워드를 사용한다.
- 접근 제어자로 public, default를 사용한다.
- extends 키워드로 다중 상속이 가능하다.

### 구현

```java
// interface 코드 작성
public interface interfaceName {
    
    // 상수
    타입 상수명 = 값;
    
    // 추상 메서드
    타입 메서드명(매개변수, ...);
    
    // Default 메서드
    default 타입 메서드명(매개변수, ...){
    }
    
    // 정적 메서드
    static 타입 메서드명(매개변수, ...){
    }
}

// 구현
public class HiInterface implements interfaceName, ... {
    
    @Override
    ... interface에서 작성한 code...
    }
}
```

- 상수는 static final이 붙어야 하지만 생략가능하며, 그 의미(상수)는 유지된다. (컴파일러가 처리)
- public 추상 메서드를 위한 public abstract도 생략 가능하며, 그 의미는 유지된다. (역시 컴파일러가 처리)
- 인터페이스 구현 시, implements 키워드를 이용하며 다중 상속이 가능하다. 

### 익명 구현객체(Anonymous Object)

인터페이스는 원칙적으로 객체화 될 수 없지만, **익명 객체**를 통해 인터페이스를 구현하는 일회용 객체를 만들 수 있다.

다음과 같은 인터페이스가 있다고 가정해보자.

```java
public interface MyInterface {
    public void doSomething();
}
```

이 인터페이스를 구현하는 익명객체를 생성하려면 다음과 같다.

```java
MyInterface myObject = new MyInterface() {
    
    @Override
    public void doSomething() {
        System.out.println("Something is being done");
    }
};
```

Myinreface는 구현할 인터페이스를 나타내며, 중괄호{} 안에 doSomething()메서드의 구현 코드가 작성된다.

이 때, 인터페이스에 선언된 추상 메서드를 실체 메서드로 선언해야하며, 그렇지 않다면 오류가 발생한다.

myObject 인스턴스는 Myinterface를 구현하는 익명 객체로 생성되며, 생성된 인스턴스는 변수에 대입하여 사용할 수 있다.

```java
myObject.doSomething(); // "Something is being done"
```

이는 단순 인터페이스의 구현 역할을 하는 익명 객체일 뿐, 상속이나 또 다른 인터페이스를 구현하는 것은 불가능하다.

## reference(레퍼런스)를 통해 구현체를 사용하는 방법

```java
interface Runnable{
    void run();
}

interface Jumpable {
    void jump();
}

class Human implements Runnable, Jumpable {

    @Override
    public void run() {
        System.out.println("Human's run");
    }
    
    @Override
    public void jump() {
        System.out.println("Human's Jumping");
    }
}
```

Human이라는 인스턴스를 사용한다고 해보자.

```java
Runnable human = new Human();
human.run();
human.jump(); // Cannot resolve method 'jump' in 'Runnable'
```

Runnable을 통해 human을 선언하면, Runnable가 가질 수 있는 메서드만을 사용할 수 있다. (컴파일 에러)

## 인터페이스 상속

```java
interface [인터페이스 명] extends [부모 인터페이스명 ...] {
}
```

- 클래스와 달리 extneds 키워드를 사용하여, 다중 상속이 가능하다.
- 이는 인터페이스의 메서드는 추상 메서드로 구현하기 전의 메서드이기 때문에, 어떤 인터페이스의 메서드를 상속받아도 같기 때문이다.
- 자식 인터페이스는 상속된 인터페이스가 가진 메서드까지 모두 구현해야 한다.

### 추상 클래스와 인터페이스의 차이

- 추상 클래스는 일반 메서드와 추상 메서드 둘 다 가질 수 있다.
    - 인터페이스는 추상 메서드와 상수만을 가지며, 구현 로직을 작성할 수 없다.
    - but 하기에 소개될 Java 8의 default, static 메서드를 작성할 수 있다.
- 인터페이스 내 존재하는 메서드는 무조건 public abstract로 선언되며 생략 가능하다.
- 인터페이스 내 존재하는 변수는 무조건 public static final로 선언되며, 생략 가능하다.
- 추상 클래스는 '~는 ~이다'의 개념이다.
    - 하지만 인터페이스는 '~는 ~를 할 수 있다'의 개념이다.

## 인터페이스의 default(기본) 메서드(Java 8)

- **기본 메서드**는 메서드 선언이 아닌 '구현체'를 제공하는 방법으로, 해당 인터페이스를 구현한 클래스의 어떠한 영향없이 새로운 기능을 추가하는 방법이다.
- 기본 메서드는 해당 인터페이스를 구현한 구현체가 모르게 추가되는 기능이다.
    - 컴파일 에러는 발생하지 않지만, 특정 구현체의 로직에 따라 런타임 에러가 발생할 수 있다. (최고의 에러는 컴파일에러임을 잊지 말자!)
    - 사용 시, 구현체가 잘못 사용하지 않도록 문서화가 필요하다.
- Object가 제공하는 equlas, hashCode와 같은 기본 메서드는 제공하지 않지만, 구현체가 재정의하여 사용할 수 있다.

```java
public interface Foo {
    void printName();
}

public class DefaultFoo implements Foo {
    
    @Override
    public void printName() {
        System.out.println("DefaultFoo");
    }
}
```

Foo 인터페이스를 구현한 DefaultFoo가 있다. 

이 때, Foo 인터페이스를 구현한 모든 구현체에 공통적인 기능을 제공해야 하는 요구사항이 발생했다고 가정해보자.

추가된 요구사항에 대한 메서드는 printNameUpperCase();라 가정한다.

```java
public interface Foo {
    void printName();
    void printNameUpperCase();
    }
}
```

Foo 인터페이스에 printNameUpperCase() 메서드가 추가되었는데, 이때 Foo를 구현한 구현체들은 모두 컴파일 에러가 발생한다.

이는 추가된 인터페이스의 메서드를 구현하지 않았기 때문이며, 이 때 구현한 구현체들에게 영향을 받지 않고 메서드의 기능을 제공하고 싶다면 기본 메서드를 이용하여 해결할 수 있다.

```java
public interface Foo {
    void printName();

    default void printNameUpperCase(){
        System.out.println(getName().toUpperCase());
    }

    String getName();
}

public class FooMain {

    public static void main(String[] args){
    DefaultFoo foo = new DefaultFoo("Rome");

    foo.printName();
    foo.printNameUpperCase();
    }
}
```

기본 메서드를 추가한 코드이다. Foo를 구현한 구현체인 DefaultFoo에서는 추가 메서드 구현없이 사용할 수 있게 되었다.

이제 특징에서 살펴봤던 주의점에 대해 살펴보자. 

getName()이 어떻게 구현되어 있는지 모르는 상황에서 기본 메서드에서 getName() 메서드를 호출하는 로직을 구현했다.

이처럼 기본 메서드가 어떻게 구현되어 있는지 알 수 없기 때문에 문서화 및 주석을 통해 이 점을 알려야 한다.

Java 8 이전에는 인터페이스가 가질 수 있는 메서드는 추상 메서드 뿐으로 인터페이스의 구현 클래스들이 메서드 구현 코드를 작성할 때, 내용이 일치하더라도 클래스마다 새로 작성해야 한다는 문제점이 있었다.

하지만 소개된 기본 메서드가 도입되어 인터페이스 내부에 존재할 수 있는 구현 메서드가 되었고, 인터페이스를 implements 하면 구현없이 바로 사용할 수 있게 되었다.

## 인터페이스의 static 메서드(Java 8)

```java
static {반환타입} {메서드명} ({매개변수...}) {
    ...
}
```

인터페이스의 정적 메서드는 인터페이스 내에서 이미 body를 구현한 메서드이다. 하지만 구현 클래스에서 Overriding하여 사용할 수는 없다.

정적 메서드를 통해 인스턴스 없이 수행할 수 있는 작업을 정의할 수 있다. 

```java
public interface MyInterface {
    public static void myStaticMethod() {
        System.out.println("This is a static method in an interface!");
    }
}

public class MyClass implements MyInterface {
    public void myMethod() {
        MyInterface.myStaticMethod(); // "This is a static method in an interface!" 출력
    }
}
```

인터페이스에서 정적 메서드를 정의하면, 해당 메서드는 인터페이스를 구현하는 모든 클래스에서 공유하여 사용할 수 있으며, 해당 메서드는 인터페이스를 구현하는 클래스에서 Override할 수 없기 때문에, 일종의 '유틸리티 메서드'로 사용된다.

```java
public interface List<E> extends Collection<E> {
    ...
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5, e6);
    }
}
```

List 인터페이스의 메서드 중 하나인 of()이다. 인터페이스의 정적 메서드는 이와 같이 유틸리티 메서드를 전달할 때 사용된다.

[정적 팩토리 메서드](https://voyager003.github.io/wiki/java/effective_item1/)와 유사해 보이지만, 정적 팩토리 메서드의 경우 객체를 생성하고
인터페이스의 정적 메서드는 인터페이스와 관련된 유틸리티 메서드를 제공한다는 차이가 있다.

## 인터페이스의 private 메서드(Java 9)

Java 9에서는 추가적으로 private method와 private static method가 추가되었다.

Java 8의 기본 메서드는 단지 특정 기능을 처리하는 내부 메서드일 뿐이지만, 외부에 공개되는 public method로 만들어야 하기 불편함이 있었다.

이는 인터페이스를 구현하는 다른 인터페이스나 클래스가 해당 메서드에 접촉하거나 상속할 수 있는 것을 원치 않지만 그럴 가능성이 있는 것이다. 

이는 다음과 같은 규칙을 가진다.

```java
interface CustomCalculator{
	default int addEvenNumbers(int... nums) {
		return add(n -> n % 2 == 0, nums);
	}

	default int addOddNumbers(int... nums) {
		return add(n -> n % 2 != 0, nums);
	}

	private int add(IntPredicate predicate, int... nums) {
		return IntStream.of(nums)
                    .filter(predicate)
                    .sum();
	}
}
```

- 먼저 private 메서드는 구현부를 가져야만하며, 오직 인터페이스 내부에서만 사용할 수 있다.
- private static 메서드는 다른 static 혹은 static이 아닌 메서드에서 사용할 수 있다.
- static이 아닌 private 메서드는 다른 private static 메서드에서 사용할 수 없다.

인터페이스 내부에 private을 허용함으로써, 외부에 공개하지 않으면서 코드의 중복을 피할 수 있게 되었다.

## 참고자료

- https://five-cosmos-fb9.notion.site/4b0cf3f6ff7549adb2951e27519fc0e6 
- https://slime-television-a49.notion.site/8-0cc8c251d5374ac882a4f22fa07c4e6a