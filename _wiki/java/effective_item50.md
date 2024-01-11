---
layout  : wiki
title   : 적시에 방어적 복사본을 만들라 
summary : 
date    : 2024-01-11 09:49:12 +0900
updated : 2024-01-11 10:44:42 +0900
tag     : java effectivejava
resource: BC/CE91BC-46DE-4FE0-9157-50E50FB8818E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 방어적 복사본의 필요성

Java로 작성한 클래스는 시스템의 다른 부분에서 무슨 짓을 하든 그 불변식이 지켜진다. 메모리 전체를 하나의 거대한 배열로 다루는 언어에서는 누릴 수 없는 강점이다.

하지만 다른 클래스로부터의 침범을 아무런 노력없이 다 막을 수 있는 것은 아니다. 

item50은 기간(period)를 표현하는 클래스를 예를 들었는데 살펴보자.

```java
public class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end 종료 시각(시작 시각 보다 뒤여야한다.)
     *
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦으면 발생
     * @throws NullPointerException start나 end가 null이면 발생
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦습니다");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
    
    ... 
}
```

이 코드의 문제점은 Date 인스턴스가 가변이라는 사실이다. 

Period 클래스는 start와 end를 참조하는데, 이 참조들은 외부에서 생성된 Date 인스턴스를 가리킨다. 즉, 외부에서 Data 인스턴스를 수정할 경우 Period 인스턴스의 상태도 영향을 받게 된다.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정
```

Java 8 이후로는 Date 대신 불변인 Instant를 사용하면 쉽게 해결 가능하다. (LocalDateTime 혹은 ZOnedDateTime)

하지만 Date 처럼 가변인 값 타입을 사용하던 시절이 워낙 길었던 탓에 여전히 많은 API와 내부 구현에 잔재가 남아있다.

따라서 추가적으로 Period 인스턴스의 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다. 

### 방어적 복사

```java
// 방어적 복사본을 만드는 경우
public Period(Date start, Date end) {
    
    // 방어적 복사본 생성
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

    // 유효성 검사 시도
    if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(
            this.start + "after" + this.end);        
}
```

코드를 살펴보면, 매개변수 유효성 검사를 하기 전에 방어적 복사본을 만들고, 그 복사본으로 유효성 검사를 시도한다. 

멀티 스레딩 환경에서는 원본 객체의 유효성 검사 후에 복사본을 만드는 순간에 다른 스레드가 원본 객체를 수정(time of check/time of use)할 수 있기 때문에, 반드시 방어적 복사본을 만들어 유효성 검사를 해야한다.

이처럼 생성자에서 start와 end의 복사본을 저장하여 유효성 검증을 하게되면, Period 인스턴스가 생성된 후에 외부에서 Data 인스턴스를 변경하더라도 Period의 상태가 영향을 받지 않도록 보장할 수 있다.

생성자를 수정하면 앞서 설명한 공격은 막아낼 수 있지만, Period 인스턴스는 접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문에 아직도 변경 가능한 인스턴스이다.

### 가변 필드의 방어적 복사본을 반환

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부 수정
```

period의 인스턴스인 p를 접근자 메서드인 end()로 내부 값을 수정하는 상황이다. 이 경우 가변 필드의 방어적 복사본을 반환하도록 만들면 된다.

```java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
```

접근자 메서드를 수정한 코드이다. (책에서는 접근자라고 나오지만 접근자 메서드를 지칭한 것 같음) 

시작 시각이 종료 시각보다 나중일 수 없다는 불변식을 위배할 방법은 리플렉션과 네이티브 메서드 외에는 없으며, Period 자신 말고는 가변 필드에 접근할 방법이 사라진 상태로 모든 필드가 객체 안에 완벽하게 캡슐화 되었다.

## 방어적 복사의 목적과 고려할 점

매개 변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아니다. 

메서드, 생성자는 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 하는 경우, 그 객체가 잠재적으로 변경될 수 있는지를 염두에 둬야한다. 

변경 가능한 객체라면 그 객체가 클래스에 넘겨진 뒤에 임의로 변경되어도 그 클래스가 문제없이 동작하는지 확인해야 하며, 확신할 수 없다면 방어적 복사본을 만들어 저장해야 한다.

방어적 복사에는 성능 저하가 따르고, 항상 사용할 수 있는 것도 아니다. 

같은 패키지에 속하는 이유로 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적 복사를 생략할 수 있다. 생략 가능하더라도 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 함을 문서화하는 게 좋다.

## 정리

클라이언트로부터 받거나 반환하는 구성요소가 가변이라면 반드시 방어적으로 복사해야 한다.

방어적 복사의 비용이 너무 크거나, 클라이언트가 그 요소를 수정할 일이 없음을 신뢰한다면, 구성 요소를 수정했을 때의 책임이 클라이언트에 있음을 문서화하여 명시하도록 하자!


## 참고자료 

- 이펙티브자바 3판
 
