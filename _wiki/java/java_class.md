---
layout  : wiki
title   : Java class 
summary : 
date    : 2023-04-15 15:13:27 +0900
updated : 2023-04-15 16:33:19 +0900
tag     : java
resource: CC/E75EAF-D559-49C2-A44A-C25EB98C705E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/5) 5주차 과제

## Class 정의

> 객체지향 프로그래밍에서 데이터와 그 조작 절차인 메서드를 정리한 객체의 추형을 정의한 것

```java
// class 선언 (접근제어자 키워드 클래스명)
public class Car {     
    
    // 필드(field)
    private String modelName;  
    private int modelYear;   
    
    // 생성자(Constructure)
    Car(String modelName, int modelYear) { 
        this.modelName = modelName;
        this.modelYear = modelYear;
    }

    // 메서드(method)
    public String getModel() { 
        return this.modelYear + "년식 " + this.modelName + " " + this.color;
    }
}
```

- **필드(field)** 는 class에 포함된 변수(variable)를 의미한다.
  - 클래스 내 필드는 선언된 위치에 따라 클래스 변수(static variable), 인스턴스 변수(instance variable), 지역 변수(local variable)로 구분된다.
  
```java
public class Car {
    static int modelOutput; // 클래스 변수
    String modelName;       // 인스턴스 변수

    void method() {
        int something = 10; // 지역 변수
    }
}
```

- **생성자**는 객체 생성과 동시에 인스턴스 변수를 원하는 값을 초기화할 수 있는 생성자 메서드를 제공한다. 

- **메서드**는 특정 작업을 수행하기 위한 명령문의 집합이다.
  - 메서드를 통해 중복되는 코드의 반복적인 프로그래밍을 피할 수 있다.

## 객체 생성

```java
// 참조 변수 선언
Car myCar; // class명 객체참조변수명

// 인스턴스 생성
myCar = new Car(); // 객체 참조변수명 = new class명();

// 변수선언 및 인스턴스 생성를 동시에 하기
Car myCar = new Car();
```

## 메서드 정의

```java
// 메서드 정의법
접근제어자 반환타입 메소드이름(매개변수목록) { // 선언부
    // 구현부
}
```

- 접근 제어자 : 해당 메소드에 접근할 수 있는 범위를 명시
- 반환 타입(return type) : 메소드가 모든 작업을 마치고 반환하는 데이터의 타입을 명시
- 메소드 이름 : 메소드를 호출하기 위한 이름을 명시
- 매개변수 목록(parameters) : 메소드 호출 시에 전달되는 인수의 값을 저장할 변수들을 명시
- 구현부 : 메소드의 고유 기능을 수행하는 명령문의 집합

```java
// 메서드 호출
객체참조변수이름.메소드이름(); // 매개변수가 없는 메소드의 호출
객체참조변수이름.메소드이름(인수1, 인수2, ...); // 매개변수가 있는 메소드의 호출

// 예시
Car myCar = new Car();   // 객체를 생성
myCar.accelerate(60, 3); // myCar 인스턴스의 accelerate() 메소드를 호출
```

## 생성자 정의

```java
// 생성자 선언
클래스이름() { ... }  // 매개변수가 없는 생성자 선언
클래스이름(인수1, 인수2, ...) { ... } // 매개변수가 있는 생성자 선언

// 생성자 예시
Car(String modelName, int modelYear, String color, int maxSpeeds) {
    this.modelName = modelName;
    this.modelYear = modelYear;
    this.color = color;
    this.maxSpeed = maxSpeed;
    this.currentSpeed = 0;
}
```

```java
// 생성자 호출
class Car {
    private String modelName;
    private int modelYear;
    private String color;
    private int maxSpeed;
    private int currentSpeed;
    
    Car(String modelName, int modelYear, String color, int maxSpeed) {
        this.modelName = modelName;
        this.modelYear = modelYear;
        this.color = color;
        this.maxSpeed = maxSpeed;
        this.currentSpeed = 0;
    }
    
    public String getModel() {
        return this.modelYear + "년식 " + this.modelName + " " + this.color;
    }
}

public class Method02 {
    public static void main(String[] args) {
        Car myCar = new Car("아반떼", 2016, "흰색", 200); // 생성자 호출
        System.out.println(myCar.getModel()); // 2016년식 아반떼 흰색
    }
}
```

- Java의 모든 클래스에는 하나 이상의 생성자가 정의되어 있어야 한다.
- 하지만 특별히 생성자를 정의하지 않고 인스턴스 생성이 가능한데, 이를 기본 생성자라고 한다.
- class에 생성자가 하나도 정의되어 있지 않으면, Java Complier가 자동으로 기본 생성자를 제공한다.
- 기본 생성자는 매개변수를 하나도 갖지 않으며, 명령어도 포함하지 않는다.

## this 키워드

### this 참조 변수

> 인스턴스가 바로 자기 자신을 참조하는 데 사용하는 변수

```java
class Car {

    private String modelName;
    private int modelYear;
    private String color;
    private int maxSpeed;
    private int currentSpeed;
    
    Car(String modelName, int modelYear, String color, int maxSpeed) {
        this.modelName = modelName;
        this.modelYear = modelYear;
        this.color = color;
        this.maxSpeed = maxSpeed;
        this.currentSpeed = 0;
    }
    ...
}
```

- 생성자의 매개변수 이름과 인스턴스 변수의 이름이 같을 경우에는 인스턴스 변수 앞에 this 키워드를 붙여 구분해만 한다.
- this 참조 변수를 사용할 수 있는 영역은 인스턴스 메서드뿐이며, 클래스 메서드에서는 사용할 수 없다.

### this() 메서드

> 생성자 내부에서, 같은 클래스의 다른 생성자를 호출

```java
class Car {

    private String modelName;
    private modelYear;
    private String color;
    private int maxSpeed;
    private int currentSpeed;
    
    Car(String modelName, int modelYear, String color, int maxSpeed) {
        this.modelName = modelName;
        this.modelYear = modelYear;
        this.color = color;
        this.maxSpeed = maxSpeed;
        this.currentSpeed = 0;
    }
    
    Car() {
        this("소나타", 2012, "검정색", 160); // 다른 생성자를 호출
    }
    
    public String getModel() {
        return this.modelYear + "년식 " + this.modelName + " " + this.color;
    }
}

public class Method05 {
    public static void main(String[] args) {
        Car tcpCar = new Car(); 
        System.out.println(tcpCar.getModel());
    }
}
```


