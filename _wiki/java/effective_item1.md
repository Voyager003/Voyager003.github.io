---
layout  : wiki
title   : 생성자 대신 정적 팩토리 메서드를 고려하라
summary : 
date    : 2023-04-30 16:22:09 +0900
updated : 2023-05-01 20:22:37 +0900
tag     : java effectivejava
resource: FC/A84C2D-B514-4D9F-8F56-2049159AE754
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 정적 팩토리 메서드(Static Factory Method)

먼저 팩토리(Factory)에 대한 정의를 알아보자.

> 다른 객체를 생성하는 책임을 갖는 프로그램 요소

**팩토리**는 자신의 책임이 다른 객체를 생성하는 것인 프로그램 요소를 말한다.

그렇다면 정적 팩토리 메서드는 정적으로 인스턴스 생성의 역할을 하는 메서드라는 의미이다. 

Java에서는 보통 인스턴스 생성 시, public 생성자를 사용하여 객체를 생성하는 방법이 많이 사용된다.

item1에서는 생성자를 이용한 인스턴스 생성 외, 정적 팩토리 메서드를 사용해 해당 클래스의 인스턴스를 만드는 방법을 제시하는데 살펴보자.

### JDK의 정적 팩토리 메서드

item1에서 소개된 예는 java.util.Collections이다. 코드를 살펴보면

```java
public static void reverse(List<?> list){
    ...
}
public static void shuffle(List<?> list){
    ...
}
...
```

라이브러리를 살펴보면 수 많은 정적 팩토리 메서드를 제공하고 있다.

Collections뿐만 아니라, String과 Optional 클래스도 많은 팩토리 메서드를 제공한다. 

```java
public static <T> Optional<T> ofNullable(T value) {
    ...
}

public static String valueOf(Object obj) {
    ...
}

// 실제 사용 시
BigInteger answer = BigInteger.valueOf(42L);
```

- 정적(static)으로 선언했기 때문에, new BigInteger(...)를 은닉하여 인스턴스 생성 시 캡슐화가 가능한 것을 확인 가능하다.
- valueOf 메서드와 같이 직접 생성자를 통해 인스턴스를 생성하는 것이 아닌 메서드를 통해 생성하는 것을 정적 팩토리 메서드라 한다.

### Custom 정적 팩토리 메서드

```java
// 기존 생성자 방식
public class User {
    
    private String name;
    private String email;
    private String city;
    
    public User(String name, String email, String city) {
        this.name = name;
        this.email = email;
        this.city = city;
    }
}

// Custom 정적 팩토리 메서드 방식
public static User createWithDefaultCity (String name, String email) {
    return new User(name, email, "Rome");
}  
```

1) 정적 팩토리 메서드의 특징 중 하나는 메서드 네이밍을 통한 가독성을 높일 수 있다는 점이다.

new 라는 키워드를 통해 인스턴스를 생성하는 생성자는 내부 구조를 알고 있어야 목적에 맞게 인스턴스 생성이 가능하다.

하지만 정적 팩토리 메서드를 이용해 city라는 필드가 "Rome"이라는 Default값을 가지도록 설계하여, 메서드 네이밍(createWithDefaultCountry)을 통해 인스턴스 생성 목적을 담아낼 수 있다.

2) 호출할 때 마다 새로운 인스턴스를 생성할 필요가 없다

```java
// 생성자 사용
User user1 = new User("Rim", "abdc@gmail.com", "Seoul");
User user2 = new User("Kim", "zabc@gmail.com", "Rome");
User user3 = new User("Song", "ambc@gmail.com", "Tokyo");


// 정적 팩토리 메서드 사용
User user1 = User.createWithDefaultCity("Kim", "abcd@gmail.com");
User user2 = User.createWithDefaultCity("Rim", "zabc@gmail.com");
```

생성자를 통한 인스턴스 생성은 매번 new 키워드를 통해 호출하게 된다.

new 키워드는 생성 비용이 비싸기 때문에, 인스턴스를 캐싱해서 사용한다면 new를 계속 호출할 필요가 없어 불필요한 객체 생성을 피할 수 있다.

3) 하위 자료형 객체를 반환할 수 있다.

시험 점수에 따라 등급이 정해지는 시스템이 있다고 가정해보자.

```java
public class Basic extends Level {
}

public class Advanced extends Level {
}

public class Master extends Level {
}

public class Level {

    public static Level (int score) {
        if (score < 50) {
            return new Basic();
        } else if (score < 80) {
            return new Advanced();
        } else {
            return new Master();
        }
    }
}

// test
@Test
@DisplayName("하위 자료형 객체를 반환")
public void test3() throws Exception {

    int score = 90;
    Level level = Level.checkLevel(score);
    
    assertThat(level).isNotInstanceOf(Basic.class);
    assertThat(level).isNotInstanceOf(Advanced.class);
    assertThat(level).isInstanceOf(Master.class);
    
    assertThat(level).isInstanceOf(Level.class);
}
```

Basic, Advanced, Master 클래스가 Level이라는 부모 타입을 상속받고 있는 구조이다.

이는 정적 팩토리 메서드(Level.checkLevel)를 만들어 하위 타입의 객체를 반환할 수 있다.

4) 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

위의 코드의 연장선이다. 

Level.checkLevel메서드는 Level 클래스를 return 하도록 명시가 되어있지만, score의 값에 따라 하위 타입의 객체를 반환할 수 있었다. 이를 통한 유연한 설계가 가능하다.

5) 정적 팩토리 메서드 작성 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

item1에서는 JDBC(Java Database Connectivity)를 예로 든다.

```java
public static void main(String[] args) {

    String driverName = "com.mysql.jdbc.Driver";
	String url = "..."
	String user = "..."
	String password = "..."
    
	try {
		Class.forName(driverName);

		Connection connection = DriverManager.getConnection(url, user, password);

	} catch (ClassNotFoundException e) {
		e.printStackTrace();
	} catch (SQLException e) {
		e.printStackTrace();
	}
}
```

코드를 살펴보면 Class.forName이라는 메서드로 driverName만을 호출했지만, DriverManager가 connection을 연결한다. 

코드 안에서 어떤 작동을 하는 것일까?

```java
package com.mysql.jdbc;

import java.sql.SQLException;

public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    ...
    
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch {SQLException E) {
            throw new RuntimeException("...")
            }
    }
        ...       
}
```

먼저 Class.forName 메서드에 의해 문자열로 전달되는 com.mysql.jdbc.Driver라는 클래스가 메모리에 load된다.

이 후, 위의 코드에 의해 static 절의 DriverManager.registerDriver() 메서드를 통해 자신을 등록하여 실행 시, JDBC Driver를 자동으로 등록할 수 있다. static절에서 반환 인스턴스의 클래스가 존재하지 않지만, 정적 팩토리 메서드(registerDriver(new Driver());)를 통해 Driver 클래스로 반환할 수 있다.

등록한 JDBC Driver는 DB connection 생성 시점에 사용되며, Runtime전 까지는 어느 JDBC Drvier가 사용될 지 모르기 때문에 동적으로 Driver를 로딩하기 위해 리플렉션을 사용한다.

item1에서 제시한 장점 5가지를 살펴보았다. 그렇다면 정적 팩토리 메서드가 가지는 단점이 있을까?

### 단점 

1) 상속 시, public 혹은 protected 생성자가 필요하기 때문에, 정적 팩토리 메서드만 제공하면 하위 클래스를 만들 수 없다.

위에서 예시로 든 JDK의 Collections을 예를 들어보자. 

java.util.Collections의 구현 클래스들은 당연히 상속할 수가 없기도 하고, 상속을 할 필요가 없는 클래스이다. 

item1은 이를 상속대신 컴포지션을 사용하도록 유도하고, 불변 타입으로 만드려면 이러한 제약은 오히려 장점이라고 말하고 있다. 

2) 프로그래머가 찾기 어렵다.

공식 문서를 봐도 Constructors Summary와 Method Summary에 대해서는 상단에 먼저 설명이 되어있지만, 정적 팩토리 메서드의 경우 API 문서에서 특별하게 다뤄주지 않는다. 

이는 클래스, 인터페이스 문서 상단에 팩토리 메서드에 대한 문서를 제공하여 해결할 수 있다.

## Lombok을 이용한 사용법

```java
@RequiredArgsConstructor(staticName = "of")
@Getter
public class LombokUser {

    private final String name;
    private final String email;
    private final String city;
}

// test
@Test
@DisplayName("Lombok을 이용한 정적 팩토리 메서드 구현")
public void test4() throws Exception {

    LombokUser user = LombokUser.of("Park", "ab@gmail.com", "Rome");

    assertThat(user.getName()).isEqualTo("Park");
    assertThat(user.getEmail()).isEqualTo("ab@gmail.com");
    assertThat(user.getCity()).isEqualTo("Rome");
}
```

@RequiredArgsConstructor 애노테이션 선언 뒤, staticName(of)을 선언하면, 해당 name을 가진 정적 팩토리 메서드를 만들 수 있다.

## 참고자료

- 이펙티브 자바 3판
- https://johngrib.github.io/wiki/pattern/factory-method/ - 기계인간님의 팩토리 메서드 패턴
- https://johngrib.github.io/wiki/pattern/static-factory-method/#%EA%B0%9C%EC%9A%94 - 정적 팩토리 메서드
- https://www.youtube.com/watch?v=X7RXP6EI-5E&t=1433s&ab_channel=%EB%B0%B1%EA%B8%B0%EC%84%A0 - 백기선님의 강의
- https://www.baeldung.com/java-constructors-vs-static-factory-methods
- https://sysgongbu.tistory.com/95 - 장점 5번의 설명


