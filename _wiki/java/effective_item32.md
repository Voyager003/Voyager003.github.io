---
layout  : wiki
title   : 제네릭과 가변인수를 함께 쓸 때는 신중하라 
summary : 
date    : 2023-10-26 10:02:51 +0900
updated : 2023-10-30 13:13:51 +0900
tag     : java effectivejava
resource: CC/45FB7A-52DA-4C6C-8F9B-DD6621D35B09
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 가변 인수(varargs)와 타입 안전성 문제

가변인수는 메서드에 넘기는 인수의 갯수를 클라이언트가 조절할 수 있게 해준다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(sum(1, 2, 3, 4, 5));  // 가변 인수를 사용하여 여러 개의 인자를 넘김
    }
    
    public static int sum(int... numbers) {  // 가변 인수 선언
        int total = 0;
        for(int num : numbers) {
            total += num;
        }
        return total;
    }
}
```

매개변수 선언 시, 타입 뒤에 '...'를 명시하면 가변 인수를 받도록 만들 수있다.

하지만 가변인수는 구현 방식에 허점이 있다. 가변인수 메서드 호출 시, 가변인수를 담기 위한 배열이 자동으로 하나 만들어지는데 이 배열이 외부 클라이언트에 노출되는 문제가 생기는 것이다. 

```java
public <T> void add(T... elements) {
    Object[] objectArray = elements; // Valid
    objectArray[0] = new Object(); // Heap pollution warning occurs
    T t = elements[0]; // ClassCastException throw
}
```

가변 인자는 내부적으로 배열로 처리되고, 이 배열은 공변성을 가진다. 즉, Integer[]는 Object[]의 하위 타입이며 이 특성때문에 런타임에 타입 오류가 발생할 수 있다.

 add() 메서드는 제네릭 타입 T의 가변 인자를 받아서 Object[]로 형 변환하는 부분이 문제이다. 
 
 이렇게 되면 objectArray[0] = new Object();에서 원래의 배열에 예상치 못한 타입(Object)의 객체가 추가되어 힙 오염(heap pollution)이 발생한다.
 
 > 여기서 힙 오염이란 매개변수화된 유형의 변수가 해당 매개변수화된 유형이 아닌 개체를 참조할 때 발생하는 것이다 [^1]
 
이 후, 나중에 원래의 타입(T)으로 객체를 가져오려 할 때 (T t = elements[0];) ClassCastException이 발생하게 되는 것이다.

이처럼 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생하며, 제네릭 타입 시스템이 약속한 타입 안전성의 근간이 흔들리기 때문에 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.

하지만 이 가변인수 매개변수는 실무에서 매우 유용하기 때문에 언어 설계자는 이 모순을 수용하기로 했다.

## @SafeVarargs 애노테이션

Java 7 이전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었지만, Java 7에서 추가된 @SafeVarargs 애노테이션은 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다.

```java
// Collections.addAll() - 언어 설계자가 모순을 수용한 예
@SafeVarargs
public static <T> boolean addAll(Collection<? super T> c, T... elements) {
    boolean result = false;
    for (T element : elements)
        result |= c.add(element);
    return result;
}
```

@SafeVarags 애노테이션의 기능은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치로 컴파일러는 이 메서드가 안전하지 않을 수 있다는 경고를 표시하지 않는다.

주의할 점은 메서드가 안전한 게 확실하지 않다면 절대 @SafeVarargs 애노테이션을 명시해선 안된다. 메서드가 안전한지 판단하려면 위에서 설명했듯이, 가변인수 메서드를 호출할 때 가변인수 매개변수를 담는 제네릭 배열이 만들어진다는 사실을 기억하자. 

메서드에서 이 배열에 아무것도 저장하지 않고, 참조가 밖으로 노출되지 않는다면 타입 안전하다. 즉 매개변수 배열이 그 메서드로 순수하게 인수들을 전달하는 역할만 한다면 메서드가 안전하다고 확신할 수 있다.

### 주의점

주의점으로는 varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안정성을 깰 수도 있다. 

```java
static <T> T[] toArray(T... args) {
    return args;
}
```

이 코드에서 메서드가 반환하는 배열의 타입은 메서드에 인수를 넘기는 컴파일 타임에 결정된다. 하지만 그 시점에는 컴파일러에게 충분한 타입 정보가 주어지지 않아 잘못 판단할 수 있다.

```java
Integer[] integerArray = new Integer[]{1, 2, 3};
Object[] objectArray = toArray(integerArray);

objectArray[0] = "varargs";
```

toArray() 메서드를 Integer 타입의 배열을 입력으로 받아 Object 타입의 배열로 반환하는 코드이다. objectArray는 실제로 Integer 객체를 저장하고 있는 배열을 참조하게 되는데, objectArray[0]에 문자열을 삽입하게 되면 문제가 생긴다.

objectArray가 Object[] 타입이므로 문자열 "varargs"를 저장할 수 있다고 생각했지만, 실제로 이 배열은 원래의 integerArray와 동일한 객체를 참조하고 있으므로, 여기에 문자열을 저장하려 하면 ArrayStoreException이 발생한다.

즉, 컴파일 시점에는 문제가 없어 보였지만 런타임 시점에서 예상치 못한 오류가 발생한 것이다. 이는 이 메서드를 호출한 쪽의 콜스택까지 전이하는 결과를 낳을 수 있다.

제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점을 항상 염두에 두자.

### 예외 케이스

예외 케이스 중 첫 번째는 @SafeVarags 제대로 명시된 또 다른 varargs 메서드에 넘기는 것은 안전하다. 이를 판단하는 조건은 다음과 같다.

1. varags 매개변수 배열에 아무것도 저장하지 않으며,
2. 그 배열 혹은 복제본을 신뢰할 수 없는 코드에 노출하지 않는 것이다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

위 예제는 varargs 매개변수를 안전하게 사용하는 예이다. varargs 배열을 직접 노출하지 않고, T타입의 와일드 카드를 사용하여 ClassCastException이 발생할 일이 없어진다. 

또 다른 방법은 varargs 매개변수를 List로 대체하는 방법이다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

매개변수만 List로 수정한 코드이다. 이 코드에서는 List.of()를 활용하면 

```java
audience = flatten(List.of(friends, romans, countrymen));
```

위와 같이 임의 개수의 인수를 넘길 수 있다. 

이 방법이 가능한 이유는 정적 팩토리 메서드인 List.of에 @SafeVarargs 애노테이션이 명시되어 있기 때문이다.

```java
@SafeVarargs
@SuppressWarnings("varargs")
static <E> List<E> of(E... elements) {
    switch (elements.length) { // implicit null check of elements
        case 0:
            @SuppressWarnings("unchecked")
        var list = (List<E>) ImmutableCollections.EMPTY_LIST;
            return list;
        case 1:
            return new ImmutableCollections.List12<>(elements[0]);
        case 2:
            return new ImmutableCollections.List12<>(elements[0], elements[1]);
        default:
            return ImmutableCollections.listFromArray(elements);
        }
    }
    ...
```

가변인수의 매개변수를 List로 수정하는 방법은 컴파일러가 메서드의 타입 안정성을 검증할 수 있는 장점을 가진다. 애노테이션을 명시하지 않아도 되며, 실수로 안전하다고 판단할 걱정도 없어지는 것이다. 하지만 클라이언트 코드가 살짝 지저분해지며 속도가 조금 느려질 수 있다는 점이 흠이다.

가변인수와 제네릭은 배열을 노출하고, 타입 규칙이 서로 달라 궁합이 좋지 않다. 제네릭 varargs 매개변수를 사용하고자 한다면 메서드가 타입 안전한지 확인하고, @SafeVarargs 애노테이션을 명시하여 불편함이 없도록 만들어보자!

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다. 읽어 주셔서 감사합니다.

## 참고자료 

- 이펙티브자바 3판
- https://docs.oracle.com/javase/tutorial/java/generics/nonReifiableVarargsType.html#heap_pollution - heap pollution

## 주석
 
[^1]: Heap pollution occurs when a variable of a parameterized type refers to an object that is not of that parameterized type. 
