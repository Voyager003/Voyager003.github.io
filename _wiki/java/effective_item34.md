---
layout  : wiki
title   : int 상수 대신 열거 타입을 사용하라 
summary : 
date    : 2023-11-07 10:08:51 +0900
updated : 2023-11-07 13:50:25 +0900
tag     : java effectivejava
resource: DC/3647A5-6F17-43E7-9EB5-519D0B7AA87E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 열거(ENUM) 타입 등장 이전

[ENUM](https://voyager003.github.io/wiki/java/java_enum/)은 열거 형을 나타내는 특별한 데이터의 유형으로 상수의 집합으로 구성되며, 주로 연관된 상수 그룹을 정의하는 데 사용된다. 사계절, 카드 게임의 카드 종류가 좋은 예이다.

jdk 1.5부터 지원되는 열거 타입이 지원되기 전에는 정수 상수를 묶어 다음과 같이 선언했다.

```java
// 정수 열거 패턴
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;

// 문자열 열거 패턴
public static final String APPLE_FUJI = "apple fuji";
public static final String APPLE_PIPPIN = "apple pippin";
```

위처럼 선언한 방법은 정수 열거 패턴(int enum pattern), 문자열 열거 패턴(string enum pattern)이라고 한다.

이 방식은 ORANGE를 건네야 할 메서드에 사과를 보낸 뒤 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경고를 하지 않는다.

```java
APPLE_FUJI == ORANGE_NAVEL; // true
```

평범한 상수를 나열한 것 뿐이기 때문에 컴파일하게되면 그 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수의 값이 바뀌면 클라이언트도 다시 컴파일 해야하며, 컴파일 하지 않은 클라이언트는 실행은 되겠지만 예상과는 다르게 동작할 것이다.

문자열 열거 패턴도 마찬가지로 문자열 값을 그대로 하드코딩하게 만들기 때문에, 오타가 있어도 컴파일러는 확인할 길이 없고 자연스럽게 런타임 버그를 유발하게 된다. 

## 정수, 문자열 열거 패턴의 대안 열거 타입

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

JDK 1.5에 도입된 열거 타입은 자체로 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.

따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없어 열거 타입으로 선언된 인스턴스들은 딱 하나, 즉 싱글톤(singleton)임을 보장한다.([item3](https://voyager003.github.io/wiki/java/effective_item3/))

### 장점 

- 컴파일 타입 안정성을 제공

```java
// Enum의 컴파일 타임 안전성
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}

// 잘못된 상수 사용 (컴파일 오류)
Day invalidDay = Day.INVALID;  // 컴파일 오류
```

위와 같이 enum을 선언했다면, 건네 받는 참조는 Day의 값 중 하나임이 확실해진다. 따라서 다른 타입의 값을 넘기려고 한다면 컴파일 오류가 발생한다. 타입이 다른 열거 타입 변수에 할당하거나 열거 타입의 값끼리 동등 연산자로 비교하는 것이기 때문이다.

- 이름이 같은 상수가 공존

```java
public enum Apple {
    PIE, JAM;
}

public enum Strawberry {
    PIE, JAM;
}

public static void main(String[] args) {
    System.out.println(Apple.PIE);
    System.out.println(Strawberry.PIE);
}
```

각자의 이름 공간이 있어 이름이 공존 가능하다.

- 임의의 메서드나 필드를 추가할 수 있고, 임의의 인터페이스를 구현하도록할 수 있다.


```java
public abstract class Enum<E extends Enum<E>>
        implements Constable, Comparable<E>, Serializable {
        
    /**
     * The name of this enum constant, as declared in the enum declaration.
     * Most programmers should use the {@link #toString} method rather than
     * accessing this field.
     */
     
    private final String name;
    ...
}
```

Comparable과 Serializable을 구현하여 직렬화 형태도 변형을 가해도 문제없이 동작하게끔 구현되어 있다. 

## 열거 타입에 필드와 메서드 추가하기

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    ...
```
 
열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다. 근본적으로 불변이기 때문에 필드는 final이어야 하며, 필드의 경우 private으로 두고 별도의 public 접근자 메서드를 두는것이 낫다. ([item16 캡슐화](https://voyager003.github.io/wiki/java/effective_item16/)) 

## 상수별 메서드 구현(constant-specific method implementation)을 활용한 열거 타입

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVIDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

사칙연산 기능을 하는 열거 타입을 선언했다. apply() 메서드와 switch문으로 연산을 시작하는데, 동작하지만 좋은 방법은 아니다. 새로운 상수 추가 시에, case문이 추가되며, 깜빡할 경우에 컴파일은 되지만 AssertionERror 오류가 발생한다.

이를 실제 연산까지 열거 타입 타입 상수가 직접 수행하도록 바꿔보자.

```java
public enum Operation {
    PLUS {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    public abstract double apply(double x, double y);    
}
```

위와 같이 열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체(constant-specific class body)에 각 상수에서 자신에 맞게 재정의하는 방법이다.

이를 상수별 메서드 구현이라 한다.

apply 메서드가 상수 선언 옆에 명시되어 새로운 상수를 추가할 때 apply를 재정의해야 한다는 사실을 잊기 어려울 것이다. 또한 apply()가 추상 메서드이기 때문에 재정의하지 않으면 컴파일 오류로 알려준다. 

### valueOf() 

열거 타입이 제공하는 또 다른 메서드들이 있다.

상수의 이름을 입력받아 그 이름에 해당하는 상수를 반환하는 valueOf() 메서드를 제공한다.

```java
Operation i = Operation.valueOf("PLUS");
System.out.println(i); // PLUS
```

### fromString

열거 타입의 toString 메서드를 재정의했다면, toString 이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드를 제공하는 것을 고려할 수 있다.

```java
import java.util.Map;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public enum Operation {
    PLUS("+")    {public double apply(double x, double y){return x + y;}},
    MINUS("-")   {public double apply(double x, double y){return x - y;}},
    TIMES("*")   {public double apply(double x, double y){return x * y;}},
    DIVIDE("/")  {public double apply(double x, double y){return x / y;}};

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
    public abstract double apply(double x, double y);

    // 열거 타입 상수 생성 후 정적 필드가 초기화될 때 추가됨.
    private static final Map<String, Operation> stringToEnum = Stream.of(values()).collect(Collectors.toMap(Object::toString, e->e));

    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }
}
```

이 때, 열거 타입의 정적 필드 중 생성자에 접근할 수 있는 것은 상수 변수뿐이다. 따라서 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 초기화되기 전이기 때문에 자기 자신(this)을 추가하지 못하도록 제약이 필요하다.

## 전략 열거 타입 패턴

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다. 


```java
enum PayrollDay {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        int basePay = minutesWorked * payRate;

        int overtimePay;
        switch (this) {
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2; // 주말
                break;
            default: // 주중
                overtimePay = minutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }

} 
```

시간 당 기본 임금과 그 날에 일한 시간이 주어지면 일당을 계산하는 메서드를 예를 들었다.

간결하지만, 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case문이 필요하고 깜빡한다면, 컴파일은되지만 의도하지 않은 임금을 받게 될 것이다.

이를 해결하는 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다. 

```
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입을 추가
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
    }
}
```

잔업수당 계산(PayType)을 중첩 열거 타입으로 옮겨 PayrollDay 열거 타입의 생성자에서 선택하도록 코드를 수정했다. 그렇다면 PayrollDay 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여 switch문이나 상수별 메서드 구현을 할 필요가 없다.

하지만 기존 열거 타입에 상수별 동작을 혼합하는 경우에는 다음과 같이 switch문이 더 좋은 선택이 될 수 있다. 

```java
public static Operation inverse(Operation op) {
    switch (op) {
        case PLUS:      return Operation.MINUS;
        case MINUS:     return Operation.PLUS;
        case TIMES:     return Operation.DIVIDE;
        case DIVIDE:    return Operation.TIMES;
        default: throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면, 직접 만든 열거 타입이라도 이 방식을 적용하는 것을 고려해보자. 

## 열거 타입의 성능과 고려해야하는 상황

열거 타입의 성능은 정수 상수와 별반 다르지 않으며, 열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들지만 체감될 정도는 아니다. 

그렇다면 사용 시에 어떤 점을 고려해야 하는가?

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합인 경우

위 예시의 태양계 행성과, 한 주의 요일 등이 이에 해당한다.

- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다는 것

열거 타입은 상수가 추가되어도 바이너리 수준에서 호환되도록 설계되었다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판

