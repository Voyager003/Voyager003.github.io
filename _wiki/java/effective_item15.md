---
layout  : wiki
title   : 클래스와 멤버의 접근 권한을 최소화하라 
summary : 
date    : 2023-07-31 09:21:52 +0900
updated : 2023-07-31 11:48:11 +0900
tag     : java effectivejava
resource: 9A/0F95D5-9575-4889-AC03-82D73F7D0B3D
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 잘 설계된 컴포넌트의 특징, 캡슐화

어설프게 설계된 컴포넌트와 잘 설계된 컴포넌트의 가장 큰 차이는 클래스 내부 데이터와 내부 구현 정보를 외부 컴포넌트로부터 얼마나 잘 숨겼느냐이다. 

(여기서 컴포넌트(Component)는 여러 개의 프로그램 함수들을 모아 하나의 특정한 기능을 수행할 수 있도록 구성한 작은 기능적 단위를 말한다.)

모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리하여 오직 API를 통해 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다.

item15에서 말하는 캡슐화(정보은닉)의 장점은 다음과 같다.

1) 여러 컴포넌트를 병렬로 개발하여 시스템 개발속도를 높인다.

2) 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담이 적어 시스템 관리 비용을 낮출 수 있다.

3) 완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 뒤, 다른 컴포넌트에 영향을 주지않고 해당 컴포넌트만 최적화가능하다.

4) 외부에 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면, 그 컴포넌트와 함께 개발되지 않는 낯선 환경에서도 유용하게 쓰일 가능성이 있다. 즉, 재사용성을 높일 수 있다.

5) 시스템 전체가 완성되지 않은 상태에서 개별 컴포넌트의 동작을 검증할 수 있어 큰 시스템을 제작하는 난이도를 낮출 수 있다.

Java에서는 캡슐화를 위한 다양한 장치를 제공한다.

## 접근 제한자(Access Modifier)

접근 제한자를 제대로 활용하는 것이 캡슐화의 핵심으로, 기본 원칙은 '모든 클래스와 멤버의 접근성을 가능한 좁혀야 한다.'는 것이다.

톱레벨 클래스 혹은 인터페이스를 public으로 선언하면 공개 API가 되며, package-private으로 선언하면 해당 패키지 안에서만 사용할 수 있다. 

패키지 외부에서 사용할 이유가 없다면 package-private으로 선언하라. 그렇다면 API가 아닌 내부 구현이 되어 클라이언트에 아무런 영향 없이 다음 release에서 수정, 교체, 제거할 수 있다. public으로 선언시에는 API가 되므로 영원히 관리 대상이 된다.

## private static 중첩 클래스

한 클래스에서만 사용하는 package-private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static으로 중첩시키자.

```java
// package-private access level
class OuterClass {

    // OuterClass에서만 접근 가능한 중첩(private static nested) 클래스
    private static class NestedClass {
        private void doSomething() {
            System.out.println("NestedClass에서 작업 수행");
        }
    }

    void outerMethod() {
        System.out.println("OuterClass의 outerMethod 호출");

        // 중첩 클래스의 인스턴스 생성 및 메서드 호출
        NestedClass nested = new NestedClass();
        nested.doSomething();
    }
}

public class Main {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.outerMethod();
    }
}
```

OuterClass를 package-priavet 톱레벨 클래스라고 해보자. 이 클래스 안에 private static으로 NestedClass라는 중첩 클래스를 정의했다. 

NestedClass는 OuterClass 내부에서만 사용할 수 있으며, 외부 클래스나 패키지에서는 접근할 수 없다. 또한 NestedClass는 private으로 선언되어 다른 클래스나 패키지에서 직접적으로 접근할 수 없다. 

중첩 클래스를 선언함으로써 NestedClass는 OuterClass에 강하게 결합되어 있어 OuterClass의 내부 구현을 변경할 때, NestedClass도 수정되어야 한다.

이를 통해 외부에서 접근하지 않고 OuterClass의 내부 구현에만 사용되는 코드를 캡슐화하여 클래스의 설계를 더 견고하고 유지보수하기 쉬운 구조로 만들 수 있다.

## 멤버 필드

클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만들자.

이후, 오직 같은 패키지의 다른 클래스가 접근해야 하는 멤버에 한해 package-private으로 접근 수준을 풀어주자.

권한을 풀어주는 일이 잦다면 컴포넌트 분해 여부를 좀더 고민해보자. Serializable을 구현한 클래스에서는 package-private, private 멤버가 의도치 않게 공개 API가 될 수 있으니 조심하자. 

public 클래스에서 package-private에서 protected로 바꾸는 순간 그 멤버에 접근할 수 있는 대상 범위가 엄청 넓어지게 된다. public의 protected 멤버는 공개 API로 영원히 지원되어야 하며 내부 동작 방식을 API 문서에 적어 공개해야할 수 있따. 따라서 protected 멤버의 수는 적을수록 좋다.

하지만 멤버 접근성을 못하게 방해하는 제약이 있다. 

바로 상위 클래스의 메서드르를 재정의할 때 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다는 것이다. 이 제약은 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수있어야 한다는 리스코프 치환원칙을 지키기 위해 필요하다.

## 주의점

### 테스트 코드만을 위해 클래스, 인터페이스 멤버를 공개 API로 만들면 안된다.

테스트 코드를 작성하려는 목적으로 클래스, 인터페이스, 멤버의 접근 범위를 조정하는 경우 private멤버를 package-private까지 풀어주는 것까지만 허용하며 그 이상은 안된다. 

### public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.

```java
public class Person {

    public String name;
    pulbic int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    ...
    
```

Person 클래스의 필드 name과 age를 public으로 선언하면 필드가 가변 객체를 참조하거나 final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃는다. 또한 public 가변 필드를 갖는 클래스는 일반적으로 Thread-safe하지 않다. 

```java
@Getter
public class Person {

    private String name;
    private int age;
    
    ...
```

private 접근 제어자를 사용해 외부에서 직접 접근할 수 없도록 변경했다. 이는 클래스 내부의 상태에 대한 접근을 Getter 및 Builder 패턴 등을 통해서 변경할 수 있다. 메서드를 통해 간접적으로 접근함으로써 데이터의 무결성을 보호할 수 있다.

### 정적 필드에서의 public 필드 예외

위의 경우에는 정적 필드에서도 마찬가지이지만, 한 가지 예외가 있다.

해당 클래스가 표현하는 추상 개념을 완성하는 데 꼭 필요한 구성요소로써의 상수라면 public static final 필드로 공개해도 좋다. 이는 관례상 대문자 알파벳으로 쓰며 단어 사이에 (_)를 넣는다. 이런 필드는 반드시 기본 타입 값이나 불변 객체를 참조해야 한다.

```java
public class Constants {

    // 상수 정의
    public static final int MAX_COUNT = 100;
    public static final String DEFAULT_NAME = "ROME";
    public static final double PI = 3.14159;

    // 불변 객체를 참조하는 경우
    public static final List<String> COLORS = Collections.unmodifiableList(Arrays.asList("Red", "Green", "Blue"));
}
```

public static final 필드를 사용해 상수를 정의함으로써, 외부에서 접근하여 값을 변경할 수 없도록 보장할 수 있다.

이 때 주의할 점은 이 필드를 반환하는 접근자 메서드를 제공해선 안된다. 이는 클라이언트에서 배열의 내용을 수정할 가능성이 있기 때문이다. 이를 해결하려면 다음과 같은 방법이 있다.

```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

pulbic 배열을 private으로 만들고 public 불변 리스트를 추가하는 것과

```java
private static final Thing[] PRIVATE_VALUES = {...};
public static final Thing[] values(){
  return PRIVATE_VALUES.clone();
}
```

배열을 priavate으로 만들고 그 복사본을 반환하는 public 메서드를 추가하는 방법이다.(방어적 복사)

첫 번째 방법은 클래스 외부에서 직접 접근할 수 없어 배열의 불변성을 보장할 수 있으며, 배열을 불변 리스트로 Wrapping하고 있으므로 외부에서 내용을 변경할 수 있다. 

두 번째 방법 역시 배열의 불변성을 보장할 수 있으며, values()를 통해 배열의 복사본을 반환하므로 외부에서는 복사된 배열을 사용하게 된다. 이는 외부에서 반환된 배열을 변경해도 원본 배열에는 영향을 주지 않는다.

반환 타입과 성능을 고려해 사용하자.

## Java 9 Module

Java 9에 추가된 도입된 개념으로 두 가지 암묵적 접근 수준이 추가됐다.

Package의 경우 클래스의 묶음이라면, Module은 패키지의 묶음이다.

Module은 자신에 속하는 패키지 중에 공개(export)할 것들을 선언하여(이 때, 관례상 module-info.java 파일에 선언한다.) 클래스를 외부에 공개하지 않으면서 같은 모듈을 이루는 Package 사이에서 자유롭게 공유할 수 있다. 이 때 해당 패키지를 공개하지 않았더라면 protected, public 멤버일지라도 Module 외부에서는 접근이 불가능하다.

주의점은 Module의 jar 파일을 자신의 모듈 경로가 아닌 애플리케이션 클래스 패스에 둔다면 그 Module 안의 모든 package는 마치 Module이 없는 것처럼 동작한다. 

Module이 공개(export)했는지 여부와 상관 없이 public 클래스가 선언한 모든 public, protected 멤버를 Module 밖에서도 접근할 수 있게 된다.

Module의 모든 장점을 누리기 위해서는 패키지들을 모듈 단위로 묶고, Module 선언에 패키지들의 모든 의존성을 명시해야한다. 이 후, 소스 트리를 재배치하고, Module 안에서 일반 패키지로의 모든 접근에 특별한 조치를 취해야 한다. 그러므로, item15는 꼭 필요한 경우가 아니라면 당분간은 사용하지 않는 것이 좋고 말한다.

Module과 관련한 내용은 따로 공부하여 정리해봐야겠다.

## 참고자료

- 이펙티브 자바 3판









