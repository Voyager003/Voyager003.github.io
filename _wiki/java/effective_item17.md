---
layout  : wiki
title   : 변경 가능성을 최소화하라 
summary : 
date    : 2023-08-13 23:17:31 +0900
updated : 2023-08-14 13:46:33 +0900
tag     : java effectivejava
resource: F6/CAF743-CEB8-4305-987D-33737879FA01
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 불변 클래스(Immutable Class)

불변 클래스는 해당 인스턴스의 내부 값을 수정할 수 없는 클래스를 말한다. 불변 클래스의 인스턴스는 고정되어 인스턴스가 파괴되는 순간까지 절대 달라지지 않는다. 

Java의 불변 클래스는 무엇이 있을까?

대표적으로 String이 있으며, *기본 타입(Primitive type)의 Boxing된 클래스들(Integer, Double..), BigInterger, BigDecimal이 이 불변 클래스에 속한다.

클래스를 불변으로 만들기위한 규칙은 다음과 같다. 

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다. (setter 키워드)

- 클래스를 확장할 수 없도록 한다. (extend 비허용)

- 모든 필드를 final로 선언한다. (final 키워드)

- 모든 필드를 private으로 선언한다. (private 키워드)

- 자신 외 내부의 가변 컴포넌트에 접근할 수 없도록 한다. 

## 장점

불변 객체를 사용함으로써 얻을 수 있는 이점은 다음과 같다.

- 근본적으로 Thread-safe하다.

여러 thread가 동시에 사용하더라도 절대 훼손되지 않아 클래스를 thread-safe하게 만드는 가장 쉬운 방법이다. 

- 안심하고 공유할 수 있다.

1.의 연장선이다. thread간 영향을 주고받을 수 없어, 방어적 복사도 필요없다.

- 인스턴스 재활용 가능

```java
public class BigInteger extends Number implements Comparable {

    final int signum;
    
    final int[] mag;
    
    ...
}

public Big Integer negate() {
    return new BigInteger(this.mag, =this.signum); 
}

BigInteger(int[] magnitude, int signum) {
    this.signum = (magnitude.length == 0 ? 0 : signum);
    this.mag = magnitude;
    if (mag.length>= MAX_MAG_LENGTH) {
        checkRange();
    }
}
```

자주 사용되는 인스턴스를 캐싱하여 정적 팩토리 메서드로 제공할 수 있다. 이는 인스턴스를 공유하기 때문에 메모리 사용량과 GC의 비용이 줄어든다.

BigInteger가 여기에 속하며, 음수를 구하는 negate()가 mag배열을 재사용하여 참조값을 넘기는 것을 확인할 수 있다. 

- 객체 생성 시, 다른 불변 객체들을 구성요소로 사용하면 이점이 생긴다.

불변 객체로 이뤄진 객체라면 구조가 아무리 복잡하더라도, 불변식을 유지하기 쉬워진다.

```java
public class ImmutableExample {
    public static void main(String[] args) {
    
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 85);

        String key = "Alice"; 
        int score = scores.get(key);
        System.out.println(key + "'s score: " + score);

        Set<LocalDate> holidays = new HashSet<>();
        holidays.add(LocalDate.of(2023, 1, 1));
        holidays.add(LocalDate.of(2023, 3, 8));

        LocalDate holiday = LocalDate.of(2023, 1, 1); 
        boolean isHoliday = holidays.contains(holiday);
        System.out.println(holiday + " is a holiday: " + isHoliday);
    }
}
```

예시의 String과 LocalDate는 모두 불변객체로, 각각 Map의 Key와 Set의 원소로 사용했다. 이는 안전하고 예측가능한 동작을 보장할 수 있다.

- 그 자체로 실패 원자성을 제공한다.

여기서 실패 원자성이란, 메서드에서 예외가 발생한 뒤에도 그 인스턴스는 메서드 호출 전의 상태와 같은 상태를 가지는 것을 말한다.

상태가 절대 변하지않기 때문에, 잠깐이라도 불일치 상태에 빠질 가능성이 없다.

```java
public class ImmutableFailureAtomicityExample {
    public static void main(String[] args) {
        ImmutableCounter counter = new ImmutableCounter(0);

        try {
            counter = counter.increment(); // 증가시키는 작업
            counter = counter.divideBy(0); // 0으로 나누는 작업 (실패 예시)
        } catch (ArithmeticException e) {
            System.out.println("An error occurred: " + e.getMessage());
        }

        System.out.println("Counter value: " + counter.getValue());
    }
}

final class ImmutableCounter {
    private final int value;

    public ImmutableCounter(int value) {
        this.value = value;
    }

    public ImmutableCounter increment() {
        return new ImmutableCounter(value + 1);
    }

    public ImmutableCounter divideBy(int divisor) {
        if (divisor == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return new ImmutableCounter(value / divisor);
    }

    public int getValue() {
        return value;
    }
}
```

예시의 ImmutableCounter는 불변객체로, 카운터 값을 증가시키고 나누는 메서드를 가진다. divideBy()에서는 0으로 나누는 경우, 예외를 던지도록 했는데 실행 결과에서는 divideBy()가 실패하여 예외가 발생하더라도 ImmutableCounter 객체의 상태는 변하지 않고 실패한 작업의 영향을 받지 않는다. 

## 단점

값이 다르면 반드시 독립된 인스턴스로 만들어야 한다. 값의 가짓수가 많다면 이를 생성하는데 큰 비용을 치러야 한다.

item17에서 말하는 예시를 보자.

```java
// BigInteger의 flipBit
public BigInteger flipBit(int n) {
        if (n < 0)
            throw new ArithmeticException("Negative bit address");

        int intNum = n >>> 5;
        int[] result = new int[Math.max(intLength(), intNum+2)];

        for (int i=0; i < result.length; i++)
            result[result.length-i-1] = getInt(i);

        result[result.length-intNum-1] ^= (1 << (n & 31));

        return valueOf(result);
    }
    
// BitSet의 flipSet
public void flip(int bitIndex) {
        if (bitIndex < 0)
            throw new IndexOutOfBoundsException("bitIndex < 0: " + bitIndex);

        int wordIndex = wordIndex(bitIndex);
        expandTo(wordIndex);

        words[wordIndex] ^= (1L << bitIndex);

        recalculateWordsInUse();
        checkInvariants();
    }
```

백만 비트짜리 BigInteger에서 비트 하나를 바꿔야 한다고 가정해보자.

flipBit()는 새로운 BigInteger 인스턴스를 생성하는데, 이 연산은 BigInteger의 크기에 비례해 시간과 공간을 잡아먹는다.(O(n)의 시간복잡도) 

반면 BitSet은 원하는 비트 하나만을 상수 시간안에 바꿔주는 메서드이다. (O(1)의 시간복잡도)

원하는 인스턴스를 완성하기까지의 단계가 많고, 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 발생한다. 이를 어떻게 대처할까?

### 다단계 연산이 예측될 경우

이러한 경우 기본 기능으로 제공하는 방법이 있다. 

```
├── java 
│   └── math
│       ├── BigDecimal
│       └── BigInteger
│       └── BigSieve
        ...
```

BigInteger는 모듈러 지수같은 다단계 연산 속도를 높여주는 가변 동반 클래스(companion class)를 package-private으로 두고 있다. 다단계 연산을 기본으로 제공한다면, 각 단계마다 인스턴스를 생성하지 않아도 될 것이다.

예측할 수 없는 경우에는 가변 동반 클래스를 public이로 제공하는게 최선이다. 여기에 대한 예시로 String 클래스는 연산속도를 높이기 위해 StringBuilder라는 가변 동반 클래스를 사용할 수 있다.

## 추가적 불변객체 설계법

- 상속을 막는다.

위에서 설명했지만, 불변임을 보장하려면 상속하지 못하게 해야한다는 규칙이 있었다. 상속을 못하게 하는 가장 쉬운 방법은 final로 선언하는 방법도 있지만, 더 유연한 방법이 있다. 

모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리 메서드를 제공하는 방법이다. 패키지 바깥의 클라이언트에서 바라본 불변 객체는 사실상 final로 public이나 protected 생성자가 없어 다른 패키지에서 이 클래스를 확장하는게 불가능하다.

- 불변 객체 기준 완화

초입에서 나열한 불변 클래스의 규칙 목록에 따르면 모든 필드가 final이며, setter와 같은 키워드를 사용할 수 없어 객체를 수정할 수 없어야 한다. 사실 이 규칙은 엄격한 감이 있어, 성능을 위해 살짝 완화하여 사용할 것을 제안한다. 

어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다. 어떤 불변 클래스는 계산 비용이 큰 값을 나중에 계산하여 final이 아닌 필드에 캐시해놓기도 한다. 이는 같은 값을 요청하면 캐시해둔 값을 반환하여 비용을 절감할 수 있다. 

불변 클래스는 장점이 많으며 단점이라곤 특정 상황에서의 잠재적 성능 저하 뿐이다. 모든 클래스를 불변으로 만들 수는 없기때문에 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이도록 하자.

## 참고자료 

- 이펙티브 자바 3판





