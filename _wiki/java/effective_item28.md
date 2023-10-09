---
layout  : wiki
title   : 배열보다는 리스트를 사용하라 
summary : 
date    : 2023-10-09 09:49:25 +0900
updated : 2023-10-09 12:08:27 +0900
tag     : java effectivejava
resource: 7A/A3FF5E-159E-4E40-9E4A-B98EAC5942D9
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 배열과 제네릭 타입의 차이

배열과 제네릭 타입에는 중요한 차이가 있다.

- 배열은 공변(covariant)이지만, 제네릭은 불공변(invariant)이다.

공변은 하위 타입 관계를 유지하는 것을 말한다. 코드로 살펴보자.

```java
// 28-1
Object[] objectArray = new Long[1];
objectAraayp[0] = "I don't fit in"; // ArrayStoreException 
```

배열의 경우 공변이기 때문에, 자식 클래스(sub-class)가 부모 클래스(super-class)의 하위 타입이라면, 자식 클래스의 배열은 부모 클래스의 하위 타입이 된다. 

반면 제네릭 타입을 살펴보면

```java
// 28-2
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```

제네릭 타입의 경우 불공변으로 서로 다른 타입 Type1,Type2가 있을 때 List<Type1> 은 List<Type2> 의 하위타입도 아니고 상위타입도 아니다. 쉽게 말해 두 개의 타입은 전혀 관련이 없다는 것이다.

이런 차이로 발생하는 문제는 배열에서는 예외 발생 시점을 런타임에 알게되지만, 제네릭 타입의 경우 타입 오류를 컴파일 시점에 알리기 때문에 오류를 바로 확인할 수 있다. 가장 좋은 에러는 컴파일 에러임을 잊지말자.

- 배열은 실체화(reify)되지만, 제네릭 타입은 비구체화(non-reify)된다.

배열의 경우 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 이를 구체화 타입이라 한다. 

반면 제네릭 타입의 경우 런타임에 타입 정보가 소거(erasure)된다. 즉 런타임에는 원소 타입을 알 수 조차없다는 것이다.

## 제네릭 배열 생성을 허용하지 않는 이유

결론부터 말하면 type-safe하지 않기 때문이다.

이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있는데, 이는 제네릭 타입 시스템의 취지에 어긋난다.(잘못된 타입이 들어올 수 있는 것을 컴파일 타임에 방지하는 것을 포기하는 것이다.)

E, List<E>, List<String>와 같은 타입을 실체화 불가 타입(non-reifiable type)이라 한다. 실체화되지 않아 런타임에는 컴파일타임보다 타입 정보를 적게 가진다. 소거 매커니즘으로 인해 매개변수화 타입 가운데 실체화될 수 있는 타입은 비한정적 와일드카드(item26) 뿐이다.

## 배열의 불편한 점

- 제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는 것이 보통은 불가능하다.

```java
public class GenericClass<T> {
    private List<T> elements;

    public T[] toArray() {
        return elements.toArray(new T[0]); // 컴파일 에러
    }
}
```

new T[0]과 같은 코드에서 컴파일러는 T가 어떤 클래스인지 알 수 없기 때문에 컴파일 에러를 발생시킨다. 

- 제네릭 타입과 가변인수(Varargs, Variable arguments) 메서드를 함께 쓰면 해석하기 어려운 경고 메세지를 받게된다.

가변 인수는 패러미터로 들어오는 값의 갯수와 상관없이 동적으로 인수를 받아 기능하도록 해주는 문법을 말한다.

```java
public class VarargsAndGenerics {
    public static <T> void print(T... args) {
        for (T t : args) {
            System.out.println(t);
        }
    }

    public static void main(String[] args) {
        List<String> strings = Arrays.asList("Java", "C++", "Python");
        print(strings); // Unchecked generic array creation for varargs parameter
    }
}
```

예시를 살펴보면 print 메서드는 제네릭 타입 T의 가변 인수(T.. args)를 받고있다. main 메서드에서 이를 호출할 때 List<String>을 인자로 전달하는데, 컴파일러는 print 메서드에게 전달되는 실제 인자들이 배열로 만들어지므로, 제네릭 배열 생성에 대한 경고를 발생시킨다.

이는 가변인수 메서드를 호출할 때, 가변 인수를 담는 배열이 만들어지는데 이 때 그 배열의 원소가 실체화 불가 타입이어서 경고가 뜬 것이다. 이처럼 해석하기 힘든 경고 메세지를 받게 된다.

## 배열대신 리스트

여기까지 배열과 제네릭의 차이와 배열의 불편한 점을 알아봤다. 

그렇다면 item28의 주제인 리스트를 사용하여 얻을 수 있는 이점은 무엇이 있을까?

```java
// 배열의 경우
public class Chooser<T>{
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    // 수정 전 : choicesArra = choices.toArray; 
    choicesArra = (T[]) choices.toArray; 
  }

    public Object choose(){
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

위에서 살펴봤지만 제네릭에서는 원소의 타입정보가 소거되기 때문에 런타임에는 타입 정보를 알 수 없다. 

결과적으로 수정 전의 코드를 살펴보면 T[]에서 형변환 과정에서 ' incompatible types: Object[] cannot be converted to T[]'라는 오류 메시지를 볼 수 있다. 

이를 해결하기 위해 Object 배열을 'T[]'로 형변환 하더라도 unchecked cast 경고를 볼 수 있다. item27에서 봤던 비검사 경고인 것이다!

```
// 리스트의 경우
public class Chooser<T>{
  private final List<T> choiceList;

  public Chooser(Collection<T> choices){
    choiceList = new ArrayList<>(choices);
  }

  public T choose(){
    Random rnd = ThreadLocalRandom.current();
    return choiceList.get(rnd.netInt(choiceList.size()));
  }
}
```

item27에 나왔던 @SuppressWarnings 애노테이션을 추가할 수도 있지만, 리스트를 사용하여 형변환 경고를 제거할 수 있다.

코드가 조금 복잡해지고 성능이 약간 떨어질 수는 있지만 type-safe하며, 비검사 경고를 제거할 수 있는 방법이다.

## 마무리

보면서 이상하다 싶은 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다. 읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/8/docs/technotes/guides/language/varargs.html - 가변인수



