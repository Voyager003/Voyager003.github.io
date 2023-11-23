---
layout  : wiki
title   : 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라 
summary : 
date    : 2023-11-23 09:46:58 +0900
updated : 2023-11-23 11:10:21 +0900
tag     : java effectivejava
resource: 65/6A90B0-4ED8-4AD6-9B85-C294DDD37D8C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 열거 타입의 확장

```java
public class Day {
    public static final Day MONDAY = new Day("Monday");
    public static final Day TUESDAY = new Day("Tuesday");
    ...

    private final String name;

    private Day(String name) {
        this.name = name;
    }
    
    ...
```

위 예시는 일주일을 나타낸 타입 안전 열거 패턴(typesafe enum pattern)이다.

열거형 등장 이전에는 내부 상수를 public static final로 정의하여 사용했는데, Java 5에 도입된 열거형의 도입으로 이러한 패턴을 대체하고 보다 간결하고 tpye-safe한 상수를 정의할 수 있게 되었다.

열거형으로 수정한 코드는 다음과 같다.

```java
public enum Day {
    MONDAY("Monday"),
    TUESDAY("Tuesday"),
    WEDNESDAY("Wednesday"),
    // ...

    private final String name;

    private Day(String name) {
        this.name = name;
    }
    ...
```

모든 상황에서 타입 아넌 열거패턴보다 간결성함과 타입 안정성면에서 뛰어나지만, 문제가 있다. 

타입 안전 열거 패턴에서는 열거한 값들을 그대로 가져와 다른 값을 더 추가하여 다른 목적으로 확장할 수 있는 반면에 열거 타입은 확장이 불가능하다는 것이다.

여기서 짚고가야할 점은 대부분 상황에서 열거 타입을 확장하는 것은 좋지 않다는 것이다. 이유는 다음과 같은데,

- 확장 타입의 원소는 기반 타입의 원소로 취급하지만, 반대는 성립하지 않는다.

- 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법이 마땅하지 않다.

```java
public class EnumerationTraversalExample {
    public enum BaseType {
        VALUE1,
        VALUE2,
        VALUE3;
    }

    public enum ExtendedType {
        EXTENDED_VALUE4,
        EXTENDED_VALUE5;
    }

    public static void main(String[] args) {
        System.out.println("BaseType Elements:");
        for (BaseType baseType : BaseType.values()) {
            System.out.println(baseType);
        }

        System.out.println("\nExtendedType Elements:");
        for (ExtendedType extendedType : ExtendedType.values()) {
            System.out.println(extendedType);
        }
    }
}
```

위 코드와 같이 독립된 열거형을 따로 순회하는 것이 불가능하며, 함께 순회하려면 추가적인 로직이 필요하다.

- 확장성을 높이기 위해 고려해야할 부분이 많다.

열거 타입이 독립적으로 존재한다는 것이 장점일 수 있지만, 통합이 필요한 상황에는 고려할 점이 많다는 것이다.

item38에서는 확장할 수 있는 열거 타입이 어울리는 쓰임 중 하나인 연산 코드를 예시로 들어 설명한다.

```java
// Operation interface
public interface Operation {
    double apply(double x, double y);
}

// BasicOperation enum
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

연산 코드용 인터페이스를 정의하고 열거 타입이 인터페이스의 표준 구현체 역할을 하도록 만든 모습이다.

열거형인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장이 가능하여, 이 인터페이스를 연산의 타입으로 사용한다. 

여기에 연산 타입을 확장하여 지수 연산(EXP)과 나머지 연산(REMAINDER)을 추가한다고 하면

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```

코드에 Operation 인터페이스를 구현한 열거 타입을 작성하면 된다.

이렇게 추가된 연산은 기존 연산을 쓰던 곳이면 어디든 사용할 수 있으며, 기본 열거 타입 대신에 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용할 수 있다.

```java
@Test
void testEx() {
    double x = 1.0;
    double y = 2.0;
    test(ExtendedOperation.class,x, y);
}

private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                        x, op, y, op.apply(x, y));
}
```

test 메서드에 ExtendedOperation의 class 리터럴을 넘겼다. 

<T extends Enum<T> & Operation>는 Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 의미로, 열거 타입이어야 원소를 순회 가능하며, Operation이어야 원소가 뜻하는 연산을 수행할 수 있기 때문이다.

두 번째 대안은 Class 객체 대신 한정적 와일드 카드를 넘기는 방법이다.

```java
private static void test(Collection<? extends Operation> opSet,
        double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
}
```

여러 구현 타입의 연산을 조합하여 호출할 수 있게 되었지만, 이는 특정 연산에서는 EnumSet이나 EnumMap을 사용하지 못하는 방법이다.

### 문제점 

인터페이스를 사용해 확장 가능한 열거 타입을 흉내내는 방식에는 사소한 문제가 몇가지 있다.

바로 열거 타입끼리 구현을 상속할 수 없다는 점이다. 

아무 상태에도 의존하지 않은 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다. 

하지만 예제의 Operation은 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야하는데, 공유하는 기능이 많다면 별도의 도우미 클래스와 정적 도우미 메서드로 분리하는 방식으로 코드의 중복을 없애는 방법이 있다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.



## 참고자료 

- 이펙티브자바 3판











