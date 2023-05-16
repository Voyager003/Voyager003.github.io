---
layout  : wiki
title   : private 생성자나 열거 타입으로 Singleton임을 보증하라 
summary : 
date    : 2023-05-15 19:14:18 +0900
updated : 2023-05-16 10:49:07 +0900
tag     : java effectivejava
resource: 7D/64EEF1-C4BD-4655-8812-C3366641F2D4
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Singleton(싱글톤)

> 소프트웨어 디자인 패턴에서 싱글톤 패턴을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 반환한다.

- 싱글톤의 전형적인 예로 함수와 같은 stateless 객체 혹은 설계상 유일해야 하는 시스템 컴포넌트를 들 수 있다.
- 하지만 클래스를 싱글톤으로 만들면 사용하는 클라이언트를 테스트하기 어려워질 수 있다   
  - 이는 타입을 인터페이스로 정의한 뒤, 그 인터페이스를 구현해서 만든 싱글톤이 아니라면 싱글톤 인스턴스를 mock 구현으로 대체할 수 없기 때문이다.
- 싱글톤을 만드는 방법은 다음과 같은 방법이 있다.

### public static final 필드

```java
package effectivejava.practicecode.item3;

public class Singleton1 {

    public static final Singleton1 INSTANCE = new Singleton1();

    private Singleton1() {
    }
}
```

- private 생성자인 Singleton1()은 Singleton1.INSTANCE를 초기화할 때 한 번만 호출된다.
- public, protected 생성자가 없으므로 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐이라는 것이 보장된다.
- public 필드 방식은 해당 클래스가 싱글톤임이 API에 명백히 드러나게 되며, 코드가 간결하다.

### 정적 팩토리 메서드 

```java
package effectivejava.practicecode.item3;

public class Singleton2 {
    private static final Singleton2 INSTANCE = new Singleton2();
    private Singleton2() {
    }

    public static Singleton2 getInstance() {
        return INSTANCE;
    }
}
```

- item3에서 말하는 정적 팩토리 메서드가 가지는 장점은 다음과 같다.
- Singleton2 역시 Singleton2.INSTANCE는 항상 같은 인스턴스의 레퍼런스를 반환하여 제2의 Singleton2 인스턴스는 만들어지지 않는다.
- 정적 팩토리 메서드는 API를 바꾸지 않고 싱글톤이 아니게 변경할 수 있다.
    - return INSTANCE 대신 return new Singelton2()로 바꾸게되면 Singelton 인스턴스가 아니게 된다.
  
- 정적 팩토리를 제네릭 싱글톤 팩토리로 만들 수 있다.
    ```java
    public class GenericFactory {
        public static final Set EMPTY_SET = new HashSet();
        
        public static final <T> Set<T> emptySet() {
            return (Set<T>) EMPTY_SET;
        }
    }
    ```
    - 제네릭으로 type 설정이 가능한 인스턴스를 만들어두고, 반환 시 제네릭으로 받은 타입을 이용해 타입을 결정하는 것이다.
    - 이는 여러 타입으로 내부 인스턴스를 받아도 에러가 발생하지 않아 코드의 유연성을 높인다.

- 정적 팩토리의 메서드 참조를 supplier로 사용할 수 있다.
    ```java
    Supplier<Singleton2> sp = Singleton2::getInstance;
    ```
    - supplier는 매개변수를 받지않고 단순히 반환하는 추상메서드가 존재한다.
    - 정적 팩토리 메서드로 작성한 레퍼런스를 supplier로 사용할 수 있다.
    - 이로써 함수를 변수화 하여 직접 계산하지 않아도 된다는 장점이 생긴다.(Lazy Evaluation)

## 직렬화(Serialization)

**직렬화**는 Java 시스템 내부에서 사용되는 인스턴스, 데이터를 외부의 Java 시스템에서도 사용할 수 있도록 byte 형태로 데티어 변환하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 역직렬화하는 것을 아우르는 용어이다.

설명한 두 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순 Serializable을 구현한다고 선언하는 것만으로 부족하다.

싱글톤 클래스는 직렬화가능한 클래스가 되기 위해 Serializable 인터페이스를 구현하는 순간, 직렬화를 통해 초기화해둔 인스턴스가 아닌 다른 인스턴스가 반환된다. 

이는 직렬화과정에서 wirteObject(), 역직렬화과정에서 readObject()가 자동으로 호출되기 때문이다.

코드로 확인해보자.

```java
public void Test2() throws Exception {

        Singleton2 instance1 = Singleton2.getInstance();

        // 직렬화
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(instance1);
        oos.close();

        // 역직렬화
        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);
        Singleton2 instance2 = (Singleton2) ois.readObject();
        ois.close();

        assertNotSame(instance1, instance2);
    }
```

싱글톤으로 생성한 인스턴스 instance1과 직렬,역직렬화를 거친 인스턴스 instance2를 비교해보면 서로 다른 인스턴스임을 확인할 수 있다.

이러한 문제는 Serializable의 숨겨진 readResolve()를 사용하여 해결할 수 있다.

```java
private Object readResolve() {
    return INSTANCE;
}
```

readResolve()를 직접 정의하여 역직렬화 과정에서 만들어진 인스턴스 대신 기존에 생성된 싱글톤 인스턴스를 반환하도록 한다. 이는 역직렬화 과정에서 자동으로 호출되는 readObejct()가 있더라도 readResolve()에서 반환한 인스턴스로 대체된다. readObject()를 통해 만들어진 인스턴스는 GC의 대상이 된다.

```java
// Singleton3
public class Singleton3 implements Serializable {

    private static final Singleton3 INSTANCE = new Singleton3();
    private Singleton3() {
    }

    public static Singleton3 getInstance() {
        return INSTANCE;
    }

    private Object readResolve() {
        return INSTANCE;
    }
}

// Test
@Test
@DisplayName("readResolve를 정의한 싱글톤 클래스 객체")
public void Test3() throws Exception {

    Singleton3 instance1 = Singleton3.getInstance();

    // 직렬화
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(baos);
    oos.writeObject(instance1);
    oos.close();

    // 역직렬화
    ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
    ObjectInputStream ois = new ObjectInputStream(bais);
    Singleton3 instance2 = (Singleton3) ois.readObject();
    ois.close();

    assertSame(instance1, instance2);
}
```

- readResolve()에서 싱글톤 인스턴스를 반환하도록 정의하고, 역직렬화 과정에서 만들어진 인스턴스와 비교했을 때 같은 인스턴스를 반환하여 테스트를 성공하는 것을 확인할 수 있다.

## ENUM

하지만 이는 너무 복잡하다. item3에서 제시하는 또 다른 방법은 원소가 하나인 ENUM(열거)타입을 선언하는 것이다.

```java
public enum SingletonEnum {
    INSTANCE;
}
```

ENUM 타입은 JVM내에 하나의 싱글톤 인스턴스만 존재한다는 것이 100% 보장된다.
또한 본질적으로 직렬화가 가능하므로 직렬화 가능 인터페이스로 구현할 필요도 없으며, Reflection 문제에도 안전하다.

```java
SingletonEnum instance1 = SingletonEnum.INSTANCE;

        // 직렬화
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(instance1);
        oos.close();

        // 역직렬화
        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);
        SingletonEnum instance2 = (SingletonEnum) ois.readObject();
        ois.close();

        assertSame(instance1, instance2);
```

- 별도의 Serialzable 인터페이스와 readResolve() 정의 없이도 같은 인스턴스를 반환하는 것을 확인할 수 있다.
- 단, 만드려고 하는 싱글톤이 Enum외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다. 

## 참고자료 
- https://www.youtube.com/watch?v=xBVPChbtUhM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=4&t=905s&ab_channel=%EB%B0%B1%EA%B8%B0%EC%84%A0 
- https://madplay.github.io/post/what-is-readresolve-method-and-writereplace-method - readResolve
- https://dzone.com/articles/java-singletons-using-enum
