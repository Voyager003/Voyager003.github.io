---
layout  : wiki
title   : 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라 
summary : 
date    : 2024-06-10 11:07:50 +0900
updated : 2024-06-10 14:42:21 +0900
tag     : java effectivejava
resource: 1B/238E95-6672-4D4E-BB12-ECF829671E38
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 싱글톤 패턴과 직렬화

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
```

위 코드는 item3에서 다뤘던 싱글톤 패턴의 예이다. 바깥에서 생성자를 호출하지 못하게 막음으로써 인스턴스가 오직 하나만 만들어지는 것을 보장했다.

이 코드에 직렬화를 적용하도록 implements Serializable을 추가하면 어떻게 될까?

기본 직렬화를 쓰지 않고, readObject 메서드를 제공하더라도 추가하는 순간 싱글톤이 아니게 된다.

이유는 직렬화된 객체가 역직렬화가 되면 새로운 객체가 생성되며, 역직렬화 과정에서 readObject 메서드는 새 인스턴스를 만들게되어 싱글톤의 속성이 깨지게 되는 것이다. 

하지만 **readResolve** 기능을 이용하면 readObject가 만들어낸 인스턴스를 다른 것으로 대체 가능ㅎ다. 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의했다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다. 

이때 새로 생성된 객체의 참조는 유지하지 않아 GC의 대상이된다. 

앞서 살펴보면 Elive 클래스가 Serializable을 구현한다면 다음의 readResolve 메서드를 추가해 싱글톤이라는 속성을 유지할 수 있다. 

```java
private Object readResolve() {
    // 진짜 Elvis를 반환하고, 가짜 Elvis는 GC에 맡김
    return INSTANCE;   
}
```

readResolve는 역직렬화한 객체는 무시하고 클래스초기화 시 만들어진 Elivs 인스턴스를 반환한다. 이 때 readResolve 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드를 모두 transient로 선언해야 한다. 

싱글톤이 transient가 아닌 참조 필드를 가진다면, 그 필드의 내용은 readResolve 메서드가 실행되기 전에 역직렬화된다. 잘 조작된 스트림은 해당 참조 필드의 내용이 역직렬화되는 시점에 인스턴스의 참조를 훔쳐올 수 있다. 

item89가 제시하는 잘못된 싱글톤을 통해 살펴보자.

```java
// tranient가 아닌 참조 필드를 가지는 싱글톤
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}

// 싱글턴의 비휘발성 인스턴스 필드를 훔쳐러는 Stealer(도둑) 클래스
public class ElvisStealer implements Serializable {
    private static final long serialVersionUID = 0;
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
        // resolve되기 전의 Elvis 인스턴스의 참조를 저장
        impersonator = payload;
        // favoriteSongs 필드에 맞는 타입의 객체를 반환
        return new String[] {"There is no cow level"};
    }
}
```

readResolve 메서드와 인서튼서 필드 하나를 포함한 Stealer 클래스에서는 인스턴스 필드가 Stealer가 숨길 직렬화된 싱글톤을 참조하는 역할을 한다. 직렬화된 스트림에서 싱글톤의 비휘발성 필드를 이 Stealer의 인스턴스로 교체한다. 이로써 싱글톤은 Stealer를 참조하고 Stealer는 싱글톤을 참조하는 순환 고리가 만들어진다.  

```java
// 직렬화의 약점을 이용해 싱글턴 객체를 2개 생성
public class ElvisImpersonator {
    private static final byte[] serializedForm = new byte[]{
        (byte) 0xac, (byte) 0xed, 0x00, 0x05, 0x73, 0x72, 0x00...
        ...
    };    

    private static Object deserialize(byte[] sf) {
            try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(sf)) {
        try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
        return objectInputStream.readObject();
    } catch (IOException | ClassNotFoundException e) {
        throw new IllegalArgumentException(e);
    }
}

public static void main(String[] args) {
    // ElvisStealer.impersonator 를 초기화한 뒤,
    // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;
    
    elvis.printFavorites(); // [Hound Dog, Heartbreak Hotel]
    impersonator.printFavorites(); // [There is no cow level]
    }
}
```

직렬화의 허점을 이용해 싱글톤 객체 2개를 생성한 뒤 출력한 결과로, 서로 다른 2개의 Elvis 인스턴스를 생성한 것을 확인했다.

이러한 문제는 favoriteSongs 필드를 transient로 선언해 해결할 수 있지만, Elvis 원소 하나짜리 ENUM 타입으로 바꾸는 편이 나은 선택이다.

이처럼 readResolve 메서드를 사용해 순간적으로 만들어진 역직렬화된 인스턴스에 접근하지 못하게 하는 방법은 깨지기 쉬우며 신경써야하는 작업이다.

### ENUM 도입 및 주의점

직렬화 가능한 인스턴스 통제 클래스를 ENUM 타입을 이용해 구현하면 선언한 상수 외의 다른 객체는 존재하지 않음을 Java가 보장한다. 

여기서 공격자가 AccessibleObject.setAccessible 같은 특권(privileged) 메서드를 악용한다면, 임의의 네이티브 코드를 수행할 수 있어 모든 방어법이 무력화된다.

다음은 Elvis를 ENUM으로 구현한 코드이다.

```java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = 
        { "Hound Dog", "Heartbreak Hotel" };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

readResolve메서드의 접근성은 매우 중요하다. 

final 클래스라면 readResolve 메서드는 private이어야 하며, final이 아닌 클래스에서는 다음 몇가지를 주의해야 한다.

- private 

private으로 선언 시, 하위 클래스에서 사용할 수 없다.

- package private

같은 패키지에 속한 하위 클래스에서만 사용 가능하다. 

- protected, public

이를 재정의하지 않은 모든 하위 클래스에서 사용 가능하다. 재정의하지 않았다면 하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.

## 참고자료

- 이펙티브자바 3판

