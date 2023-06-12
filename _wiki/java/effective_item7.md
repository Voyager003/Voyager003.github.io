---
layout  : wiki
title   : 다 쓴 객체 참조를 해제하라 
summary : 
date    : 2023-06-12 20:00:38 +0900
updated : 2023-06-12 23:07:22 +0900
tag     : java effecitivejava
resource: A3/630AC1-9661-45DA-B351-85E6851BC65A
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 메모리 누수(Memory Leak)가 발생하는 위치

```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = 0;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

item7의 예제는 elements 배열을 DEFAULT_CAPACITY만큼의 크기로 초기화하고, 저장 공간 유무를 확인하고 push, pop하는 로직을 가진다.

Java의 GC가 메모리를 관리를 해주기 때문에 자원을 해제하지 않아도 될 것이라 생각하지만, 실제로 그렇지 않다.

프로그램에서 그 인스턴스를 사용하지 않더라도, Stack에 push, pop을 반복해도 Stack이 차지한 메모리는 줄어들지 않는다. 

이유는 Stack이 늘어났다가 줄어드는 경우에 Stack이 해당 인스턴스들의 다 쓴 참조(obsolete reference)를 가지고 있기 때문이다.

객체 참조 하나를 살려두면 GC는 그 인스턴스 뿐만 아니라, 그 인스턴스가 참조하고 있는 모든 인스턴스를 회수하지 못한다.

이처럼 필요없는 메모리를 차지하고 있는 문제를 어떻게 해결할까?

```java
public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
```

pop()을 다 쓴 참조를 null을 처리하도록 변경했다.

null 처리한 참조를 사용한다고해도 프로그램은 NullPointerException을 던져 종료하게 된다.

하지만 모든 인스턴스를 다 쓰자마자 null 처리하는 것은 프로그램을 필요 이상으로 지저분하게 만들 수 있다.

객체 참조를 null 처리하는 것은 예외적인 경우여야 하며, 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것이다.

지역 변수(Local Variable)를 사용하면 메서드 영역 블럭을 벗어날 때 무의미한 레퍼런스가 되기 때문에 GC에 의해 정리가 된다.

item7의 Stack은 명시적으로 null로 설정하지 않는다면 GC에 의해 정리되지 않기 때문에 예외적인 상황인 것이다.

## 캐시

캐시 사용 시에도 메모리 누수를 주의해야 한다. 인스턴스의 레퍼런스를 캐시에 넣고 캐시를 비우는 것을 잊기 쉽다. 

이는 캐시의 키에 대한 레퍼런스가 캐시 밖에서 필요 없어지면 해당 엔트리를 캐시에서 자동으로 비워주는 WeakHashMap을 사용하여 해결할 수 있다.

설명에 앞서 Java 레퍼런스의 종류를 짚고 넘어가자.

### GC 처리를 위한 레퍼런스

GC 처리를 위한 Reference는 Strong Reference, Soft Reference, Weak Reference, Phantom Reference로 구분되며, 뒤로 갈수록 GC에 의해 제거될 우선순위가 높다.

- Strong Reference

```java
Integer i = 1;
```

일반적으로 사용되는 레퍼런스 유형으로, String Reference를 가진다면 GC는 해당 객체를 회수하지 않는다.

- Soft Reference

```java
SoftReference<String> softRef = new SoftReference<>("softRef"); 
```

최초 생성 시점에 이용 대상이 되었던 String Reference는 없고 해당 객체를 참조하는 객체가 Soft Reference만 존재할 경우 GC의 대상이 된다.

이 때, 메모리가 부족하지 않으면 GC는 해당 객체를 회수하지 않는다.

- Weak Reference

```java
WeakReference<String> weakRef = new WeakReference<>("weakRef"); 
```

최초 생성 시점에 이용 대상이 되었던 Strong Reference는 없고, 해당 객체를 참조하는 객체가 Weak Reference만 존재할 경우 GC의 대상이 된다.

이 때, 메모리가 부족하지 않아도 바로 GC의 대상이 된다.

- Phantom Reference

```java
ReferenceQueue<String> queue = new ReferenceQueue<>();
PhantomReference<String> phantomRef = new PhantomReference<>("phantomRef", queue);
```

GC의 과정은 GC 대상 객체를 찾는 작업, 객체를 처리(finalize), 메모리 회수 단계로 나뉜다.

이 때, Soft와 Weak의 경우 GC의 대상 객체를 찾는 작업에 개발자가 관여할 수 있게 해주는 반면, Phantome Reference의 경우 객체를 처리하고 메모리를 회수하는 것에 관여한다.

다시 돌아와서 item7의 WeakHashMap을 살펴보자.

```java
public class CacheEx {
    
    public static void main(String[] args) {
        
        Object key1 = new Obejct();
        Object value1 = new Object();
        
        Map<Object, Object> cache = new HashMap<>();
        cache.put(key1, value1);
    }
}    
```

일반 Hashmap의 경우 어떤 인스턴스가 null이 된다면 해당 객체를 key로 하는 HashMap의 key1이 쓸모가 없어져 더 이상 꺼낼일이 없는 경우가 발생하면 메모리를 차지하게 된다. 

```java
Map<Object, Object> cache = new WeakHashMap<>();
cache.put(key1, value1);
```

WeakHashMap을 사용한 예시이다. 

key1에 대한 레퍼런스를 weakReference를 한번 감싸게 되는데, 이는 Strong Reference(위의 경우 new Object()) 가 없어진다면 GC의 대상이 되어 메모리를 회수하게 된다. 

이 때, 주의할 점은 WeakHashMap의 Value는 Strong Reference에 의해 보관 유지되는데 Value 객체가 직/간접적으로 자신의 Key를 Strong Reference하지 않도록 주의해야 한다. 

Key를 참조하는 Value를 사용해 WeakHashMap이 올바르게 동작하길 원한다면 WeakReference로 Wrapping하는 방식을 사용하자.

```java
weakHashMap.put(key, new WeakReference(value));
```

## 참고자료

- 이펙티브 자바 3판
- https://www.youtube.com/watch?v=YijcBaS4cu8&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=7&ab_channel=%EB%B0%B1%EA%B8%B0%EC%84%A0 - 백기선님의 강좌
- https://docs.oracle.com/javase/8/docs/api/java/util/WeakHashMap.html - weakHashMap docs

