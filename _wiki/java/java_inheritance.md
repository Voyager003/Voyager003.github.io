---
layout  : wiki
title   : Java의 상속 
summary : 
date    : 2023-04-21 20:45:08 +0900
updated : 2023-04-21 23:31:29 +0900
tag     : java
resource: 79/31718F-DFE6-418C-A221-FC9F4169A45D
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/6) 6주차 과제

## 상속(Inheritance)의 특징

> 부모 클래스(super,parent class)로부터 자식 클래스(sub,child class)가 속성(변수)과 메서드를 물려받는 것

- 상속 횟수에는 제한이 없지만, 단일 상속만 가능하다.

```java
// 단일 상속
class subClass extends superClass1 {
    
}

// 단일 상속이 아닌 경우
class subClass extends superClass1, superClass2 {

}

// 상속 횟수는 제한이 없다
class A{
    
};

class B extends A{
    
};

class C extends B{
    
};
```

- Java 계층 구조 최상위에는 java.lang.Object 클래스가 존재한다.

![alt](https://docs.oracle.com/javase/tutorial/figures/java/classes-object.gif)

- 부모 클래스의 메서드와 변수만 상속되며, 생성자는 상속되지 않는다. 
  - 부모 클래스의 메서드는 Overriding을 통해 재정의하여 사용한다.

## super 키워드

> 부모 클래스의 생성자 혹은 메서드를 호출하는 것

```java
class Parent{
   int a = 10;
}

class Child extends Parent{
    int a = 20;
    
    void childMethod(){
        System.out.println(a);        //20
        System.out.println(super.a);  //10
    }
}
```

super 키워드로 부모 클래스 Parent에 선언된 int a를 가져와, 자식 클래스 Child에서 출력할 수 있다.

```java
// 생성자 호출
class Parent{
  int a;
  int b;

  public Parent(int a, int b) {
    this.a = a;
    this.b = b;
  }
}

class Child extends Parent{
  int c;

  public Child() {
    this(1, 2, 3);
  }

  public Child(int a, int b, int c) {
    super(a, b);
    this.c = c;
  }
}
```

부모 클래스의 생성자를 호출하여, 자식 클래스의 속성을 초기화할 수 있다.

## 메서드 오버라이딩(method overriding)

> 부모 클래스의 인스턴스 메서드와 같은 시그니처(이름, 매개변수의 번호 및 유형을 포함) 및 반환 유형이 동일한 서브클래스의 인스턴스 메소드는 
> 부모 클래스의 메소드를 재정의한다. [^1]

```java
public class Animal {
  public void makeSound() {
    System.out.println("Animal is making a sound");
  }
}

public class Dog extends Animal {
  @Override
  public void makeSound() {
    System.out.println("Woof!");
  }
}

public class Main {
  public static void main(String[] args) {
    Animal animal = new Animal();
    animal.makeSound(); // 출력 결과: "Animal is making a sound"

    Dog dog = new Dog();
    dog.makeSound(); // 출력 결과: "Woof!"
  }
}
```

Animal 클래스에 makeSound 메서드가 정의되어 있다. 

Dog 클래스는 Animal 클래스를 상속받았고, makeSound 메서드를 오버라이딩하여 재정의했다. 

Main 클래스에서 Animal과 Dog 객체를 생성하고 각 객체의 makeSound 메서드를 호출하여 출력한 뒤 확인해보면

Animal 객체는 원래 makeSound 메서드의 내용대로 "Animal is making a sound"를 출력하지만,
Dog 객체는 오버라이딩한 makeSound 메서드의 내용대로 "Woof!"를 출력한다.

## 다이나믹 메서드 디스패치 (Dynamic Method Dispatch)

> 같은 클래스를 상속하고 있는 여러 클래스 중 어느 자식 클래스를 사용할 것인지를 Runtime 시점까지 미뤄 클래스의 재사용성을 높이는 방법

설명에 앞서 정적 메서드 디스패치에 대해 먼저 알아보자.

```java
public class StaticDispatch {
   public static void print(String message) {
      System.out.println("Printing message: " + message);
   }

   public static void print(int number) {
      System.out.println("Printing number: " + number);
   }

   public static void main(String[] args) {
      print("Hello World!"); // 출력 결과: "Printing message: Hello World!"
      print(10); // 출력 결과: "Printing number: 10"
   }
}
```

StaticDispatch 클래스에 print 메서드가 두 개 정의되어 있다.

main 메서드에서는 print 메서드를 두 번 호출하는데, 첫 번째 호출에서는 문자열 인자를, 두 번째는 정수 인자를 전달한다.

컴파일러가 각 print 메서드를 호출할 때, 전달된 인자의 타입에 따라 정적으로 메서드를 선택한다. 

즉, 첫 번째 print 메서드를 호출할 때는 문자열 타입의 인자를 전달했기 때문에 문자열을 출력하는 메서드가 선택되고, 

두 번째 print 메서드를 호출할 때는 정수 타입의 인자를 전달했기 때문에 정수를 출력하는 메서드가 선택된다.

이러한 과정을 **정적 메서드 디스패치** 혹은 **정적 바인딩**이라고 한다.

```java
public class DynamicDispatch {
   public static void main(String[] args) {
      Animal animal = new Animal();
      animal.makeSound(); // 출력 결과: "Animal is making a sound"

      animal = new Dog();
      animal.makeSound(); // 출력 결과: "Woof!"
   }
}

class Animal {
   public void makeSound() {
      System.out.println("Animal is making a sound");
   }
}

class Dog extends Animal {
    
   @Override
   public void makeSound() {
      System.out.println("Woof!");
   }
}
```

Animal 클래스에는 makeSound 메서드가 정의되어 있고, 
Dog 클래스는 Animal 클래스를 상속받아서 makeSound 메서드를 오버라이딩하여 재정의했다.

main 메서드에서는 먼저 Animal 객체를 생성하고 makeSound 메서드를 호출하면

이때 Animal 객체의 makeSound 메서드가 실행되어 "Animal is making a sound"를 출력한다.

그리고 다시 Dog 객체를 생성하고 makeSound 메서드를 호출하고, 이때 Dog 객체의 makeSound 메서드가 실행되어 "Woof!"를 출력한다.

이 과정에서 컴파일러는 animal 변수가 참조하는 객체의 타입 정보를 이용하여 실행 시점에 어떤 makeSound 메서드를 호출해야 하는지 결정하는데,
이를 **다이나믹 메서드 디스패처** 혹은 **동적 바인딩**이라 한다.

## 추상 클래스(Abstract Class)

> 추상 클래스는 인스턴스화할 수 없지만, 하위 클래스로 만들 수 있다. [^2]

- 자식 클래스에서 반드시 오버라이딩해야만 사용할 수 있는 메서드를 추상 메서드라 한다.
  - 추상 메서드는 선언부만 존재하며, 구현부는 작성되어있지 않다.
  - 하나 이상의 추상 메서드를 포함하는 클래스를 가르켜 **추상 클래스**라 한다.

```java
public abstract class Animal {
  public void eat() {
    System.out.println("Animal is eating");
  }

  public abstract void makeSound();
}

class Dog extends Animal {
  @Override
  public void makeSound() {
    System.out.println("Woof!");
  }
}

class Cat extends Animal {
  @Override
  public void makeSound() {
    System.out.println("Meow!");
  }
}

public class AbstractClass {
  public static void main(String[] args) {
    Animal animal = new Dog();
    animal.makeSound(); // 출력 결과: "Woof!"

    animal = new Cat();
    animal.makeSound(); // 출력 결과: "Meow!"
  }
}
```

Animal 클래스에는 eat 메서드와 추상 메서드인 makeSound가 정의하고, makeSound 메서드는 어떤 동물의 소리를 출력할지 결정하기 위해서 추상 메서드로 정의했다.

main 메서드에서 Dog을 생성하고 makeSound 메서드를 호출했다. 이때 Dog 클래스에서 구현된 makeSound 메서드가 실행되어 "Woof!"를 출력하게 된다.

그리고 다시 Cat 객체를 생성하고 makeSound 메서드를 호출하는데, 이때 Cat 클래스에서 구현된 makeSound 메서드가 실행되어 "Meow!"를 출력한다.

이 과정에서 Animal 추상 클래스를 상속받은 Dog와 Cat 클래스가 makeSound 메서드를 각각 구현함으로써, 객체 생성 시점에 컴파일러는 makeSound 
메서드가 어떻게 구현되어 있는지에 따라 실행 시점에서 호출될 메서드가 결정된다.

```java
interface Vehicle {
  int getNumberOfWheels();
  String getColor();
}

class Car implements Vehicle {
  private int wheels;
  private String color;

  public Car(int wheels, String color) {
    this.wheels = wheels;
    this.color = color;
  }

  @Override
  public int getNumberOfWheels() {
    return this.wheels;
  }

  @Override
  public String getColor() {
    return this.color;
  }
}

public class InterfaceExample {
  public static void main(String[] args) {
    Vehicle myCar = new Car(4, "red");
    System.out.println("My car has " + myCar.getNumberOfWheels() + " wheels and is " + myCar.getColor());
  }
}
```

**인터페이스(Interface)** 는 추상 클래스보다 더 추상화된 개념으로, 추상 메서드와 상수만을 포함하며, 
다른 클래스에서 구현될 수 있는 행동의 규격(specification)을 정의한다. 추상 클래스와 차이점을 비교해보면

- 추상 클래스는 객체를 직접적으로 생성할 수 없지만, 인터페이스는 객체를 생성할 수 없다.
- 추상 클래스는 상속을 통해 확장될 수 있지만, 인터페이스는 다른 클래스에서 구현될 수 있다. 
- 인터페이스는 다중 상속을 허용하며, 다양한 객체들이 공통적으로 가져야 할 행동을 정의할 수 있다.

## final 키워드

Java의 final 키워드는 다음과 같다.

- final 변수: 값을 변경할 수 없는 상수를 정의할 때 사용한다
- final 메서드: 오버라이딩을 막고 해당 메서드가 상속 계층 구조에서 변경되지 않도록 한다
- final 클래스: 상속을 막고 해당 클래스가 확장되지 않도록 한다
- final 매개변수: 메서드 내에서 매개변수의 값을 변경할 수 없도록 한다

## Object 클래스

> java.lang.Object 클래스는 모든 클래스의 최상위 클래스이다.

모든 클래스는 정의할 때부터 명시적으로 Object 클래스를 상속받으므로, 설명하게 될 아래 메서드는 어떤 클래스에서도 호출이 가능하다.

- clone() :  객체를 복제하여 똑같은 객체를 생성한다.
  - 이때 객체를 복제하기 위해서는 해당 클래스가 Cloneable 인터페이스를 구현하고 있어야 한다.
  - 이 메소드를 사용하여 객체를 복제할 경우, 복제된 객체와 원본 객체는 서로 다른 객체이지만, 내부의 값이나 상태는 동일하다.

- equals() :  객체의 동등성 여부를 비교합니다. 
  - 기본적으로 Object 클래스에서는 참조 값 비교를 수행하므로, 두 객체가 같은 객체인지를 비교한다.
  - 이 메서드를 오버라이딩하여 객체의 내용이 같은지 여부를 판단할 수 있다.

- hashCode() : 객체의 해시코드 값을 반환한다.
  - 객체의 해시코드는 equals() 메서드와 함께 사용된다.
  - hashCode() 메서드를 오버라이딩하여 객체가 같은 경우 같은 해시코드 값을 반환하도록 구현할 수 있다.

- toString() : 객체의 문자열 표현을 반환한다.
  - System.out.println() 메서드나 문자열 연결 연산자(+) 등에서 객체를 문자열로 출력할 때 자동으로 호출된다.
  - 이 때 반환되는 문자열은 객체의 클래스명과 해시코드 값으로 구성된다.
  - 이 메서드를 오버라이딩하여 객체의 원하는 문자열 표현을 반환할 수 있다.

- finalize() : 객체가 가비지 컬렉터에 의해 수거되기 전에 호출되는 메소드이다.
  - 이 메소드를 오버라이드하여 객체가 메모리에서 제거되기 전에 어떤 작업을 수행할 수 있다.
  - finalize() 메소드는 호출되는 시점이 보장되지 않기 때문에, (거의) 사용하지 않는다.

- getClass() : 객체의 클래스 정보를 반환한다.
  - 이 메서드를 호출하면 Class 객체가 반환되며, 이 Class 객체를 통해 해당 클래스의 정보를 가져올 수 있다.
  - 이 메서드를 사용하여 객체의 클래스 정보를 가져와서, 객체의 클래스 이름, 상위 클래스, 인터페이스 등의 정보를 알 수 있다.

## 참고자료
- https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html
- https://docs.oracle.com/javase/tutorial/java/IandI/super.html
- https://docs.oracle.com/javase/tutorial/java/IandI/override.html
- https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html

## 각주
[^1] : An instance method in a subclass with the same signature (name, plus the number and the type of its parameters) and return type as an instance method in the superclass overrides the superclass's method.
[^2] : Abstract classes cannot be instantiated, but they can be subclassed.




