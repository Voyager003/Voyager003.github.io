---
layout  : wiki
title   : Comparable을 구현할지 고려하라 
summary : 
date    : 2023-07-24 10:02:07 +0900
updated : 2023-07-24 13:50:35 +0900
tag     : java effectivejava
resource: EA/DB200C-28A1-4B39-96B1-2DD955414E7D
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Comparable, compareTo

> 이 인터페이스는 이를 구현하는 각 클래스의 객체에 전체 순서를 부과한다.

```java
public interface Comparable<T> {
    /**
     * 이객체와 주어진 객체의 순서를 비교한다.
     * 객체가 주어진 객체보다 작으면 음의 정수를
     * 같으면 0을
     * 크면 양의 정수를 반환한다.
     * 비교할 수 없는 타입의 객체가 주어지면 ClassCastException가 발생한다.
     */
     
    public int compareTo(T o);
}
```

Comparable 인터페이스의 compareTo는 단순 동치성 비교와 순서 비교를 할 수 있으며 Generic하다. 

여기서 Generic하다라는 것은 Comparable 인터페이스에 사용된 타입 매개변수 'T'가 실제 사용될 때 구체적 타입으로 대체되어 해당 타입에 맞게 동작하도록 만들어지는 것을 의미한다.

Comparable을 구현했다는 것은 그 클래스의 인스턴스들에게 자연적 순서(natural order)가 있음을 뜻하며, Comparable을 구현한 객체들의 배열은 다음과 같이 정렬할 수 있다.

```java
Arrays.sort(a);
```

이는 검색, 극단값 계산, 자동 정렬되는 컬렉션 관리도 쉽게 할 수 있다.

다음 코드는 명령줄 인수들을 알파벳순으로 출력하는데, String이 Comparable을 구현한 덕분이다.

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Colletions.addAll(S, args);
        System.out.println(s);
    }
}

// String이 구현한 Comparable
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    ...
}
```

사실상 Java 플랫폼 라이브러리의 모든 값 클래스와 enum이 Comparable을 구현했다.(item34) 

알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 Comparable 인터페이스를 구현하는 것이 좋다.

## compareTo() 메서드 일반 규약

```java
public interface Comparable<T> {
    int compareTo(T t);
}
```

compareTo 메서드의 일반 규약은 다음과 같으며, sgn은 부호 함수(signum function)을 뜻한다.

- Comparable을 구현한 클래스는 모든 x, y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. (따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질 때 예외를 던져야 한다.
    - 두 객체의 참조순서를 바꿔 비교해도 예상한 결과가 나와야 한다는 의미이다.

- 추이성을 보장해야 한다. 즉 x.compareTo(y) > 0 && y.compareTo(z) > 0이면 x.compareTo(z) > 0 이다.

- 모든 z에 대해 x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z)) 이다.
    - 크기가 같은 객체들 끼리는 어떤 객체와 비교하더라도 항상 같아야 함을 의미한다.

- (x.compareTo(y) == 0 ) == (x.equals(y)) 여야한다. 이는 필수는 아니지만 꼭 지키는게 좋다.
    - compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다.
    - 이 클래스의 객체를 컬렉션에 넣으면 해당 컬렉션이 구현한 인터페이스(Collection, Set, Map..)에 정의된 동작과 다르게 동작할 수 있다.
    
    ```java
    @Test
    public void DecimalTest() throws Exception {

        BigDecimal a = new BigDecimal("1.0");
        BigDecimal b = new BigDecimal("1.00");

        Set<BigDecimal> hs = new HashSet<>();
        hs.add(a);
        hs.add(b);


        Set<BigDecimal> ts = new TreeSet<>();
        ts.add(a);
        ts.add(b);
        
        Assertions.assertThat(hs.size()).isEqualTo(2);
        Assertions.assertThat(ts.size()).isEqualTo(1);
    }
    ```
    - 이 인터페이스들은 equals 메서드의 규약을 따르지만, 정렬된 컬렉션들은 동치성을 비교할 때는 equals 대신 compareTo를 사용한다. 코드로 살펴보자.
    - 예시에서 빈 HashSet 인스턴스를 생성한 뒤, '1.0'과 '1.00'을 차례로 추가했다.
    - 두 BigDecimal은 equals 메서드로 비교시 서로 다르기 때문에 HashSet은 원소 2개를 갖게된다.
    - TreeSet의 경우, compareTo 메서드로 비교하여 하나의 원소를 가진다.
    - 이는 compareTo 메서드로 비교하게되면 두 BigDecimal 인스턴스가 똑같기 때문이다.

Comparable은 타입을 인수로 받는 제네릭 인터페이스로, compareTo 메서드의 인수타입은 컴파일 타임에 결정된다. 

이 때 인수 타입 자체가 잘못됐다면, 컴파일 자체가 되지 않을 것이고 null을 인수로 넣어 호출한다면 NullPointerException을 던져야 한다.


이펙티브 자바 2판에서는 compareTo 메서드에서 정수 primitive type 필드를 비교할 때는 관계 연산자 '<', '>'를, 실수 type 필드를 비교할 때는 정적 메서드인 Double,Float.compare를 사용하라고 권했다.

그런데 Java 7부터는 compareTo 메서드에서 관계 연산자를 사용하는 방식은 오류를 유발하여 추천되지 않는다. 

클래스에 핵심 필드가 여러 개라면 핵심적인 필드부터 비교해나가자. 비교 결과가 0이 아니라면(순서가 결정되면), 거기서 끝이기 때문이다. 따라서 결과를 곧장 반환하자. 예시는 다음과 같다.

```java
// 기본 타입 필드가 여러개일 경우
public int compareTo(PhoneNumber pn){
    int result = Short.compare(areaCode, pn.areaCode); // 가장 중요한 필드
    if(result == 0){
        result = Short.compare(prefix, pn.prefix); // 두 번째로 중요한 필드
        if(result == 0){
            result = Short.compare(lineNum, pn.lineNum); // 세 번째로 중요한 필드
        }   
    }
    return result;
}
```

Java 8에 들어서는 Comparator 인터페이스가 일련의 비교자 생성 메서드(comparator construction method)와 메서드 연쇄 방식으로 비교자를 생성할 수 있게 됐다.

```java
private static final Comparator<PhoneNumber> COMPARATOR = Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode).thenComparingInt(pn -> pn.lineNum).thenComparingInt(pn -> pn.prefix);

    public int compareTo(PhoneNumber pn){
        return COMPARATOR.compare(this, pn);
    }
```

## Comparator 주의사항

```java
// hashCode 값의 차를 기준으로 하는 Comparator
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
}
```

위의 코드는 정수 오버 플로우를 일으키거나 부동소수점 계산방식에 다른 오류를 야기할 수 있다. 위의 방법대신 다음의 두 방식 중 하나를 사용하자.

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2){
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
}
```

첫 번째 방식은 정적 compare 메서드를 활용한 Comparator이다. 

이는 인스턴스를 생성하지 않고 바로 사용할 수 있으며, 인스턴스 생성에 따른 오버헤드를 줄일 수 있는 방법이다. 또한 코드가 더 간결해진다.

```java
// Comparator 생성 메서드를 활용한 방법
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o->o.hashCode);
```

두 번째 방식은 lambda 표현식을 이용해 비교자를 정의해 더 간결한 코드를 작성할 수 있으며 가독성을 향상시킬 수 있다.

또한 o.hashCode를 이용한 비교 기준은 직관적이며, 쉽게 이해할 수 있다.

## 참고자료

- 이펙티브 자바 3판
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Comparable.html - docs

