---
layout  : wiki
title   : 비트 필드 대신 EnumSet을 사용하라 
summary : 
date    : 2023-11-16 09:34:07 +0900
updated : 2023-11-16 12:00:39 +0900
tag     : java effectivejava
resource: 7A/074CAD-FFB8-466D-993A-EB2B62510E7C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 비트 필드(bit field)

```java
public class Vehicle {
    
    private static final int ENGINE_ON = 0b00000001;  // 1
    private static final int LIGHTS_ON = 0b00000010;  // 2
    private static final int AC_ON = 0b00000100;      // 4
    private static final int RADIO_ON = 0b00001000;   // 8
    
    private int status;

    // 엔진 상태 설정
    public void setEngineOn(boolean on) {
        if (on) {
            status |= ENGINE_ON;
        } else {
            status &= ~ENGINE_ON;
        }
    }
    ...
```

위와 같은 식으로 비트별 OR을 사용해 여러 상수를 모아 만든 집합을 비트 필드라 한다. 

비트별 연산을 사용해 집합 연산을 수행할 수 있지만 정수 열거 상수가 가지는 단점을 그대로 가져왔으며, 추가적인 문제점이 있다.

### 문제점

- 비트 필드 값 출력 시, 단순 정수 열거 상수를 출력할 때보다 해석하기 힘들다.

```java
public void printStatus() {
    System.out.println("Status: " + status);
}
```

위에서 정의한 차량의 상태를 출력하는 메서드이다. 출력된 이진수 값 자체는 해석이 쉽지만 각 비트가 어떤 상태를 나나내는지는 알아보기 힘들다.

- 비트 필드 하나에 녹아있는 모든 원소 순회것이 까다롭다.

```java
public void iterateStatus() {
        for (int i = 0; i < Integer.SIZE; i++) {
            int mask = 1 << i;
            if ((status & mask) != 0) {
                System.out.println("Bit " + i + " is set");
            }
        }
    }
    ...
```

비트 필드에 있는 모든 원소를 순회하기 위해서는 비트 연산을 사용해야 한다. 이는 이해하기 어렵고 새로운 원소를 추가하거나 제거해야하는 경우에 코드 수정이 복잡해진다.

- 최대 몇 비트가 필요한지 API 작성 시 미리 예측이 필요하다.

비트 필드의 크기를 넘어가는 상태를 표현하려면 비트 필드를 수정해야하므로, API 설계 시 이런 예측이 필요하다.

item36은 이런 문제점에 대한 대안으로 EnumSet을 제안한다.

## EnumSet

```java
public class Vehicle {

    public enum Status {
        ENGINE_ON, LIGHTS_ON, AC_ON, RADIO_ON
    }

    private EnumSet<Status> statusSet = EnumSet.noneOf(Status.class);
```

util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현한다. Set 인터페이스를 완벽히 구현하여 type-safe하고 다른 Set 구현체와 함께 사용할 수 있다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}
```

EnumSet은 elementType과 universe 두 가지 속성을 가지고 있다.
 
 elementType은 EnumSet에 속하는 열거 상수들의 클래스를 나타내고, universe는 모든 열거 상수를 포함하는 배열이다. 
 
noneOf() 메서드에서 지정된 elementType에 대한 빈 EnumSet을 생성하고, getUniverse 메서드를 통해 elementType에 해당하는 열거 상수 배열인 universe를 가져오게 된다.

이 때, universe.length가 64 이하인 경우 RegularEnumSet을, 그 이상인 경우 JumboEnumSet을 생성하는데, 이는 비트 벡터의 크기에 따라 다른 구현체를 선택하는 것이다.

```java
class JumboEnumSet<E extends Enum<E>> extends EnumSet<E> {

    private long elements[];
    
    private int size = 0;

    JumboEnumSet(Class<E> elementType, Enum<?>[] universe) {
        super(elementType, universe);
        elements = new long[(universe.length + 63) >>> 6];
    }
    ...
```

JumboEnumSet의 API를 살펴보면 비트 벡터를 저장하기 위한 elements 배열을 가지고 있으며, 각 비트는 열거 상수의 존재 여부를 나타낸다. 

이 때, 비트 벡터의 크기는 universe.length에 따라 동적으로 할당되는 것이다. 

다시 본론으로 돌아와서 비트 필드를 대체하는 EnumSet의 사용 예시를 보자.

```java
public class NewText {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public void applyStyles(Set<Style> styles) { ... }    
    
    ...
    
    text.applyStyles(EnumSet.of(Style.BOLD, Style.UNDERLINE));
}
```

applyStyles 메서드에 EnumSet 인스턴스를 넘기는 클라이언트 코드로, EnumSet이 제공하는 집합을 생성하는 정적 팩터리 메서드 of()를 사용했다.

컴파일 시점에 요소의 타입을 정확하게 추론할 수 있고, 짧고 깔끔하게 사용할 수 있다.

### 단점

EnumSet의 유일한 단점은 불변 EnumSet을 만들 수 없다는 것이다. 

찾아본 바로는 구글의 Guava 라이브러리가 불변 EnumSet을 제공하지만 내부에서 EnumSet을 사용해 구현하고 있기는 한데 성능이 떨어지며, Java 21까지 릴리즈된 내용을 봐도 딱히 변경된 내용이 없다는 것을 확인했다. 

명확성과 성능이 조금 떨어지지만 Collections.unmodifiableSet으로 Enum을 감싸 사용하는 것도 하나의 방법이다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료 

- 이펙티브자바 3판
- https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html - EnumSet
- https://stackoverflow.com/questions/28413670/immutable-version-of-enumset

