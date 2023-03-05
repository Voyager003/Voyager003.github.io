---
layout  : wiki
title   : 정적, 동적 바인딩 
summary : 
date    : 2023-03-05 20:37:20 +0900
updated : 2023-03-05 23:04:29 +0900
tag     : java terminology
resource: F4/1EBCEC-4D43-4972-9229-E33334177F88
toc     : true
public  : true
parent  : [[/terminology]]
latex   : false
---
* TOC
{:toc}

## Binding

> 프로그램에 사용된 구성 요소의 실제 값 또는 프로퍼티를 결정짓는 행위

- 각종 값들이 확정되어 더 이상 변경할 수 없도록 구속(Bind)상태가 되는 것을 의미한다.
- 프로그램 내 변수, 배열, 절차 등의 명칭 즉 식별자가 그 대상인 메모리 주소, 데이터형 혹은 실제 값으로 배정되는 것이 이에 해당된다.

## 특징  

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FGaqpy%2FbtrI1h6Hfc2%2FgvmDErWIIWKYfGZXimVHi1%2Fimg.png)

### Static Binding(정적 바인딩)

> 실행 시간(Runtime) 전에 일어나고, 실행 시간에는 변하지 않은 상태로 유지되는 binding

정적 바인딩(Static)의 경우 **Complie time**에 결정되는 binding을 의미한다.

예를 들어 int a = 1; 은 Data type이 int로 바인딩 되어있고, a라는 변수명이 바인딩(메모리에 값을 할당하는 것도 binding), 1이라는 값도 바인딩 되는 것이다.

Data type으로 int 형이 바인딩 되는 것은, compile할 때 메모리에 할당되므로 정적 바인딩이고, 변수 a 또한 컴파일 시 메모리에 할당되므로 정적 바인딩이 된다.

프로그램이 실행되도 변하지 않는 것이 특징이며, static(final, private 메서드도 마찬가지)으로 선언되는 것은 메모리를 한 번밖에 
할당하지 않기 때문에 컴파일 시 메모리에 할당된다.  

Java의 Overloading을 예로 들어보자.

```java
public class Test {
    public void show(String Name1, String Name2) {
        System.out.println("Name1: " + Name1);
        System.out.println("Name2: " + Name2);
    }

    public void show(String Name1) {
        System.out.println("Name1: " + Name1);
    }

    public static void main(String[] args) {
        MadPlay instance = new MadPlay();
        instance.show("hi", "hello");
        instance.show("hi");
    }
}
```

Java의 메서드 Overloading은 같은 이름의 메서드를 매개변수의 타입과 개수를 다르게 정의하여 메서드 구현하는 것이다.

Test 클래스에서 show()라는 메서드를 호출한다고 하면, 2개의 show() 메서드는 입력으로 받는 Argument 종류와 개수가 다르기 때문에 컴파일 과정에서 구분할 수 
있다. 이는 컴파일 과정에서 어떤 메서드를 호출할 지 결정되기 때문에 Overloading을 정적 바인딩이라고 한다.

### Dynamic Binding(동적 바인딩)

> 실행 시간(Runtime)에 이루어지거나 실행 시간에 변경되는 binding

Compiletime에 결정되는 정적 바인딩과는 달리 실행시간(RunTime)에 결정된다.

int a = 1; 이라는 코드에서 1이라는 값은 코드 실행 시에 할당되기 때문에 동적 바인딩이라고 한다.
Java에서는 메서드를 기본적으로 동적 바인딩하기 때문에 메서드 오버라이딩이 가능하다고 한다.

```java
class Parent{
	void method(){
        System.out.println("parent...."); 
    }
}

class Child extends Parent{
    
    @Override
    void method(){
        System.out.println("Child...");
    }
}

public class DynamicBindingTest {
    public static void main(String[] args) { 
        Parent parent = new Child();
        parent.method(); 
    }
}
```

Child 클래스는 Parent를 상속받아 method()를 오버라이딩 했다. 

참조 변수는 Parent 타입이지만 Child 객체이므로 Child가 오버라이딩한 메서드를 호출하고,
컴파일러는 method()를 수행하기 위해서는 실제 객체가 무엇인지 컴파일타임에는 판단할 수 없어 바인딩을 Runtime까지 미루게 된다.

이는 parent라는 객체가 student라는 객체가 될 수도 있기 때문에 타입을 확정지을 수 없고 컴파일러는 단지 객체의 타입오류를 확인할 뿐이다.

이처럼 실행 시간에 어떤 메서드를 호출할지가 정해지기 때문에 오버라이딩을 동적 바인딩이라한다.

## 참고자료
- http://www.tcpschool.com/php/php_oop_binding - 바인딩 설명
