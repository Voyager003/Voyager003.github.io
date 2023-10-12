---
layout  : wiki
title   : 이왕이면 제네릭 타입으로 만들라 
summary : 
date    : 2023-10-12 18:59:48 +0900
updated : 2023-10-12 20:36:03 +0900
tag     : java effectivejava
resource: CA/AB2814-87F4-4F76-8882-A3F9CEE1E7E5
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 제네릭 클래스 만들기

먼저 클래스 선언 시, 타입 매개변수를 추가한다.

```java
// 일반 클래스
public class Stack {
    private Object[] elements;
    ...
    
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
    ...
}

// 제네릭 클래스
public class Stack<E> {
    private E[] elements;
    ...
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        E result = elements[size--];
        elements[size] = null;
        return result;
    }
    ...
}
```

타입 이름으로는 보통 E를 사용하며, 제네릭 필드를 쓴다는 것을 명시한다. 

추가적으로 타입 매개변수에 제약을 두는 제네릭 타입도 있다.

java.util.concurrent.DelayQueue가 그 예시이다.

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

타입 매개변수 목록인 <E extends Delayed>는 concurrent.Delayed의 하위타입만 받는다는 의미이다. 결과적으로 DelayQueue 자신과 DelayQueue를 사용하는 클라이언트는 DelayQueue의 원소에서 형변환 없이 곧바로 Delayed 클래스의 메서드를 호출할 수 있다.  

다시 본론으로 돌아와서, [item28](https://voyager003.github.io/wiki/java/effective_item28/#%EB%B0%B0%EC%97%B4%EA%B3%BC-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%83%80%EC%9E%85%EC%9D%98-%EC%B0%A8%EC%9D%B4) 에서 말했지만 E와 같은 실체화 불가 타입은 배열을 만들 수 없다. 따라서 배열을 사용하는 코드를 제네릭으로 만드려고 할 때는 두 가지 해결책이 있다.

- 제네릭 배열 생성을 금지하는 제약을 우회

```java
E[] elemet = (E[]) new Object[DEFAULT_INITAL_CAPACITY]; // unchecked cast
```

Object 배열을 생성하고 제네릭 배열로 형변환을 시도한다면, 컴파일러는 오류 대신 경고를 내보낼 것이다. 하지만 이는 일반적으로 타입 안정성을 해친다.

컴파일러는 작성한 프로그램이 타입 안전하지 증명할 수 없지만 코드를 작성하는 개발자라면 가능하다. 배열 elements는 priavet 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되는 일이 없다. 

push()를 통해 배열에 저장되는 원소의 타입은 항상 E로, 비검사 형변환은 안전하다.

```java
    // elements[]는 push(E)로 넘어온 E의 인스턴스만 담는다.
    // 런타임에는 E[]가 아닌 Object[]
    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }
```

비검사 형변환이 안전함을 증명했다면 범위를 좁혀 @SuppressWarnings 애노테이션으로 해당 경고를 숨긴다. 

결과적으로 Stack은 깔끔히 컴파일되고, 명시적 형변환 없이 ClassCastException 걱정 없이 사용 가능하다.

이 방법은 가독성이 좋으며, 배열 타입을 E[]로 선언해 오직 E타입 인스턴스만 받는다는 것을 확실히 알릴 수 있고 코드도 짧다.

하지만 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(heap pollution: item32)를 일으킨다.

- 필드 타입을 Object[]로 변경

```java
public class Stack<E> {
    
    private Object[] elements;
    ...
    
    public E pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }

        E result = elements[size--]; 
        elements[size] = null;
        return result;
    }
}
```

elements의 필드 타입을 Object[]로 변경하면 pop()의 E result = elements[size--]; 에서 오류대신 다음 경고가 발생한다.

```java
java: incompatible types: java.lang.Object cannot be converted to E
```

배열이 반환원소를 E로 형변환하면 이어서

```java
Unchecked cast: 'java.lang.Object' to 'E'
```

Unchecked cast 경고를 볼 수 있다. E는 런타임에 이뤄지는 형변환이 안전하지 증명할 방법이 없다. 이번에도 마찬가지로 직접 타입 안정성을 증명하고 경고를 숨길 수 있다.

```java
// push에서 E타입만 허용하므로 안전
@SuppressWarnings("unchecked")E result = (E) elements[size--];
```

두 번째 방식은 배열에서 원소를 읽을 때 마다 형변환을 해줘야한다는 특징이 있으며, 첫째 방식과 달리 E를 사용하지 않아 힙오염의 우려가 없다.

## 정리 

제네릭 타입 안에서 리스트를 사용하는 것이 항상 가능한 것도, 꼭 더 좋은 것도 아니다. Java는 List를 기본 타입으로 제공하지 않으므로 ArrayList같은 제네릭 타입도 결국 기본 타입인 배열을 사용해 구현해야한다. 또한 HashMap같은 제네릭 타입은 성능 목적으로 배열을 사용하기도 한다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다. 읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판
