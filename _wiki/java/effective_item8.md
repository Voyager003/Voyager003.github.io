---
layout  : wiki
title   : finalizer와 cleaner 사용을 피하라 
summary : 
date    : 2023-06-19 09:53:51 +0900
updated : 2023-06-19 14:27:47 +0900
tag     : java effectivejava
resource: 2A/CCA62E-CFE1-4E24-AA08-A2920C4AB04C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 객체 소멸자(Destructor)

Java는 두 가지 소멸자를 제공한다. (이 때, C++의 Destructor와는 다른 개념이다.)

**finalizer**와 **cleaner**가 그 소멸자로, GC가 더 이상 사용하지 않는 자원에 대한 정리작업을 하기위해 사용된다.

item8에서는 이 두가지 소멸자는 예측 불가능하며, 일반적으로 불필요하기 때문에 사용을 지양하라고 말한다. (finalizer의 경우 Java 9에서 deprecated)

## 원인

### finalizer, cleaner는 즉시 수행된다는 보장이 없다.

인스턴스에 접근할 수 없게 된 후에 finalizer와 cleaner가 실행되기까지 어느정도 걸리는 지 알 수 없다.

예를 들어, 파일 닫기를 finalizer 혹은 cleaner에 맡긴다면 시스템이 동시에 열 수 있는 파일 개수의 한계가 있기 때문에 중대한 오류를 일으킬 수 있다.

```java
Finalization is a feature of the Java programming language that allows you to perform postmortem cleanup on objects that the garbage collector has found to be unreachable. 
It is typically used to reclaim native resources associated with an object. Here's a simple finalization example:
            
public class Image1 {

    // pointer to the native image data
    private int nativeImg;
    private Point pos;
    private Dimension dim;

    // it disposes of the native image;
    // successive calls to it will be ignored
    private native void disposeNative();
    public void dispose() { disposeNative(); }
    protected void finalize() { dispose(); }

    static private Image1 randomImg;
    }
```

finalize()의 호출 시점은 finalization queue에 등록이 되고, reclaimed되기 전이다. 호출이 되면 finalization이 된 object로 판단하고 기록하게 된다.

하지만 GC에서 Mark and Sweep 과정을 거치면서 마킹을 해서 소멸되어야할 객체로 인식하고 작업을 수행하기 전까지는 시간이 걸리게 된다.

또한 Java에서 할당할 가용 메모리가 부족한 경우(명시적으로 해제가 되는 경우가 아닐 때) GC가 작동하기 때문에 finalizer를 완벽하게 수행하지 못하는 경우가 생기게 된다.

따라서 상태를 영구적으로 수정하는 작업에서는 절대 finalizer, clenaer에 의존해서는 안된다.

## 성능 문제

AutoCloseable 객체를 생성해 GC가 수거하기 까지 12ns가 걸린 것이, finalizer를 사용하니 550ns가 걸렸다. finalizer가 GC의 효율을 떨어뜨리기 때문이다.

cleaner의 경우도 클래스의 모든 인스턴스를 수거하는 형태로 사용하면 finalizer와 성능은 유사하며, 안정망 방식을 사용하면 약 66ns가 걸리나, 안전망을 설치하면 성능이 약 5배 정도 느려진다.

## finalizer를 사용한 클래스의 보안 문제

생성자나 직렬화 과정에서 예외가 발생하면, 생성되다 만 객체에서 악의적으로 하위 클래스의 finalizer가 수행될 수 있다. 

```java
public class ParentClass {
    public ParentClass() {
        if (isEvil()) {
            throw new RuntimeException("악의적인 동작 발생");
        }
    }

    protected boolean isEvil() {
        return false;
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("부모 클래스의 finalizer 수행");
        super.finalize();
    }
}

public class EvilChildClass extends ParentClass {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("악의적인 하위 클래스의 finalizer 수행");
        // 악의적인 동작 수행
    }

    @Override
    protected boolean isEvil() {
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        try {
            ParentClass parent = new EvilChildClass();
        } catch (RuntimeException e) {
            e.printStackTrace();
        }
    }
}
```

ParentClass는 부모 클래스로 생성자에서 isEvil() 메서드를 호출해 악의적 동작을 판별한다. 이 때 반환 값이 true일 때 예외를 발생시키는 경우이다.

EvilChildClass는 ParentClass의 하위 클래스로 finalize()를 재정의하여 악의적 동작을 수행하도록 구현했다.

Main 클래스에서 EvilChindClass의 인스턴스를 생성하려고할 때, ParentClass에서 예외가 발생한다. 이는 EvilChildClass의 인스턴스는 완전히 생성되지 않지만 이미 생성된 인스턴스에 대해 GC가 동작하여 finalizer를 호출하여 EvilChildClass의 finalizer가 수행되는 악의적 동작이 발생하게 된다.

finalizer는 정적 필드에 자신의 참조를 할당해 GC가 수집하지 못하게 막을 수 있다.

```java
public class ParentClass {
    private static ParentClass instance;

    public ParentClass() {
        instance = this; // static 필드에 자기 자신의 참조 할당
        if (isEvil()) {
            throw new RuntimeException("악의적인 동작 발생");
        }
    }

    protected boolean isEvil() {
        return false;
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("부모 클래스의 finalizer 수행");
        super.finalize();
    }

    public static void main(String[] args) {
        try {
            ParentClass parent = new EvilChildClass();
        } catch (RuntimeException e) {
            e.printStackTrace();
        }
    }
}

public class EvilChildClass extends ParentClass {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("악의적인 하위 클래스의 finalizer 수행");
        // 악의적인 동작 수행
        instance = this; // static 필드에 자기 자신의 참조 재할당
    }

    @Override
    protected boolean isEvil() {
        return true;
    }
}
```

ParentClass의 main 메서드에서 EvilChildClass의 인스턴스를 생성하려고 할 때, ParentClass의 생성자에서 예외가 발생한다. 

이로 인해 EvilChildClass의 인스턴스는 완전히 생성되지 않지만, 이미 생성된 객체에 대해 가비지 컬렉터가 동작하여 finalizer를 호출한다.

EvilChildClass의 finalizer에서는 instance 필드에 자기 자신의 참조를 재할당하므로, 해당 인스턴스는 가비지 컬렉터에 의해 수집되지 않게 되는 것이다.

자기 자신의 레퍼런스를 정적 필드에 할당하여 GC를 속이는 것은 메모리 누수 및 GC 성능에 영향을 미칠 것이라고 생각하는데 옳은 방법인지는 모르겠다.

## finalizer, cleaner를 언제 사용해야할까?

item8은 자원의 소유자가 close 메서드를 호출하지 않은 것에 대비한 안정망 역할이라고 말한다.

```java
public class ResourceOwner {
    private Resource resource;

    public ResourceOwner() {
        resource = new Resource();
    }

    public void close() {
        if (resource != null) {
            resource.release(); // 리소스 정리 작업 수행
            resource = null;
        }
    }

    @Override
    protected void finalize() throws Throwable {
        try {
            if (resource != null) {
                resource.release(); // 리소스 정리 작업 수행
            }
        } finally {
            super.finalize();
        }
    }

    public static void main(String[] args) {
        ResourceOwner owner = new ResourceOwner();
        owner = null; // owner 변수에 대한 참조 제거

        // 가비지 컬렉터가 동작하면 finalize() 메서드가 호출
    }
}

class Resource {
    public void release() {
        // 리소스 정리 작업 수행
        System.out.println("리소스 정리 작업 수행");
    }
}
```

ResourceOwner 클래스는 Resource라는 리소스를 소유하는 클래스다. 생성자에서 Resource 인스턴스를 생성하여 resource 필드에 할당한다.

ResourceOwner 클래스에는 close()가 있으며, resource를 확인하고 정리 작업을 수행한 후 해당 리소스에 대한 참조를 해제한다.

또한, ResourceOwner 클래스에는 finalize() 메서드가 재정의되어 있다. 이는 자원 소유자가 close() 메서드를 호출하지 않은 경우에 대비하여 resource를 확인하고 리소스 정리 작업을 수행한다. finalize()가 호출된 이후에는 super.finalize()를 호출하여 기본 finalizer 동작을 수행한다.

main 메서드에서 ResourceOwner 인스턴스를 생성, owner 변수에 할당한 후 owner 변수에 null을 할당하여 해당 인스턴스에 대한 참조를 제거한다. 

이렇게 되면 GC가 동작할 때 finalize() 메서드가 호출되어 안전망 역할을 수행하게 된다.

또한 Java 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 피어(native peer)와 연결된 네이티브(native) 자원의 정리에 사용한다.

Java 객체가 아니므로 GC가 관리하는 대상이 아니기 때문이다. 

이는 finalizer를 명시적으로 호출함으로써 자원을 회수할 수 있다.

다만 item8은 성능 저하를 감당할 수 있고, 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당하며 자원을 즉시 회수해야 한다면 close 메서드를 사용해야한다고 말한다.

### cleaner 사용 시

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다! 
    private static class State implements Runnable {
        int numJunkPiles; // 방(Room) 안의 쓰레기 수

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출
        @Override public void run() {
            Syste.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleannable과 공유
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override 
    public void close() {
        cleanable.clean();
    }
}
```

static으로 선언된 중첩 클래스 State는 cleaner가 방을 청소할 때 수거할 자원을 담고 있다.

State는 Runnable을 구현하고, run()은 cleanable에 의해 한 번만 호출이 된다. 

cleanable 인스턴스는 Room 생성자에서 cleaner에 Room과 State를 등록할 때 얻게된다.

보통은 Room의 close() 호출하고 close()에서 Cleanable의 clean()을 호출하면 이 메서드에서 run을 호출하거나, GC가 Room 인스턴스를 회수할 때까지 클라이언트가 close를 호출하지 않는다면 cleaner가 State의 run을 호출할 것이다. 

이 때, State 인스턴스는 Room 인스턴스를 참조해선 안된다. 이유는 순환 참조가 생겨 GC가 Room의 인스턴스를 회수할 기회가 오지 않는다. 

이는 State가 정적 중첩 클래스인 이유이며, 정적이 아닌 중첩 클래스는 자동으로 바깥 인스턴스의 참조를 갖게 된다.

```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("hi");
        }
    }
}
```

이 코드에서는 기대한 대로 Adult는 "hi"를 출력하고, 방 청소를 출력하게 될 것이다. 다음 코드를 보자.

```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("hi");
    }
}
```

Room 클래스의 생성자에서는 cleaner.register(this, state)를 호출하여 cleanable 객체를 등록했다.

그러나 cleanable 객체의 clean() 메서드는 호출되지 않았으므로, state.run()에서 "방 청소"가 실행되지 않는다.

## 참고자료

- 이펙티브 자바 3판
- https://hodongman.github.io/2019/11/30/Java-Finalize().html
