---
layout  : wiki
title   : 람다보다는 메서드 참조를 사용해라 
summary : 
date    : 2023-12-11 09:37:33 +0900
updated : 2023-12-11 11:36:28 +0900
tag     : java effectivejava
resource: 45/EA8E8C-1219-42EE-9D6B-DD6574AF39EE
toc     : true
public  : true
parent  : [[/java]] 
latex   : false
---
* TOC
{:toc}

## 메서드 참조(method reference)

람다 표현식의 장점 중 하나는 간결함이다. 하지만 Java는 메서드 참조라는 방식으로 더 간결하게 만드는 방법이 있다. 

```java
// Map.merge
default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) 
        ...
        
// 람다 표현식
map.merge(key, 1, (count, incr) -> count + incr);
```

Map의 merge 메서드는 키, 값, 매핑 함수를 인수로 받아서 주어진 키가 Map에 아직 없다면 주어진 쌍을 그대로 저장하고, 이미 있는 키라면 세 번째 인수로 받은 함수를 현재 값과 주어진 값에 적용한 뒤에 현재 값을 덮어쓰는 메서드이다.

람다 표현식으로 간결하게 바꿨봤지만 count와 incr이라는 변수는 하는 일이 없이 공간을 차지하고 있다. 

Java 8부터 Integer 클래스와 모든 기본 타입의 박싱 타입은 람다 표현식과 기능이 같은 정적 메서드 sum을 제공하기 시작했다. 그렇게 메서드의 참조를 전달하면

```java
map.merge(key, 1, Integer::sum);
```

같은 기능이지만 더 간결한 결과를 얻을 수 있다.

하지만 어떤 람다 표현식에서는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기도 한다. 이 경우는 간결함은 떨어지지만 메서드 참조보다 읽기 쉽고 유지보수가 더 쉬울 수 있다.

즉 람다 표현식으로 작성할 코드를 새로운 메서드에 담은 뒤, 람다 표현식 대신 그 메서드 참조를 사용하는 방식이다. 메서드 참조에는 기능을 드러내는 이름을 지어줄 수 있고 설명을 문서로 남길 수도 있다.

IDE가 람다 표현식을 메서드 참조로 대체하라고 권할 때는 권고를 따르는게 보통은 이득이지만 항상 그렇지만은 않다. 

```java
// 람다 표현식
service.execute(() -> action());

// 메서드 참조 방식
service.execute(GoshThisClassNameIsHumonous::action);
```

람다 표현식을 IDE의 권고대로 메서드 참조 방식으로 바꾼 경우인데, 메서드 참조를 보면 짧지도 않고 명확하지도 않다. 이 경우는 람다 표현식을 사용하는 편이 코드도 짧고 명확하다.

## 메서드 참조 유형

메서드 참조 유형은 다섯 가지가 있다.

- 정적 메서드 참조(Static method reference)

```java
// 람다 표현식
str -> Integer.parseInt(str)

// 메서드 참조
Integer::parseInt
```

위 예시에서 살펴봤던 유형으로 가장 일반적인 메서드 참조 유형이다.

- 한정적 인스턴스 메서드 참조(Bound instance method reference)

```java
// 람다 표현식
Instant then = instant.now()); t ->> then.isAfter(t);

// 메서드 참조
Instant.now()::isAfter
```

수신 객체(receiving object: 참조 대상 인스턴스)를 특정하는 한정적 인스턴스 메서드 참조이다.

함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다는 점에서 정적 참조와 비슷하다.

- 비한정적 인스턴스 메서드 참조(Unbound instance method reference)

```java
// 람다 표현식
str -> str.toLowerCase();

// 메서드 참조
String::toLowerCase
```

비한정적 참조는 주로 Stream 파이프라인에서의 매핑과 필터 함수에 사용된다.

- 클래스 생성자 참조(Constructor reference)

```java
import java.util.function.Function;

public class ConstructorReferenceExample {

    public static void main(String[] args) {
    
        // 람다 표현식
        Function<String, Integer> lambdaConstructor = s -> new Integer(s);

        // 클래스 생성자 참조
        Function<String, Integer> referenceConstructor = Integer::new;

        // 테스트
        Integer resultLambda = lambdaConstructor.apply("123");
        Integer resultReference = referenceConstructor.apply("456");

        System.out.println("Result from Lambda: " + resultLambda);
        System.out.println("Result from Constructor Reference: " + resultReference);
    }
}
```

Integer::new는 Integer 클래스의 생성자를 참조한다. 

이것은 함수형 인터페이스의 apply 메서드에 전달된 문자열을 입력으로 받아 Integer 객체를 생성하게 된다.

- 배열 생성자 참조(Array constructor reference) 

```java
import java.util.function.Function;
import java.util.Arrays;

public class ArrayConstructorReferenceExample {

    public static void main(String[] args) {
        // 람다 표현식
        Function<Integer, int[]> lambdaArrayConstructor = size -> new int[size];

        // 배열 생성자 참조
        Function<Integer, int[]> referenceArrayConstructor = int[]::new;

        // 테스트
        int[] resultLambda = lambdaArrayConstructor.apply(3);
        int[] resultReference = referenceArrayConstructor.apply(5);

        System.out.println("Result from Lambda: " + Arrays.toString(resultLambda));
        System.out.println("Result from Constructor Reference: " + Arrays.toString(resultReference));
    }
}
```

int[]::new는 정수 배열을 생성하는 생성자를 참조하고, 함수형 인터페이스의 apply 메서드에 전달된 크기를 입력으로 받아 정수 배열을 생성하게 된다.


이처럼 메서드 참조는 람다 표현식의 명료한 대안이 될 수 있다. 메서드 참조 쪽이 짧고 명확한 경우 이를 사용하고, 그게 아니라면 람다 표현식을 사용하자.

## 제네릭 함수 타입(generic function type) 구현

람다 표현식으로 불가능하지만 메서드 참조로 가능한 유일한 예이다.

함수형 인터페이스의 추상 메서드가 제네릭일 수 있듯이 함수 타입도 제네릭 일 수 있다.

```java
interface G1 {
    <E extends Exception> Object m() throws E;
}

interface G2 {
    <F extends Exception> Object m() throws Eception;
}

interface G extends G1, G2 {}

// 함수형 인터페이스 G를 함수 타입으로 표현 한 경우
<F extends Exception> () -> String throws F
```

함수형 인터페이스를 위한 제네릭 함수 타입은 메서드 참조 표현식으로 구현할 수 있지만 람다 표현식으로는 불가능하다. 

이유는 제네릭 람다식이라는 문법이 존재하지 않기 때문이다.

현재로서는 직접 제네릭 함수 타입을 표현하는 람다 표현식의 문법은 존재하지 않아 함수형 인터페이스를 통해 제네릭 함수 타입을 사용하려면 해당 함수형 인터페이스를 먼저 정의하고 그 인터페이스에 따라 람다 표현식을 작성해야 한다.

## 마무리 

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료 

- 이펙티브자바 3판
- https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html - method reference docs

