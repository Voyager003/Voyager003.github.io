---
layout  : wiki
title   : 멤버 클래스는 되도록 static으로 만들라 
summary : 
date    : 2023-09-18 09:41:52 +0900
updated : 2023-09-18 14:28:58 +0900
tag     : java effectivejava
resource: 51/ABF9EE-FB9B-4A3C-80FE-A8D59328D6C0
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 중첩 클래스(nested class)

>> Java 언어는 다른 클래스 내에 클래스를 정의할 수 있다. ... 이를 중첩 클래스라고 한다. [^1]

```java
class OuterClass {
    ...
    class NestedClass {
        ...
    }
}
```

중첩 클래스의 종류는 정적 멤버 클래스, (비정적) 멤버 클래스, 익명 클래스 지역클래스 이렇게 네 가지로, 정적 멤버 클래스를 제외한 나머지 중첩 클래스는 내부(inner) 클래스이다. item24는 각 중첩 클래스를 언제, 왜 사용해야하는지 다룬다.

### 정적 멤버 클래스(static member class)

```java
public class OuterClass {

    // 정적 멤버 클래스
    public static class StaticInnerClass {
        ...
    }
}
```

 - 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점을 제외하고 일반 클래스와 같다. 

- 다른 정적 멤버와 같은 접근 규칙을 적용 받는다.

- 바깥(Outer) 클래스와 함께 쓰일 때 pulbic 도우미(helper) 클래스로 사용된다.

## 비정적 멤버 클래스(non-static member class)

```java
public class OuterClass {

    // 비정적 멤버 클래스
    public class InnerClass {
        ...
    }
}
```

- 정적 멤버 클래스와 비교했을 때, 구문상 차이는 static 유무이다.

- 의미상으로는 비정적 멤버 클래스의 인스턴스와 바깥 클래스의 인스턴스와 암묵적으로 연결된다.

- 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나, 바깥 인스턴스의 참조를 가져올 수 있다. 
  
```java
public class OuterClass {
    private int outerField;

    public OuterClass(int outerField) {
        this.outerField = outerField;
    }

    public class InnerClass {
        private int innerField;

        public InnerClass(int innerField) {
            this.innerField = innerField;
        }

        public void display() {
            System.out.println("Outer Field: " + outerField);
            System.out.println("Inner Field: " + innerField);
        }
    }
}

// main
public static void main(String[] args) {
    OuterClass outerInstance = new OuterClass(10);
        
    // 비정적 멤버 클래스의 인스턴스 생성
    OuterClass.InnerClass innerInstance = outerInstance.new InnerClass(20);

    innerInstance.display(); // 바깥 클래스와 내부 클래스의 멤버에 접근
}
```

InnerClass의 생성자에서 outerInstance.new InnerClass(20)로 내부 클래스의 인스턴스를 생성하는 부분에서 바깥 클래스의 outerInstance와 내부 클래스 InnerClass의 인스턴스 innerInstance가 연결된다. 

따라서 내부 클래스의 display 메서드에서 outerField에 암묵적으로 접근할 수 있게 된다.

비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문에 중첩 클래스의 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다.

또한 비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다. 

이 관계는 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지는게 보통이지만, 드물게는 다음과 같이 수동으로 만들기도 한다.

```java
바깥 인스턴스의 클래스.new MeberClass(args)
```

하지만 이러한 관계는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하고 생성시간이 더 걸린다는 점을 유의해야 한다.

### 언제 쓰이나?

비정적 멤버 클래스는 어댑터를 정의할 때 자주 쓰인다. 여기서 어댑터는 무엇일까?

>> 직접 연결할 수 없는 두 개의 호환되지 않는 인터페이스 사이의 커넥터 역할을 한다.[^2]

item24는 **어댑터**는 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게하는 뷰로 사용한다고 설명한다.

```java
public class MySet<E> extends AbstractSet<E> {

    ...
    
    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ...
    }
}
```

Map 인터페이스의 구현체들은 보통 자신의 Collection 뷰를 구현할 때 비정적 멤버 클래스를 사용하며, Set과 List같은 다른 Collection 인터페이스 구현들도 자신의 반복자를 구현할 때 비정적 멤버 클래스를 주로 사용한다.

그 이유는, 많은 Map 구현체는 key-value 쌍으로 표현하는 Entry 객체를 가지는데 Map과 연관되어 있지만 Entry 메서드를 직접 사용하지는 않는다. 따라서 Entry를 비정적 멤버 클래스로 표현하는 것은 자원의 낭비고, private 정적 멤버 클래스가 알맞다.

따라서 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static 키워드를 붙여 정적 멤버 클래스로 만들자. static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 가지며, GC가 바깥 클래스의 인스턴스를 수거하지 못하는 Memory leak가 생길 수 있다.([item7](https://voyager003.github.io/wiki/java/effective_item7/))

## 익명 클래스(anonymous class)

```java
public class AnonymousExample {

    private double x;
    private double y;

    public double operate() {
        Operator operator = new Operator() {
            @Override
            public double plus() {
                System.out.printf("%f + %f = %f\n", x, y, x + y);
                return x + y;
            }
            @Override
            public double minus() {
                System.out.printf("%f - %f = %f\n", x, y, x - y);
                return x - y;
            }
        };
        return operator.plus();
    }
}

interface Operator {
    double plus();
    double minus();
}
```

- 이름 그대로 이름이 없다.

- 바깥 클래스의 멤버가 아니며, 메멉와 달리 쓰이는 시점에 선언과 동시에 인스턴스가 생성된다.

- 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.

- 익명 클래스를 사용하는 클라이언트는 상위 타입에서 상속한 멤버 외에 호출할 수 없다.

- 표현식 중간에 등장하여 짧지 않다면, 가독성이 떨어진다.

람다 표현식이 등장하기 전에는 즉석에서 작은 함수 인스턴스 혹은 처리 객체를 만드는데 익명 클래스를 사용했지만, 람다 표현식이 그 자리를 대체했다. 주 쓰임은 정적 팩터리 메서드를 구현할 때이다.

```java
public class StaticFactoryWithAnonymousClass {
    
    public static List<String> createStringList() {
    
        // 익명 클래스를 사용해 ArrayList를 생성하고 반환
        return new ArrayList<String>() {
            {
                add("Item 1");
                add("Item 2");
                add("Item 3");
            }
        };
    }
}

// main
public static void main(String[] args) {

        // 정적 팩토리 메서드를 호출하여 리스트를 생성
        List<String> stringList = createStringList();

        // 생성된 리스트 출력
        for (String item : stringList) {
            System.out.println(item);
        }
    }
```

## 지역 클래스(local calss)

```java
public class LocalExample {
    private int number;

    public LocalExample(int number) {
        this.number = number;
    }

    public void foo() {
    
        class LocalClass {
            private String name;

            public LocalClass(String name) {
                this.name = name;
            }

            public void print() {
                System.out.println(number + name);
            }
        }
        LocalClass localClass = new LocalClass("local");
        localClass.print();
    }
}
```

- 지역 변수를 선언할 수 있는 곳이라면 어디서든 선언할 수 있다.

- 지역 변수와 유효 범위가 같다.

- 다른 세 중첩 클래스와의 공통점도 하나씩 가진다.
    - 멤버 클래스 : 이름이 있고 반복해서 사용 가능하다.
    - 익명 클래스 : 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버를 가질 수 없다.

## 참고자료

- 이펙티브자바 3판
- https://www.baeldung.com/java-adapter-pattern - 어댑터 패턴

## 주석

[^1]: The Java programming language allows you to define a class within another class. Such a class is called a nested class

[^2]: An Adapter pattern acts as a connector between two incompatible interfaces that otherwise cannot be connected directly.
