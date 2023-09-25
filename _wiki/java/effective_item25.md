---
layout  : wiki
title   : 톱레벨 클래스는 한 파일에 하나만 담으라 
summary : 
date    : 2023-09-25 20:21:13 +0900
updated : 2023-09-25 20:48:50 +0900
tag     : java effectivejava
resource: A8/6FB3C9-E2CB-4088-96D8-BD36E7495CE7
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 톱레벨 클래스(Top-level class)

```java
// in Foo.java:
public class Foo { // top level class

    public static class NestedBar { // nested class
    }
}
```

JLS에 따르면 톱레벨 클래스는 중첩 클래스가 아닌 클래스라고 설명한다. [^1]

위 코드를 살펴보면 해당 파일이 있는 foo.java 파일과 동일한 이름을 가진 정의된 클래스가 있는데, 이를 톱레벨 클래스라한다.

톱레벨 클래스를 여러 개 선언하더라도 Java 컴파일러는 불평하지않지만, 이는 위험을 감수해야하는 행위다. 

## 문제점

```java
// Utensil.java
class Utensil {
      static final String NAME = "pan";
}

class Dessert {
        static final String NAME = "cake";
}

// Dessert.java
class Utensil {
      static final String NAME = "pot";
}

class Dessert {
        static final String NAME = "pie";
}

// Main
public class Main {
      public static void main(String[] args){
          System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

Utensil과 Dessert 클래스에 각각 이름이 같은 Utensil, Dessert 클래스가 한 파일에 정의되어 있으며, Main에서는 이 톱레벨 클래스 2개(Utensil, Dessert)를 참조하고 있다.

이 우연히 똑같은 두 클래스를 javac Main.java Dessert.java 명령으로 컴파일한다면 컴파일 오류가 발생하고 클래스를 중복 정의했다고 알려줄 것이다. 컴파일러는 Main.java를 컴파일하고, 그 안에서 Dessert 참조보다 먼저 나오는 Utensil 참조를 만나면 Utensil.java를 살펴서 Utensil, Dessert 모두를 찾아낼 것이다. 

한편 javac Main.java Utensil.java 명령으로 컴파일 한다면 Dessert.java를 작성하기 전처럼 pancake를 출력하지만, 전자의 Dessert.java 명령으로 컴파일하면 potpie를 출력한다.

이처럼 컴파일러에 어떤 소스 파일을 먼저 넘기느냐에 따라 동작이 달라지므로 큰 문제가 발생한다.

## 해결책

```java
public class Test {

    public static void main(String[] args){
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
        static final String NAME = "pan";
    }
    
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

해결책은 단순 톱레벨 클래스들을 서로 다른 소스파일로 분리하는 것이다. 

굳이 톱레벨 클래스를 한 파일에 담고 싶다면 위의 코드와 같이 정적 멤버 클래스를 사용하는 방법을 고민해볼 수 있다.

소스파일 하나에는 반드시 톱레벨 클래스,인터페이스를 하나만 담도록 하자.

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html

## 주석

- [^1]: A top level class is a class that is not a nested class. 
