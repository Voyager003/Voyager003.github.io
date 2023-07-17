---
layout  : wiki
title   : clone 재정의는 주의해서 진행하라 
summary : 
date    : 2023-07-17 09:48:11 +0900
updated : 2023-07-17 14:38:17 +0900
tag     : java effectivejava
resource: CF/ECE78C-5101-4A5E-97D0-6FF46DD7C5BC
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Cloneable과 mixin interface

```
// Cloneable interface 
public interface Cloneable {
}

// object.clone()
@IntrinsicCandidate
protected native Object clone() throws CloneNotSupportedException;
```

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 **mixin interface** 지만, 의도한 목적을 제대로 이루지 못했다.

여기서 mixin interface는 클래스가 구현할 수 있는 타입으로, mixin을 구현한 클래스에 원래의 주된 타입 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 줄 수 있다. Comparable 인터페이스로 예를 들어보자.

```java
// Comparable
public interface Comparable<T> {
    public int compareTo(T o);
}

// Point
public class Point implements Comparable<Point> {
    int x;
    int y;
    
    public void print() {
        System.out.println(String.format("x: %d\ny: %d", x, y));
    }    
}
```

이 코드에서 Point 클래스가 구현할 수 있는 타입은 무엇일까?

여러가지가 있을 수 있겠지만, 그 중 하나는 Comparable로 이유는 Point 클래스를 COmparable 인터페이스를 구현한 구현체로 만들 수 있기 때문이다.

```java
public class Point implements Comparable<Point> {
    int x;
    int y;
    
    public void print() {
        System.out.println(String.format("x: %d\ny: %d", x, y));
    }
    
    // 인터페이스 구현
    @Override
    public int compareTo(final Point o) {
        return 0;
    }
}
```

여기서 클래스가 구현할 수 있는 타입을 mixin이라고 부르는 이유는 

대상 타입(위의 경우 Point)의 주 기능에 선택적 기능(Comparable)을 혼합(mixed-in) 한다고 하기 때문이다.

자 이제 Cloneable 다시 살펴보자.

```java
public interface Cloneable {
}
```

Cloneable 인터페이스는 mixin interface라 했지만, Comparable과 달리 인터페이스만 있을 뿐, 선택적 기능을 제공하는 메서드를 찾을 수 없다.

```java
public class Object { 
    
    ...

    @IntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
    ...
```

clone()가 선언된 곳이 Cloneable이 아닌 Object이며 그마저도 protected로 선언되어 reflection을 사용하지 않으면 재정의한 메서드에 접근할 수 있다는게 보장되지 않으며, reflection을 사용한다 하더라도 메서드를 재정의하지 않으면 예외를 던지기 때문에 확실한 메서드 접근을 보장하지 못한다.

이 Cloneable은 대체 무슨 역할을 하는 걸까?

## Cloneable interface

결론부터 말하면 Cloneable은 clone()의 동작 방식을 결정한다.

Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 인스턴스의 필드들을 복사한 인스턴스를 반환하며, 구현하지 않았다면 CloneNotSupportedException 예외를 던진다.

interface를 구현한다는 것은 일반적으로 해당 클래스가 그 interface에서 정의한 기능을 제공한다고 선언하는 행위지만, Cloneable은 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.

실무에서 Cloneable을 구현한 클래스는 clone()을 public으로 제공하며, 사용자는 복제가 제대로 이뤄질거라 기대한다. 이 기대를 만족시키려면 그 클래스와 모든 상위 클래스는 복잡하고 허술한 프로토콜을 지켜야 하는데 이는 모순적 메커니즘을 탄생시킨다. 

실제 Object의 clone 명세는 다음과 같다.

- x.clone() != x;
- x.clone().getClass() == x.getClass();
- x.clone().equals(x);
- 이 메서드가 반환하는 인스턴스는 super.clone을 호출해 얻어야 한다.
- 이 클래스와 Object를 제외한 모든 상위 클래스가 이 관례를 따른다면 다음은 참이다.
- x.clone().getClass() == x.getClass();
- 이는 강제성이 없으며 선택사항이다.

이를 따라서 clone()이 super.clone()이 아닌 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일러는 불평하지 않지만, 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 인스턴스가 만들어져 하위 클래스의 clone()이 제대로 동작하지 않게 된다.

실제로 그런지 코드로 확인해보자. 

### Cloneable 구현

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public Person clone() throws CloneNotSupportedException {
        return (Person) super.clone();
    }
}

// test
public void CloneableTest1() throws Exception {

    Person p = new Person("kim", 20);
    Person p2 = p.clone();
    boolean result = p == p2;
    Assertions.assertThat(result).isFalse();
}

// 결과
java.lang.CloneNotSupportedException: effectivejava.practicecode.Person
```

clone()까지 재정의하고 컴파일러도 문제를 제기하지 않았지만, 실행결과로 checkedException인 CloneNotSupportedException이 발생했다.

```
Thrown to indicate that the clone method in class Object has been called to clone an object, but that the object's class does not implement the Cloneable interface.
```

API를 살펴보면 clone()이 호출됐지만, Cloneable을 구현하지 않으면 발생하는 예외라고 적혀있다. 이를 

```java
class Person implements Cloneable {
    ...
    
    ...

@Override
public Person clone() {
    try {
        return (Person) super.clone();
    } catch (final CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없는 일
    }
}

// test
Person p = new Person("kim", 20);
Person p2 = p.clone();

boolean result = p == p2;
Assertions.assertThat(result).isFalse(); // true
```

위에서 선언한 clone()은 super.clone()로 인해 부모 클래스인 Object의 clone()의 결과값을 반환하며, 그 인스턴스의 필드들을 복사한 인스턴스를 반환한다. 

이 때, Object는 CloneNotSupportedException을 던진다고 선언했지만, 재정의한 clone()에서는 throws를 없애는 것이 좋다. 이는 검사 예외를 던지지 않아야 메서드를 사용하기 편하기 때문이다.

### 복제된 객체의 불변식을 보장

clone()은 원본 인스턴스에 아무런 해를 끼치지 않는 동시에 복제된 인스턴스의 불변식을 보장해야 한다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public void push(Object e){
        ensureCapacity();
        elements[size++] = 0;
    }

    public Object pop(){
        if(size == 0){
            throw new EmptyStackException();
        }
        return elements[--size];
    }
    
    // 원소를 위한 공간을 적어도 하나 이상 확보한다.
    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2*size+1);
        }
    }

}
```

작성한 Stack 클래스는 가변 객체를 참조하고 있다. 

clone()이 단순히 super.clone의 결과를 그대로 반환한다면 어떻게 될까?

결과는 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조하게 된다. 이는 원본 혹은 복제본 중 하나를 수정하면 다른 한 쪽도 수정되어 불변식을 해치게 되어 프로그램이 의도치 않게 동작할 것이다.

이 때, Stack 클래스의 하나 뿐인 생성자를 호출한다면 이런 상황은 절대 일어나지 않는다. clone()은 사실상 생성자와 같은 효과를 낸다. 즉, 원본 인스턴스에 아무런 해를 끼치지 않는 동시에 복재된 인스턴스의 불변식을 보장해야 한다.

stack에서 clone()이 제대로 동작하려면 Stack의 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출하는 것이다.

```java
@Override
public Stack clone() {
    try{
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    }catch (CloneNotSupportedException e){
        throw new AssertionError();
    }
}
```

이 때, 배열의 clone은 Runtime type과 Compile type 모두가 원본 배열과 똑같은 배열을 반환하기때문에 별도로 캐스팅을 해줄 필요는 없다. 따라서 배열 복제 시 배열의 clone 메서드를 사용하라고 권장한다.

하지만 이러한 방식은 elements 필드가 final이었다면 필드가 새로운 값을 할당할 수 없어 clone()는 동작하지 않는다.

Cloneable 아키텍처는 가변 객체를 참조하는 필드는 final로 선언하라는 일반 용법과 충돌한다. 원본과 복제된 인스턴스가 가변 객체를 공유해도 된다면 상관없지만 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 키워드를 제거해야 할 수도 있다.
 
```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...ㅣ

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next){
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
    ...
    
    @Override
    public  HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
               result.buckets = buckets.clone();
               return result;
           } catch (CloneNotSupportedException e){
               throw new AssertionError();
           }
        }
}
```

위에서 재정의한 clone()은 자신만의 buckets 배열을 갖지만, 원본과 같은 연결 리스트를 참조하여 원본과 복제본 모두 예기치 않은 동작이 발생할 가능성이 생긴다.

이를 해결하려면 각 bucktes을 구성하는 LinkedList를 복사해야 한다.

```java
public class HashTable implements Cloneable{
    private Entry[] buckets = new Entry[...];

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next){
            this.key = key;
            this.value = value;
            this.next = next;
        }


        Entry deepCopy(){
            // 해당 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
            return new Entry(key, value,
                next == null ? null : next.deepCopy());
            }
        }

    @Override
    public  HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for(int i =0 ; i < buckets.length; i++){
                if(buckets[i] != null){
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e){
            throw new AssertionError();
        }
    }
}
```

private 클래스인 HashTable.Entry는 deep copy를 지원하도록 변경됐다.

clone()은 먼저 적절한 크기의 새로운 buckets 배열을 할당한 뒤 원래 buckets 배열을 순회하며 비지 않은 buckets에 대해 deep copy를 수행한다. 이 때, Entry의 deepCopy는 자신이 가르키는 LinkedList 전체를 복사하기 위해 자신을 재귀적으로 호출한다.

여기서 재귀 호출로 인한 리스트의 원소 수만큼 스택 프레임을 소비하여, 리스트가 길다면 스택 오버플로를 일으킬 위험이 생기게 된다. 이는 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

```java
Entry deepCopy(){

    Entry result = new Entry(key,value, next);
    for(Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

이제 super.clone()을 호출해 얻은 인스턴스의 모든 필드를 초기 상태로 설정하고 원본 인스턴스의 상태를 다시 생성하는 메서드를 호출한다.

## 정리

새로운 인터페이스를 만들 때 절대 Cloneable을 확장해선 안되며, 새로운 클래스도 이를 구현해선 안된다. 

item13은 기본 원칙은 복제 기능은 생성자와 팩토리를 이용하는 것이 최선이며 단, 배열만은 clone() 방식이 깔끔하며 규칙의 합당한 예외라 말한다.

## 참고자료

- 이펙티브 자바 3판
- https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#clone() - docs
- https://johngrib.github.io/wiki/java/object-clone/ - 기계인간님의 블로그
- 
