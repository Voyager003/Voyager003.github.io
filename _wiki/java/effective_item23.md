---
layout  : wiki
title   : 태그 달린 클래스보다는 클래스 계층구조를 활용하라
summary : 
date    : 2023-09-13 19:14:14 +0900
updated : 2023-09-13 20:35:22 +0900
tag     : java effectivejava
resource: 92/39EBDF-3071-4170-ACB1-E74F40535C09
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 태그(tagged) 클래스

태그 클래스는 인스턴스의 특징을 나타내는 태그 필드를 포함하는 클래스이다. 코드로 살펴보자. 

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 
    final Shape shape;

    // shape가 RECTANGLE일때만 사용
    double length;
    double width;

    // shape이 CIRCLE 일때만 사용
    double radius;

    // 원용 생성자
    Figure(double radius){
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width){
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape){
          case RECTANGLE:
              return length * width;
          case CIRCLE:
              return Math.PI * (radius * radius);
          default:
              throw new AssertionError(shape);
        }
    }
}
```

예시의 Figure 클래스는 생성자의 종류에 따라 Rectangle이 될 수도 있고, Circle도 될 수 있다. 이처럼 한 클래스가 조건에 따라 여러 다른 특성의 클래스의 형태로 변하는 클래스를 태그 클래스라 한다.

하지만 이 코드는 몇 가지 단점이 있다. 하나씩 살펴보면

- ENUM(열거) 타입 선언, 태그 필드, switch문 등 쓸데없는 코드가 많다.
- 여러 구현이 한 클래스에 혼합되어 있어 가독성이 떨어진다.
- 다른 의미를 위한 코드도 있어 메모리를 많이 사용한다.
- final로 필드를 선언 시, 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화 해야한다.
- 다른 의미를 추가하려면 코드를 수정해야 한다.
- 인스턴스 타입만으로 현재 나타내는 의미를 알기 어렵다.

이런 포인트들은 오류를 내기 쉽고, 비효율적이다. 

Java와 같은 객체 지향 언어는 타입 하나로 다양한 의미의 객체를 표현하는 더 좋은 수단을 제공한다. 

## 클래스 계층구조를 활용한 서브타이핑(subtyping)

item23은 비효율적인 태그 클래스의 대안으로 클래스 계층구조를 제시하는데, 위에서 예시로 든 Figure 클래스를 클래스 계층구조로 리팩토링 해보자.

먼저 계층 구조의 루트(root)가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다. 

```java
abstract class Figure {

    // 태그 값에 따라 동작이 달라지는 메서드
    abstract double area();
}

class Rectangle extends Figure {
    ...
}


class Circle extends Figure {
    ...
}


class Square extends Rectangle {
    ...
}
```

이어서 태그 값에 상관없이 동작이 일정한 메서드들을 루트 클래스에 일반 메서드로 추가하고, 모든 하위 클래스에서 공통으로 사용하는 데이터 필드를 전부 루트 클래스로 올린다.

```java
abstract class Figure {
    abstract double area();
}

class Rectangle extends Figure {
    ...
}

class Circle extends Figure {
    ...
}
```
    
이제 Figure 클래스에는 태그 값에 상관없는 메서드가 하나도 없고, 모든 하위 클래스에서 사용하는 공통 데이터 필드도 없다. 결과적으로 루트 클래스에는 추상 메서드 area() 하나만 남게 된다.

다음으로 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의하고, 각 하위 클래스에는 각자의 의미에 해당하는 데이터 필드를 넣는다.

```java
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override
    double area() {
        return length * width;
    }
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override
    double area() { 
        return Math.PI * (radius * radius); 
    }
}

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

루트 클래스가 정의한 추상클래스를 의미에 맞게 구현한 뒤에, 계층 구조 방식으로 구현한 코드이다. 

태그 클래스의 단점을 모두 없애고, 각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 삭제했다. 또한 남아 있는 필드들은 모두 final로 각 클래스의 생성자가 모든 필드를 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.

결과적으로 타입 사이의 자연스러운 계층 관계를 반영하여 유연성은 물론 컴파일타임의 타입 검사능력을 높여준다는 메리트까지 가져왔다. 

기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩토링하는 것을 고려해보자!

## 참고자료

- 이펙티브 자바 3판
- https://appmattus.medium.com/effective-kotlin-item-23-prefer-class-hierarchies-to-tagged-classes-de99d37f815a - tagged class
