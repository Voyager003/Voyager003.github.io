---
layout  : wiki
title   : Java Enum 
summary : 
date    : 2023-05-17 21:20:40 +0900
updated : 2023-05-17 22:35:01 +0900
tag     : java
resource: 24/C58A3D-9B36-42A4-84E0-F2658C665881
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/11) 11주차 과제 

## enum 정의하는 방법

> Enum은 열거형(enumeration)을 나타내는 특별한 데이터 유형으로 상수의 집합으로 구성되며, 주로 연관된 상수 그룹을 정의하는 데 사용된다.

```java
// 선언
enum Season {
    SPRING, SUMMER, FALL, WINTER
}

// 사용
public class EnumEx {
    public static void main(String[] args) {
        printSeason(Season.SPRING);
    }

    public static void printSeason(Season season) {
        switch (season) {
            case SPRING -> System.out.println("봄");
            case SUMMER -> System.out.println("여름");
            case FALL -> System.out.println("가을");
            case WINTER -> System.out.println("겨울");
            default -> throw new IllegalArgumentException("계절의 이름이 아닙니다.");
        }
    }
}      
// 실행 결과
봄 
```

### enum의 특징

- enum에 정의된 상수들은 해당 enum type의 객체이다.
    - C와 같은 언어에서 enum이 존재하지만, Java의 enum은 단순 정수 값이 아닌 enum type의 객체이다.
- 생성자와 메서드를 가질 수 있다.
    - enum은 엄연한 클래스로 생성자와 메서드를 가질 수 있다.
    
    ```java
    enum Currency {

    PENNY(1), NICKLE(5), DIME(10), QUARTER(25);

    private int value;

    Currency(int value) {
        this.value = value;
    }

    public int value() {
        return value;
        }
    }
    ```

### enum 사용 이유

- enum의 상수 값 정의는 의미있는 이름을 사용하기 때문에 코드의 가독성이 향상된다.
    - 상수 값을 직접 사용하는 대신 enum 상수를 사용하면 보다 이해하기 쉽다.
- 열거된 값만을 허용하도록 컴파일러가 상수 값의 유효성을 검사하여 잘못된 값이나 타입 오류를 방지할 수 있다.
- 연관된 상수 값을 묶어 하나의 유닛으로 정의할 수 있다.
    - 이는 관련된 상수 값들을 그룹화하고, 함께 사용되어야 하는 값들이 함께 유지될 수 있다.
- 컴파일 타임에 모든 상수 값을 알고 있으므로 런타임 중에 예상치 못한 값이나 오류가 발생하는 것을 방지할 수 있다.
- API에서 고정된 상수 값을 나타내는 데 사용될 수 있다.
    - 예를 들어 HTTP 상태 코드나 애플리케이션의 설정 상태 등을 enum으로 정의하여 사용하면 API 사용자가 상수 값의 의미를 이해하기 쉬워진다.

## enum이 제공하는 메서드

- T[] values() : 해당 enum 타입에 정의된 상수 배열을 반환한다.
- T valueOf(Class enumType, String name) : 지정된 enum에서 name과 일치하는 열거형 상수를 반환한다.
- int ordinal() : enum 상수가 정의된 순서를 반환한다.
- String name() : enum 상수의 이름을 문자열로 반환한다.
- Class<E> getDeclaringClass() : enum의 객체를 반환한다.

```java
enum Season {
    SPRING, SUMMER, FALL, WINTER;
}

public class EnumEx {
    public static void main(String[] args) {
        for (var season : Season.values()) {
            System.out.println(season.name());
        }
    }
}

// 실행 결과
SPRING
SUMMER
FALL
WINTER
```

## java.lang.Enum

```java
public abstract class Enum<E extends Enum<E>>
        implements Constable, Comparable<E>, Serializable {

    private final String name;

    private final String name() {
        return name;
    }
    ...
}
```

- java.lang에 포함된 Enum 클래스는 모든 Java Enum의 조상이다.
- 모든 enum은 Enum 클래스를 상속받기 때문에 enum type은 별도의 상속을 받을 수 없다.

## EnumSet

![](https://media.geeksforgeeks.org/wp-content/uploads/20200911115830/EnumSetinJava.png)

EnumSet 열거형을 위해 특별히 설계된 Set 인터페이스 구현체로 특징은 다음과 같다.

- EnumSet은 Abstract 클래스를 상속하고 Set 인터페이스를 구현한다.
- 오직 Enum 상수만을 값으로 가질 수 있으며, 모든 값은 같은 enum type이어야 한다.
- null value를 허용하지 않으며, NullPointerException throw도 허용하지 않는다.
- ordinal 값의 순서대로 요소가 저장된다.
- thread-safe하지 않아 동기화가 필요하다.

## 참고자료
- https://www.geeksforgeeks.org/enumset-class-java/ - EnumSet
- https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html - java docs
