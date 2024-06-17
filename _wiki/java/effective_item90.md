---
layout  : wiki
title   : 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라 
summary : 
date    : 2024-06-17 19:38:17 +0900
updated : 2024-06-17 21:21:17 +0900
tag     : java effectivejava
resource: AB/0C11D5-9C43-4AE6-A829-6FAF556915F9
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

Serializable을 구현하기로 결정한 순간 생성자 이외의 방법으로 인스턴스를 생성할 수 있게되어 보안 문제가 일어날 가능성이 커진다.

item90은 이 위험을 줄여줄 기법으로 직렬화 프록시 패턴을 소개한다.

## 직렬화 프록시 패턴(serialization proxy pattern)

`직렬화 프록시 패턴`은 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다.

여기서 중첩 클래스가 바깥 클래스의 직렬화 프록시가 된다. 이 때, 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다. (일관성 복사나 방어적 복사도 필요하지 않다.)

코드로 살펴보자.

```java
// Period
public class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end) {
        ...
    }
}

// SerializationProxy
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;
    
    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }
    
    private static final long serialVersionUID = 4359837952347u96L; // 무작위 값
}
```

코드는 item88에서 살펴봤던 Period 클래스의 직렬화 프록시다. 코드를 보면 직렬화 프록시도 바깥 클래스(Period)와 동일한 필드로 구성되어 있다.

이제 바깥 클래스에 직렬화 프록시 패턴용 메서드를 추가한다.

```java
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

wirteReplace() 메서드는 java의 직렬화 시스템이 바깥 클래스의 인스턴스 대신 SerializationProxy의 인스턴스를 반환하는 역할을 한다. 이는 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다.

이로써 직렬화 시스템은 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다. 

하지만 공격자는 불변식을 훼손하고자 하는 시도를 할 수 있다. readObject 메서드를 바깥 클래스에 추가하면 공격을 막을 수 있다.

```java
private void readObject(ObjectInputStream stream) thorws InvalidObjectException {
    throw new InvalidObjectException("need proxy");
}
```

바깥 클래스에 추가된 readObject 메서드가 InvalidObjectException를 던지도록하여 직렬화 프록시를 사용하지 않고 클래스를 직접 역직렬화하려는 시도를 차단한다.

이제 마지막으로 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 SerializationProxy 클래스에 추가한다. 

```java
private Object readResolve() {
    return new Period(start,end);
}
```

readResolve() 메서드는 역직렬화 시, 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 한다.

이는 공개된 API 만을 사용해 바깥 클래스의 인스턴스를 생성하게 된다. 즉, 일반 인스턴스를 생성할 때와 같은 생성자, 정적 팩토리, 다른 메서드를 사용해 역직렬화된 인스턴스를 생성하는 것이다.

코드로 살펴본 직렬화 프록시 패턴의 장점을 정리하면 다음과 같다. 

- 방어적 복사처럼, 직렬화 프록시 패턴은 **가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단**하며

- Period 필드를 final로 선언해도 되므로 Period 클래스를 **진정한 불변**으로 만들 수 있으며,

- 어떤 필드가 **직렬화 공격의 목표가 될지 고민하지 않아도 되며, 역직렬화 때 유효성 검사를 수행하지 않아도** 된다.

- 또한 **역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동**한다.

### 한계 

하지만 한계점도 있는데 다음과 같다.

- 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용 불가

- 객체 그래프에 순환이 있는 클래스에는 적용 불가

순환이 있는 클래스의 메서드를 직렬화 프록시의 readResolve 안에서 호출하려고 시도한다면, 직렬화 프록시만 가졌을 뿐 실제 객체는 만들어지지 않았기 때문에 ClassCastException이 발생한다.

- 방어적 복사보다 느리다.. 
  
느리지만, 강력함과 안정성을 보장해주는 좋은 방법!

## 참고자료

- 이펙티브자바 3판

