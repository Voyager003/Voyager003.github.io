---
layout  : wiki
title   : 타입 안전 이종 컨테이너를 고려하라 
summary : 
date    : 2023-11-06 10:00:29 +0900
updated : 2023-11-06 13:26:16 +0900
tag     : java effectivejava
resource: B3/B4F520-B1BF-49E5-93FC-117005B49255
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 단일 원소 컨테이너

제네릭은 Set<E>과 같은 컬렉션이나 ThreadLocal<T>와 같은 단일 원소 컨테이너에도 흔히 사용된다. 

코드를 예시로 들어보자.

```java
// 제네릭을 사용한 단일 원소 컨테이너 클래스
public class Container<T> {
    private T element;

    public Container(T element) {
        this.element = element;
    }

    public T getElement() {
        return element;
    }

    public void setElement(T element) {
        this.element = element;
    }

    public static void main(String[] args) {
        // Integer 타입의 컨테이너 생성
        Container<Integer> intContainer = new Container<>(42);
        int intValue = intContainer.getElement();
        System.out.println("Integer value: " + intValue);

        // String 타입의 컨테이너 생성
        Container<String> strContainer = new Container<>("Hello, Generics!");
        String strValue = strContainer.getElement();
        System.out.println("String value: " + strValue);
    }
}

```

Container 클래스는 단일 원소 컨테이너로 제네릭을 사용해 어떤 타입의 원소라도 담을 수 있다. 이 때, 매개변수화 된 대상은 Container 클래스 자체로 Container<T>에서 T는 Container가 다루는 원소의 타입을 나타내는 것이다.

일반적으로 컨테이너의 일반적인 용도에 맞게 설계된 것이기 때문에 문제 될 것은 없지만, 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.

목적에 따라 더 유연한 수단이 필요할 때가 종종 있는데, 이를 위한 해법은 타입 안전 이종 컨테이너 패턴(type safe  heterogeneous container pattern)을 사용하는 것이다.

## 타입 안전 이종 컨테이너(type safe heterogeneous container)

타입 안전 이종 컨테이너의 특징은 다음과 같다.

- 컨테이너 대신 키를 타입 매개변수화 하고,
- 컨테이너에서 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다.
- 이는 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장한다.

특징을 따라 코드를 작성해보자.

```java
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> T getFavorite(Class<T> type)
}
```

item33은 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스를 예시로 든다. 

코드에서는 컨테이너 대신에 키를 매개변수화(Class<T> type)한 뒤, 컨테이너에서 값을 넣거나(put) 뺄 때 매개변수화한 키를 함께 제공한다. 이는 클래스가 제네릭(Class<T>)이기 때문에 클래스 리터럴 타입은 Class가 아닌 Class<T>가 된다. 

다시 말해, String.class의 타입이 Class<String>일 것이고, Integer.class는 Class<Integer>라는 것이다. 

이는 리터럴 타입을 Class<T>로 표현함으로써 컴파일러가 타입을 검증할 수 있어 런타임 시에 형변환 오류를 방지하고 안정성을 높일 수 있다.

Class<T>와 같이 컴파일타임의 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 클래스 리터럴을 타입 토큰(type token)이라 한다. 

```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();
    
    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    
    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```

타입 토큰을 사용한 Favorities은 위와 같이 완성된다.

이 때, Favorites가 사용하는 private 맵의 변수를 살펴보면 비한정적 와일드카드 타입이다. 

와일드카드는 중첩, 즉 모든 클래스 타입을 아우르며 서로 다른 구체적 타입들이 하나의 맵 안에 포함될 수 있다. 그렇다면

putFavorite() 메서드의 Class<T> type는 어떤 타입 T도 받아들일 수 있고, 모든 클래스 타입을 다룰 수 있다는 것을 의미하며,

getFavorite() 메서드에서는  Class<T> type를 받아들이지만, 이 메서드는 T 타입의 인스턴스를 반환하여 여기서 T는 putFavorite에서 받은 type와 동일한 것으로 간주된다.

또한 map의 값 타입은 단순 Object로 모든 값이 키로 명시한 타입임을 보증하지 않는다. 

putFavorite() 메서드는 주어진 클래스 객체와 인스턴스를 추가해 관계를 맺고 있으며, 여기서 키와 값사이의 타입 링크(type linkage) 정보는 버려져 그 값이 해당 키 타입의 인스턴스라는 정보가 사라진다.

getFavorite() 메서드에서 우선 주어진 Class객체에 해당 하는 값을 favorites 맵에서 꺼낸다. 이 객체가 반환해야할 타입 객체는 맞지만, 잘못된 컴파일타임 타입(Object)을 갖고 있어 T타입으로 변환해서 반환해줘야한다. Class의 cast() 메서드를 사용해 객체가 가리키는 타입으로 동적 형변환하여 가져오는 것을 볼 수 있다.

```java
@IntrinsicCandidate
public T cast(Object obj) {
    if (obj != null && !isInstance(obj))
        throw new ClassCastException(cannotCastMsg(obj));
    return (T) obj;
}
    ...
```

cast() 메서드에서는 주어진 인수가 클래스 객체가 알려주는 타입의 인스턴스인지 확인하고 맞다면 반환, 아니라면 ClassCastException을 던진다. 

클라이언트 코드가 깔끔히 컴파일된다면 getFavorite() 메서드가 favorites의 map 안의 값은 해당 키의 타입과 항상 일치하여 예외를 던지지 않을 것임을 알 수 있을 것이다. 

그럼에도 cast()를 하는 이유는 cast() 메서드의 시그니처가 제네릭인 점을 이용하여 Favorites를 T로 비검사 형변환하지 않고도 타입 안전하게 만들 수 있기 때문이다.

### 제약

- 악의적 클라이언트가 CLass 객체를 제네릭이 아닌 로 타입(raw type)으로 넘기는 경우

```java
Favorites favorites = new Favorites();
favorites.putFavorite(String.class, "Java");
```

putFavorite() 메서드에 로 타입을 사용해 호출을 한 상황이다. 

이 경우에 내부적으로는 Map에 Class 객체가 Class<?> 타입으로 저장되지만, 클라이언트 코드에서는 로(raw) 타입을 사용하여 정보를 은폐하게 된다. 

이로 인해 getFavorite를 사용할 때 제네릭 타입 정보를 복원할 수 없다. 

Favorites가 타입 불변식을 어기지 않도록 보장하려면 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 확인해야 한다. 이를 해결하기 위한 방법은 동적 형변환이다.

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

type.cast(instance) 부분에서 실제로 동적 형변환이 이루어져 컴파일러가 컴파일 시에 타입을 확인한다. instance가 type으로 지정된 클래스와 호환되지 않는다면 컴파일 오류가 발생하여 실행 시점에 형변환 오류가 발생할 가능성을 사전에 방지할 수 있어 타입 안정성을 확보할 수 있다.

- 실체화 불가 타입에는 사용 불가

String, String[]는 저장할 수 있어도, List<String> 은 저장할 수 없다. 

이유는 List<String>과 List<Integer> 둘다 List.class 라는 객체를 공유하기 때문에 List<String> 용 Class 객체를 알 수 없는 것이다.

이는 제네릭 타입소거에 기인한 문제로, 컴파일 타임에 타입 체크를 수행하고 실행 시에는 제네릭 타입 정보를 소거하여 일반적인 타입으로 변환하여 원시 타입(raw type)에 대한 정보만이 남게 되는 것이다,

즉 List<String>과 List<Integer>은 컴파일 시에는 다른 타입으로 간주되지만, 실행 시에는 둘 다 List로 취급된다. 따라서 컴파일러는 두 리스트에 대해 동일한 List.class를 생성하게 되는데 List.class로는 제네릭 타입 정보를 구별할 수 없다.

## 한정적 타입 토큰(pounded type token)

위에서 살펴본 Favorites 클래스의 타입 토큰은 비한정적이다. 

Map<Class<?>, Object> favorites에서 gerFavorite과 putFavorite는 어떤 클래스 객체든 받아들인다. 이 메서드들이 허용하는 타입을 제한하고 싶을 경우에는 한정적 타입 토큰을 활용하면 가능하다.

한정적 타입 토큰은 한정적 와일드카드를 사용해 표현 가능한 타입을 제한하는 타입 토큰이다. 예시로 Annotation API를 살펴보자.

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType)
```

annotationType의 인수로 애노테이션 타입을 뜻하는 한정적 타입 토큰이 사용되었다.

토큰으로 명시한 타입의 애노테이션이 대상 요소에 명시되어 있다면, 해당 애노테이션을 반환하고 없다면 null을 반환한다. 즉, 애노테이션된 요소는 키가 애노테이션 타입인 타입 안전 이종 컨테이너인 것이다.

Class<?> 타입의 객체를 한정적 타입 토큰을 받는 메서드에 넘기고 싶을 때 이를 동적으로 인스턴스 메서드를 제공한다.

asSubclass()는 호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환에 성공하면 인수로 받은 클래스 객체를 반환하고, 실패하면 ClassCastException을 던진다.

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null; //비한정적 타입 토큰
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }

    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```

이를 사용하면 컴파일 시점에는 타입을 알 수 없는 애노테이션을 asSubclass 메서드를 사용해 런타임에 읽어낼 수 있어 오류나 경고없이 컴파일할 수 있다.


## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다. 

읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판





