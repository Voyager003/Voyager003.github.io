---
layout  : wiki
title   : 한정적 와일드카드를 사용해 API 유연성을 높이라 
summary : 
date    : 2023-10-23 11:41:21 +0900
updated : 2023-10-23 14:31:52 +0900
tag     : java effectivejava
resource: B7/489E92-4292-4344-A354-540C3F947AD8
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 매개변수화 타입

 [item28](https://voyager003.github.io/wiki/java/effective_item28/#%EB%B0%B0%EC%97%B4%EA%B3%BC-%EC%A0%9C%EB%84%A4%EB%A6%AD-%ED%83%80%EC%9E%85%EC%9D%98-%EC%B0%A8%EC%9D%B4) 에서 배열과 제네릭의 차이에서 말했듯이, 매개변수화 타입은 불공변(invariant)이다. 

예를 들자면 List<String>은 List<Object>의 하위 타입이 아니다. 이는 List<Object>에는 어떤 객체든 넣을 수 있지만, List<String>의 경우 문자열만 넣을 수 있다. List<String>은 List<Object>가 하는 일을 제대로 수행하지 못하여 하위 타입이 될 수 없다는 것이다.(객체 지향 SOLID 원칙 중, 리스코프 치환 원칙에 어긋난다.)

다른 예시로 Stack 클래스를 떠올려보자.

```java
// Stack.class
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    ...
}
```

이 클래스에 일련의 원소를 Stack에 넣는 메서드를 추가해야 한다고 해보자. 

```java
// 31_1
public void pushAll(Iterable<E> src) {
  for (E e : src) {
    push(e);
  }
}
```

이 메서드는 Iterable src의 원소타입이 Stack의 원소 타입과 일치한다면 제대로 작동한다. 이 때, Stack<Number>로 선언한 뒤에 pushAll(intVal)을 호출한다면? (intVal은 Integer 타입)

Integer는 Nubmer의 하위 타입이기 때문에 잘 동작할 것이라고 예상했지만 다음 오류를 볼 수 있다.

```
incompatible types: Iterable<Integer> cannot be converted to Iterable<Number> 
```

이는 Stack<Number>가 Stack<Integer>의 하위 타입이 아니기 때문이다. 즉 매개 변수화 타입이 불공변이라는 것이다.

Java는 이를 대처하기위해 한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다.

## 한정적 와일드 카드(Bounded Wildcards)

먼저 와일드 카드에 대해 리마인드 해보자.

[와일드카드](https://voyager003.github.io/wiki/java/java_generic/#wildcard%EC%99%80%EC%9D%BC%EB%93%9C%EC%B9%B4%EB%93%9C) 는 제네릭으로 구현된 메서드의 경우 선언된 타입으로만 매개변수를 입력해야 하는데, 이를 상속받은 클래스 혹은 부모 클래스를 사용하고 싶어도 불가능하며, 어떤 타입이 와도 상관없는 경우에는 대응하기에 좋지 않다.

이를 위한 해결책인 **와일드 카드**는 알 수 없는 유형을 나타내며 패러미터 변수, 필드 등 다양한 상황에서 사용할 수 있다. 

그 중, 한정적 와일드 카드는 타입의 제한을 풀거나 제한할 때 사용하는 데,

```java
public void a(List<? extends Foo list) { ... }

public void b(List<? super Integer> list) { ... }
```
extends를 사용하여 제네릭 타입들을 상위 제네릭 타입으로 묶어주거나, super를 사용하여 지정된 타입의 상위 타입만 허용하도록 지정하는 방법을 사용한다. 
이를 각각 상한경계(Upper Bound), 하한경계(Lower Bounded) 와일드카드라 한다.

한정적 와일드카드가 가져다주는 효과로(제한 해제 및 제한) 31_1의 pushAll을 다시 수정해보자.

```java
// 상한 경계 와일드카드
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

이제 Stack은 물론 클라이언트 코드로 깔끔하게 컴파일된다.

pushAll과 같이 쓰이는 Stack 안의 모든 원소를 컬렉션으로 옮겨 담는 popAll 메서드를 작성해보자.

```java
public void popAll(Collection<E> dst) {
    while (!isEmpty()) {
    dst.add(pop()); // compile error 
    }
}
```

popAll의 경우에도 주어진 컬렉션의 타입이 Stack의 원소 타입과 일치한다면 문제없이 동작할 것이다. 하지만 Stack<Number>의 원소를 Object용 컬렉션으로 옮기려고 한다면 어떻게 될까?

```java
public static void main(String[] args) {
    Stack<Number> stack = new Stack<>();
    Collection<Object> objects = ...;
    stack.popAll(objects); // compile error
}
```

pushAll과 비슷하게 Collection<Obejct>는 Collection<Number>의 하위 타입이 아니라는 오류가 발생하게 된다.이를 하한경계 와일드 카드를 사용해서 개선해보자.

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }
}
```

popAll의 입력 매개변수(dst)의 타입이 E의 컬렉션이 아니라 E의 상위 타입인 컬렉션이어야 하는 의미로 <? super E>라고 상한 와일드 카드를 선언했다. 이제 깔끔하게 컴파일되며, 이처럼 제네릭을 사용해 유연한 API를 설계할 수 있다.

## PECS(producer-extends, consumer-super) 공식

이러한 한정적 와일드 카드를 제대로 사용하려면 어느 상황에 어떤 와일드 카드타입을 써야할 지 이해할 필요가 있는데 다음 공식이 도움을 줄 것이다.

공식의 이름과 같이 매개변수화 타입 T가 생산자(producer)라면 <? extneds T>를, 소비자(consumer)라면 <? super T>를 사용하면 된다. 

위에서 사용한 예시 Stack의 코드와 함께 이해해보자.

```java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src) {
    push(e);
  }
}

public void popAll(Collection<? super E> dst) {
  while (!isEmpty()) {
    dst.add(pop());
  }
}
```

Stack의 pushAll의 매개변수 src는 Stack이 사용할 E 인스턴스를 생산하는 역할이기 때문에 Iterable<? extends E>이며, popALl의 매개변수 dst는 E 인스턴스를 소비하는 역할로 적절한 타입은 Collcetion<? super E>이 되는 것이다.

## 타입 매개변수와 와일드 카드를 사용하는 경우

타입 매개변수와 와일드카드는 공통되는 부분이 많다. 

모두 제네릭 기능의 일부로 타입 안정성을 제공하고, 제네릭 메서드나 클래스에서 받아들일 수 있는 인자의 범위를 조정하는 데 사용된다는 점이다.

그래서 메서드 정의 시, 둘 중 어느 것을 사용해도 괜찮은 상황이 많다. 예를 들어 주어진 List에서 명시한 두 인덱스의 아이템들을 swap하는 정적 메서드를 두 방법으로 정의해보자.

```java
public static <E> void swap(List<E> list, int i, int j);

public static void swap(List<?> list, int i, int j);
```

public API라면 어떤 리스트건 swap 메서드에 넘기면 명시한 인덱스의 원소를 교환해 주는 와일드카드 방식을 사용한 2번째 방식이 간단하고 더 나은 방법이다. 

즉 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하는 것이 기본 규칙이다.

하지만 2번째 방식은 다음과 같은 문제가 있다.

```java
public static void swap(List<?> list, int i, int j){
    list.set(i, list.set(j, list.get(i)));
};
```

이 코드를 컴파일하면 

```
'Object cannot be converted to CAP#1 ...
where CAP#1 is a fresh type-variable: 
    CAP#1 extends Object from capture of ?\
```

방금 꺼낸 원소를 List에 다시 넣을 수 없다는 것이다. 원인은 List의 타입이 List<?>인데 이는 null 외에 어떠한 값도 넣을 수 없기 때문이다.

이를 해결하기위해 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 도우미 메서드(helper method)로 따로 작성하여 활용하는 방법이 있다.

```java
public static void swap(List<?> list, int i, int j){
    swapHelper(list, i, j);
};

public static <E> void swapHelper(List<E> list, int i, int j){
  list.set(i, list.set(j, list.get(i)));
};
```

swaoHelper() 메서드는 List가 List<E>임을 알고 있으므로 List에서 꺼낸 값의 타입은 항상 E고, E 타입의 값이라면 이 List에 넣더라도 type-safe하다는 것을 컴파일러가 알 수 있다. 

이처럼 외부에서 swap() 메서드를 호출하는 클라이언트는 swapHelper의 존재를 모른 채 와일드카드 기반의 선언을 유지하면서 이 메리트를 누릴 수 있게 된다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다. 읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판
- https://voyager003.github.io/wiki/java/java_generic/ - 이전에 작성했던 Java Generic 파트
