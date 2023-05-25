---
layout  : wiki
title   : Java Generic 
summary : 
date    : 2023-05-25 09:54:29 +0900
updated : 2023-05-25 11:42:00 +0900
tag     : java
resource: F6/DD7B25-C83E-489E-BF9F-3AADCDDB327B
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/14) 14주차 과제 

## Generic

> 데이터 타입을 일반화하는 것

- Java 5에 등장했으며, 클래스나 메서드에서 사용할 내부 데이터 타입을 컴파일 시 미리 지정하는 방법이다.
- 제네릭 등장 이전에는 Object 타입을 원하는 타입으로 캐스팅하여 사용했지만, 오류 발생 가능성이 생겼다.
- 컴파일시 타입체크는 클래스나 메서드 내부에서 사용되는 객체의 타입 안정성을 높일 수 있으며
- 반환값에 대한 타입 변환 및 타입 검사에 들어가는 시간을 줄인다.

### 사용법

```java
class GenericsEx<T> {
    T element;
    void setElement(T element) {
        this.element = element;
    }
    T getElement() {
        return element;
    }
}
```

- 아무 이름을 지정하더라도 컴파일하는데 지장이 없지만, 네이밍 관례를 따르는 것이 좋다.
    - E : 요소(Element로 주로 자바 컬렉션에서 사용)
    - K : 키(key)
    - N : 숫자(number)
    - T : 타입(type)
    - V : 값(value)
- 여러 개의 타입 변수는 쉼표(,)로 구분하여 명시가능하다.
- 클래스 뿐만아니라 메서드의 패러미터, 리턴값으로 사용할 수 있다.

예시를 살펴보자.

```java
public class Box<T> {
    private T content;

    public void add(T item) {
        this.content = item;
    }

    public T get() {
        return this.content;
    }

    public static void main(String[] args) {
        // 정수형 Box 생성
        Box<Integer> integerBox = new Box<>();
        integerBox.add(10);
        int intValue = integerBox.get();
        System.out.println("정수형 Box의 내용: " + intValue);

        // 문자열 Box 생성
        Box<String> stringBox = new Box<>();
        stringBox.add("안녕하세요!");
        String stringValue = stringBox.get();
        System.out.println("문자열 Box의 내용: " + stringValue);
    }
}

// 실행 결과
정수형 Box의 내용: 10
문자열 Box의 내용: 안녕하세요!
```

- Box 클래스는 제네릭 타입 매개변수 T를 사용한다. 
- add 메서드는 T 타입의 항목을 Box에 추가하고, get 메서드는 T 타입의 항목을 반환한다.
- main 메서드에서는 Box 클래스를 정수형과 문자열 타입에 대해 인스턴스화하고, 각각의 값을 추가하고 가져와서 출력한다. 
- 제네릭을 사용하여 다양한 타입의 데이터를 처리할 수 있는 유연성과 재사용성을 확인할 수 있다.

## Generic 주요 개념

### Bounded type parameter(바운디드 타입 패러미터)

- 바운디드 타입 패러미터는 특정 타입의 서브타입으로 제한하는 것이다.
    - 예를 들어 숫자에 대해 작동하는 메서드는 Nuber 혹은 해당 하위 클래스의 인스턴스만 허용하려고 하는 경우가 있다.

```java
<T extends UpperBound>

<T extends B1 & B2>

<T extends Class1 & Interface1 & Interface2>
```

- 바운디드 타입 패러미터를 선언하려면 타입 패러패러미터의 이름, extends 키워드, 상위 바운드를 나열한다.
- 여러 개의 상위 바운드를 가질 수 있다.
- 만약 여러 상위 바운드 중 클래스가 있다면 해당 클래스가 가장 먼저 앞에와야 한다. (컴파일 에러)

```java
public class BoundTypeEx <T extends Number> {
    
    public void set(T value) {}

    public static void main(String[] args) {
        BoundTypeEx<Integer> boundTypeSample = new BoundTypeSample<>();
        boundTypeEx.set("Hi"); // 컴파일 에러 발생
    }
}
```

- BoundTypeEx 클래스의 타입 패러미터를 <T extends Number>로 선언한다.
    - 이는 BoundTypeEx의 타입으로 Number의 서브 타입만 허용한다는 것을 의미한다.
- Integer는 Number의 서브타입으로 유효한 선언이지만, set 함수의 패러미터로 String을 전달하려고 했기 때문에 컴파일 에러가 발생한다.

### Wildcard(와일드카드)

![](https://docs.oracle.com/javase/tutorial/figures/java/generics-wildcardSubtyping.gif) 

```java
<? Extends UpperBound>
```

- 제네릭으로 구현된 메서드의 경우 선언된 타입으로만 매개변수를 입력해야한다.
- 이를 상속받은 클래스 또는 부모 클래스를 사용하고 싶어도 불가능하며, 어떤 타입이 와도 상관없는 경우에는 대응하기 좋지 않다.
- 이를 위한 해결책인 **와일드 카드('?')**는 알 수 없는 유형을 나타내며, 패러미터 변수, 필드, 지역변수의 타입 등 다양한 상황에서 사용할 수 있다.

- Unbounded Wildcard
    - List<?>와 같은 형태로 물음표만 가지고 정의된다.
    - 내부적으로 Object로 정의되어서 사용되고 모든 타입의 인자를 받을 수 있다.
    - 타입 패러미터에 의존하지 않는 메서드만을 사용하거나 Object 메서드가 제공하는 기능으로 충분한 경우에 사용한다.
    - 예시로 코드 타입 매개변수에 의존하지 않는 제네릭 클래스의 메서드를 사용하는 경우인 List.clear, List.size가 있다.

```java
import java.util.ArrayList;
import java.util.List;

public class UnboundedWildcardExample {
    public static double sumList(List<? extends Number> list) {
        double sum = 0;
        for (Number number : list) {
            sum += number.doubleValue();
        }
        return sum;
    }

    public static void main(String[] args) {
        List<Integer> integerList = new ArrayList<>();
        integerList.add(1);
        integerList.add(2);
        integerList.add(3);

        List<String> stringList = new ArrayList<>();
        stringList.add("Hello");
        stringList.add("World");

        double sum1 = sumList(integerList); // 정수형 리스트의 합 계산
        double sum2 = sumList(stringList); // 컴파일 에러!

        System.out.println("정수형 리스트 합: " + sum1);
        System.out.println("실수형 리스트 합: " + sum2);
    }
}
```

- sumList 메서드는 제한되지 않은 상위 바운드 와일드카드(List<? extends Number>)를 사용하여 Number 클래스의 하위 타입 리스트를 인자로 받는다.
- 그러나 sumList 메서드에 문자열 리스트를 전달하면 컴파일 에러가 발생한다.
- 이는 문자열이 Number 클래스의 하위 타입이 아니기 때문이다.


- Upper Bounded Wildcard
    - List<? extends Foo>와 같은 형태로 사용하며 특정 클래스의 자식 클래스만을 인자로 받는다.
    - 임의의 Foo 클래스를 상속받은 어느 클래스가 와도되지만 사용할 수 있는 기능은 Foo에 정의된 기능만 사용가능하다.

- Lower Bounded Wildcard
    - List<? super Foo> 와 같은 형태로 사용하며 특정 클래스의 부모 클래스만을 인자로 받는다.

```java
import java.util.ArrayList;
import java.util.List;

public class BoundedWildcardExample {
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i);
        }
    }

    public static void main(String[] args) {
        List<Object> objectList = new ArrayList<>();
        objectList.add("Hello");

        List<String> stringList = new ArrayList<>();
        stringList.add("World");

        addNumbers(objectList); // Object 타입 리스트에 정수형 추가
        addNumbers(stringList); // 컴파일 에러!

        System.out.println("Object 타입 리스트: " + objectList);
        System.out.println("문자열 리스트: " + stringList);
    }
}
```

- addNumbers 메서드는 하위 바운드 와일드카드(List<? super Integer>)를 사용하여 Integer 클래스의 상위 타입 리스트를 인자로 받는다. 
- 하지만 addNumbers 메서드에 문자열 리스트를 전달하면 컴파일 에러가 발생
- 이는 문자열이 Integer 클래스의 상위 타입이 아니기 때문이다.

### Parameterize type(매개변수화 타입)

- 하나 이상의 타입 매개변수를 선언하고 있는 클래스, 인터페이스를 제니릭 클래스, 인터페이스라고 하며 이를 제네릭 타입이라 한다.
- 각 제네릭 타입에서는 매개변수화 타입들을 정의한다.

```java
List<String> list = new ArrayList<>();

// 컴파일 시
ArrayList list = new ArrayList();
```

- <> 안의 String은 실 타입 매개변수라하며 List 인터페이스에 선언되어있는 List의 E를 형식 타입 매개변수라 한다.
- 제네릭은 타입 소거자(Type erasure)에 의해 자신의 타입 요소 정보를 삭제한다.


## Generic 메서드 만들기

```java
// Collections.sort
public static <T> void sort(List<T> list, Comparator<? super T> c) {
    list.sort(c);
}
```

- 제네릭 메서드는 메서드의 선언부에 타입 변수를 사용한 메서드를 의미한다.
- 타입 변수의 선언은 메서드 선언부에서 반환 타입 바로 앞에 위치한다.
- 제네릭 클래스에서 정의된 타입 변수 T와 제네릭 메서드에서 사용된 타입 변수 T는 별개이다.

## Erasure

### Type Erasure

- 컴파일러는 컴파일 타임에 타입 패러미터를 사용하는 대상의 타입을 컴파일러가 정하는 타입으로 대체하는 Type Erasure를 실행한다.
- 컴파일된 바이트 코드에서는 T 대신 특정 타입으로 대체된다. 
- 제네릭 타입의 타입 패러미터가 상하한이 있는 경우에 타입 패러미터를 한계 타입으로, 없는 경우 모든 타입 패러미터를 Object로 바꾼다.
    - 이렇게 생성된 바이트 코드에는 보통 클래스, 인터페이스 및 메서드만 포함된다.
- type-safety를 유지하기 위해 필요한 경우 타입 캐스팅을 사용할 수 있다.
- 제네릭 타입을 상속받은 클래스에서는 다형성을 유지하기 위해 브릿지 메서드를 생성한다.

### 제네릭 타입 Erasure

- Java 컴파일러는 타입 Erasure 프로세스로써 모든 타입 패러미터를 지우고 타입 패러미터가 바인드된 경우 첫 번째 바인드로 대체하고, 바인드되지 않은 경우 Object로 대체한다.

```java
public class WitchPot<T> {
    private T meterial;

    public WitchPot(T meterial) {
        this.meterial = meterial;
    }

    public T get() {
        return this.meterial;
    }
    public void set(T meterial) {
        this.meterial = meterial;
    }

}

// 바이트 코드
javap -c WitchPot 
Warning: File ./WitchPot.class does not contain class WitchPot
Compiled from "WitchPot.java"
public class javageneric.WitchPot<T> {
  public javageneric.WitchPot(T);
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: aload_1
       6: putfield      #2                  // Field meterial:Ljava/lang/Object;
       9: return

  public T get();
    Code:
       0: aload_0
       1: getfield      #2                  // Field meterial:Ljava/lang/Object;
       4: areturn

  public void set(T);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #2                  // Field meterial:Ljava/lang/Object;
       5: return
}
```

- 바이트코드 확인 시, T가 사용되는 부분에 lang/Object 즉 Object로 대체됨을 확인할 수 있다.

### 브릿지 메서드

- 제네릭 클래스를 상속받거나, 제네릭 인터페이스를 구현하는 클래스, 인터페이스를 컴파일할 때 컴파일러는 타입 Erasure 프로세스의 일부로 브릿지 메서드라는 합성 메서드를 만들어야할 수도 있다.
- 이 브릿지 메서드는 매개변수나 반환타입을 소거하기 위해 만들어지는 메서드이다.

```java
public class Shape<T> {
    public void draw(T shape) {
        System.out.println("Drawing shape: " + shape);
    }
}

public class Circle extends Shape<String> {
    @Override
    public void draw(String shape) {
        System.out.println("Drawing circle: " + shape);
    }
}

public class Main {
    public static void main(String[] args) {
        Circle circle = new Circle();
        circle.draw("Red circle");
    }
}
```

-  Shape 클래스는 제네릭 타입 T를 받는 메서드 draw를 가지고 있고, Circle 클래스는 Shape<String>을 상속받으며, draw 메서드를 오버라이딩하여 문자열을 받도록 정의한다.
- 이때, Shape 클래스에서 draw 메서드의 시그니처는 draw(T shape)이다. 그러나 제네릭 타입 T는 타입 소거에 의해 Object 타입으로 지워지므로, 실제로는 draw(Object shape)로 동작하게 된다.
- 따라서 Circle 클래스에서 오버라이딩한 draw 메서드는 draw(String shape)이다.
- 여기서 컴파일러에 의해 브릿지 메서드가 자동으로 생성되며 브릿지 메서드는 Circle 클래스에 추가되어서 draw(Object shape)의 형태로 생성된다.
- 이렇게 된다면 Shape 클래스의 draw 메서드와 Circle 클래스의 오버라이딩한 draw 메서드 간에 호환성이 유지되며, 타입 안전성을 보장하면서 상위 클래스와 하위 클래스 간의 형 변환 문제를 해결할 수 있다.

## 참고자료

- https://docs.oracle.com/javase/tutorial/java/generics/index.html - docs
- https://rockintuna.tistory.com/102#type-erasure - erasuer 파트
