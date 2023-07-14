---
layout  : wiki
title   : toString을 항상 재정의하라 
summary : 
date    : 2023-07-14 09:36:14 +0900
updated : 2023-07-14 11:23:53 +0900
tag     : java effectivejava
resource: 8D/1BD02B-3555-4527-9857-20D835A9519D
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Object의 toString()

Obejct의 기본 toString()는 개발자가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없고, 클래스이름@16진수해시코드를 반환한다. 다음을 보자.

```java
public class User {

    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

// 출력 
public static void main(String[] args) {
    User user = new User("name", 10);
    System.out.println(user); // User@1fb3ebeb
}
```

이처럼 의미를 알 수 없는 문자열을 반환한다.

```java
// println 
public void println(Object x) {
        String s = String.valueOf(x);
        if (getClass() == PrintStream.class) {
        ...
    }
}

// valueOf
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

이는 System.out.println()은 String.valueOf()로 패러미터로 받은 인스턴스를 문자열로 반환하고, String.valueOf()가 Object의 toString()을 호출하여 문자열을 반환하기 때문이다.

```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
```

이후 toString()에 따라 고유한 16진수 해시코드를 반환하게 된다.

이처럼 무의미한 해시코드를 개선하려먼 어떻게 해야할까?

## toString 일반 규약

item11에서 말하는 toString 일반 규약은 다음과 같다.

- 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.
- 모든 하위 클래스에서 toString()을 재정의해야 한다.

toString을 재정의함으로써 시스템은 디버깅하기 쉬워진다.

toString을 직접 호출하지 않더라도 println, printf에 넘길 때 혹은 디버거가 인스턴스를 출력할 때 자동으로 호출된다. 작성한 인스턴스를 참조하는 컴포넌트가 오류 메시지를 logging할 때 toString을 제대로 재정의했다면 다음 간단한 코드만으로 문제를 진단하기에 충분한 메시지를 남길 수 있다.

```java
System.out.println(phoneNuber + "에 연결할 수 없습니다.");
```

toString은 그 인스턴스가 가진 주요 정보 모두를 반환하는 것이 좋다.

```java
@Override
public String toString() {
    return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

// User{name='name', age=10}
```

하지만 인스턴스가 거대하거나 인스턴스의 상태가 문자열로 표현하기에 적합하지 않다면 무리가 있다. 이러한 경우 요약 정보를 담는 것이 좋다.

또한 반환값의 포맷을 문서화할지 정해야 한다.

포맷을 명시한다면 해당 인스턴스는 표준적이고 명확하겠지만, 그 포맷에 얽매일 수 있다.

```java
// 포맷 명시 O
@Override
public String toString() {
    return String.format("%...", prefix, lineNum)
}

// 포맷 명시 X
/**
 * 이 약물에 관한 대략적인 설명을 반환한다.
 * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
 * "[약물 #9: 유형=사랑, 냄새=테레빈유, 겉모습=먹물]"
 */
@Override public String toString() {
    ... 
}
```

포맷을 명시하든 아니든 의도를 정확하게 밝혀 바뀔 여지가 있음을 인식시킬 수 있다. 

포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

이 때, 정적 유틸리티 클래스(item4), 열거 타입(Enum item34)의 경우 Java에서 완벽하게 toString을 제공하기 때문에 따로 재정의할 필요 없다.

하지만 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스라면 toString을 재정의해줘야한다.

```java
public abstract class Shape {
    protected String color;
    protected int x;
    protected int y;
    
    public Shape(String color, int x, int y) {
        this.color = color;
        this.x = x;
        this.y = y;
    }
    
    // Getter, Setter 등의 코드 생략
    
    @Override
    public String toString() {
        return "Color: " + color + ", Position: (" + x + ", " + y + ")";
    }
}

```

Shape라는 추상 클래스에서 도형의 색과 위치 정보를 문자열로 반환하는 toString()을 재정의했다. 

```java
public class Circle extends Shape {
    private int radius;
    
    public Circle(String color, int x, int y, int radius) {
        super(color, x, y);
        this.radius = radius;
    }
    
    // Getter, Setter 등의 코드 생략
    
    @Override
    public String toString() {
        return super.toString() + ", Radius: " + radius;
    }
}
```

Circle 클래스가 Shape를 상속받는다면 Circle클래스에서도 toString()을 정의해 반지름 정보를 추가할 수 있다.

이처럼 하위 클래스들은 공통된 문자열 표현을 상속받고 필요에 따라 추가 정보를 포함시킬 수 있다. 상위 클래스인 Shape에서 정의된 toString() 메서드를 재정의해 하위 클래스에서 원하는 문자열을 구현할 수 있게 된다.

대다수의 컬렉션 구현체는 추상 컬렉션 클래스들의 toString()을 상속해 사용한다.

## 참고자료

- 이펙티브 자바 3판



