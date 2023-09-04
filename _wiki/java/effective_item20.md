---
layout  : wiki
title   : 추상 클래스보다는 인터페이스를 우선하라 
summary : 
date    : 2023-09-04 11:06:31 +0900
updated : 2023-09-04 15:27:23 +0900
tag     : java effectivejava
resource: C1/1B5DE0-562B-4491-9A11-03B0E6923B3C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 개요

Java가 제공하는 다중 구현 메커니즘은 추상 클래스와 인터페이스가 있다.

두 개 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.(Java 8부터 인터페이스가 default method를 제공)

둘의 가장 큰 차이는 추상 클래스의 경우 단일 상속만을 지원하고 구현 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. 반면, 인터페이스는 다중 상속이 가능하며, 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했던 같은 타입으로 취급된다. 

이러한 특징때문에 추상 클래스 방식은 새로운 타입을 정의하는 데 커다란 제약을 안게된다. 대신에 인터페이스를 사용해 얻을 수 있는 메리트는 무엇이 있을까?

## 인터페이스(Interface)의 장점

- 기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

```java
// 기존 클래스
class MyExistingClass {
    void doSomething() {
        System.out.println("기존 클래스에서 작업 수행");
    }
}

// 인터페이스
interface MyInterface1 {
    void method1();
}

// 인터페이스 구현
class ExtendedClass extends MyExistingClass implements MyInterface1 {

    @Override
    public void method1() {
        System.out.println("MyInterface1 메서드 실행");
    }
}
```

위와 같이 요구하는 메서드를 추가하고, 클래스 선언에 implements 구문만 추가하면 끝이다. 

만약 추상 클래스를 사용한다면, 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려울 것이다. 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 추상 클래스가 두 클래스의 공통 조상이어야하기 때문이다.

- 믹스인(mixin) 정의에 알맞다.

**믹스인**이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 주된 타입 외에 선택정 기능을 제공한다고 선언하는 효과를 줄 수 있다. 코드로 살펴보자.

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

Comparable은 자신을 구현한 클래스의 인스턴스들끼리 순서를 정할 수 있다고 선언하는 믹스인 인터페이스이다.

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Person other) {
        // 나이를 기준으로 정렬
        return Integer.compare(this.age, other.age);
    }

    @Override
    public String toString() {
        return name + " (" + age + " years old)";
    }
}
```

Person 클래스는 Comparable을 구현하고 있으며, compareTo()를 오버라이드하여 나이를 기준으로 정렬하도록 구현되어 있다. 

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        Person[] people = {
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35)
        };

        Arrays.sort(people); // 정렬

        for (Person person : people) {
            System.out.println(person);
        }
    }
}
```

Arrays.sort()를 사용해 Person 배열을 나이를 기준으로 정렬하는 코드이다.

Comparable 인터페이스를 구현했기 때문에, 다른 클래스에도 이 클래스를 정렬 가능하게 만들 수 있다. 추상 클래스였다면 기존 클래스에 덧씌울 수 없고, 두 부모를 섬길 수 없는 단일 상속의 특징이 있기때문에 믹스인을 정의할 수 없다.

- 계층 구조가 없는 타입 프레임워크를 만들 수 있다.

타입을 계층적으로 정의하면 많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다. 코드로 살펴보자.

```java
// 추상 클래스의 경우
public abstract class Singer {
    abstract void sing(String s);
}

public abstract class SongWriter {
    abstract void compose(int chartPosition);
}

public abstract class SingerSongWriter {
    abstract void strum();
    abstract void actSensitive();
    abstract void Compose(int chartPosition);
    abstract void sing(String s);
}

// 인터페이스의 경우
public interface SingerSongWriter extends Singer, SongWriter {
    void strum();
    void actSensitive();
}
```

추상 클래스를 사용한 경우를 살펴보자. 첫 번째로 다중 상속이 불가능하여 새로운 추상 클래스를 만들어 클래스 계층을 표현할 수 밖에 없다.

이런 방식은 계층 구조를 만들기 위해 가능한 조합 전부를 각 클래스로 정의한 고도비만 계층 구조가 만들어 진다. 흔히 조합 폭발(combinatorial explosion)이라 부르는 현상이다.

인터페이스의 경우 Singer, Songwriter 모두를 구현해도 전혀 문제되지 않으며, 확장하여 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있다.

- 래퍼(Wrapper) 클래스 관용구와 함께 사용 시, 기능을 향상시키는 안전하고 강력한 수단이 된다.

타입을 추상 클래스로 정의했다면 그 타입에 기능을 추가하는 방법은 상속뿐이다. 상속해 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기 쉽다. 

## 인터페이스 디폴트 메서드(default method) 제약

인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 구현을 디폴트 메서드로 제공해 프로그래머의 일감을 덜 수 있다.

1. 디폴트 메서드 제공 시, 상속하려는 사람을 위한 @implSpec javadoc 태그를 붙여 문서화 해야한다. ([item19](https://voyager003.github.io/wiki/java/effective_item19/#%EC%83%81%EC%86%8D%EC%9D%98-%EC%9C%84%ED%97%98%EC%84%B1))

2. equlas & hashCode는 디폴트 메서드로 정의하면 안된다.

3. 인터페이스 필드를 가질 수 없다.

4. public이 아닌 정적 멤버를 가질 수 없다.

5. 우리가 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

## 인터페이스와 추상 골격 구현(skeletal implementation)

인터페이스는 추상 골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취할 수 있다.

인터페이스로 타입을 정의하고, 필요하다면 디폴트 메서드를 함께 제공한다. 그리고 골격 구현 클래스가 나머지 메서드를 구현한다. 이 방법은 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다. 

이것이 바로 [템플릿 메서드 패턴(Template Method Pattern)](https://voyager003.github.io/wiki/Pattern/template_method_pattern/)이다.

템플릿 메서드 패턴은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 제약에서 자유롭다.

### 골격 구현 작성 방법

1. 인터페이스를 살펴 다른 메서드들의 구현에 사용되는 기반 메서드를 선정

2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드들을 모두 디폴트 메서드로 제공 
   
3. 기반 메서드 혹은 디폴트 메서드로 만들지 못한 메서드가 남았다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어서 작성한다.

4. 이 인터페이스를 구현하는 골격 구현 클래스는 필요하면 public이 아닌 필드와 메서드를 추가해도 된다.

5. 기본적으로 상속해 사용하는 것을 가정하므로 설계 및 문서화 지침을 모두 따른다.

## 시뮬레이트한 다중상속(Simulated multiple inheritance)

구조상 골격 구현을 확장하지 못하는 상황이라면, 인터페이스를 직접 구현해야 한다.

이런 경우에도 디폴트 메서드의 이점을 여전히 누릴 수 있으며, 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달하는 것이다.

이 방식을 시뮬레이트한 다중 상속이라 한다.

## 단순 구현(Simple implementation)

단순 구현은 골격 구현의 작은 변종으로 AbstractMap.SimpleEntry가 그 예이다.

```java
public static class SimpleEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        @java.io.Serial
        private static final long serialVersionUID = -8499721149061103585L;

        @SuppressWarnings("serial") // Conditionally serializable
        private final K key;
        @SuppressWarnings("serial") // Conditionally serializable
        private V value;

        /**
         * Creates an entry representing a mapping from the specified
         * key to the specified value.
         *
         * @param key the key represented by this entry
         * @param value the value represented by this entry
         */
        public SimpleEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        /**
         * Creates an entry representing the same mapping as the
         * specified entry.
         *
         * @param entry the entry to copy
         */
        public SimpleEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }
    }
    
    ...
```

골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니라는 차이점이 있다. 동작하는 가장 단순한 구현으로, 그대로 사용해도 되고 필요에 맞게 확장해도 된다.

## 참고자료

- 이펙티브자바 3판

















