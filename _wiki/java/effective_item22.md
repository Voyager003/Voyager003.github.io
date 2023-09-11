---
layout  : wiki
title   : 인터페이스는 타입을 정의하는 용도로만 사용하라 
summary : 
date    : 2023-09-11 09:42:36 +0900
updated : 2023-09-11 10:27:58 +0900
tag     : java effectivejava
resource: 0E/2A0086-A3A1-439A-B93C-D62594442142
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 인터페이스의 용도

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 하는데, 이는 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 말해주는 것이다. 인터페이스의 용도는 바로 이것이다.

item22는 용도에 맞지 않는 예로 상수 인터페이스를 들었다. 코드로 살펴보자.

```java
// 안티패턴 상수 인터페이스
public interface PhysicalConstants{

      static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      
      static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      
      static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```

상수 인터페이스는 메서드 없이 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스를 말한다. 정규화된 이름을 쓰는 걸 피하고자 인터페이스를 구현했지만 이는 안티패턴이다. 

이유는 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위기 때문이다. 크랠스가 어떤 상수 인터페이스를 사용하든 사용자에게는 아무런 의미가 없는 행위다. 오히려 사용자에게 혼란을 주고, 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속될 수 있다. final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위 클래스의 이름공간이 해당 인터페이스가 정의한 상수들로 오염될 수 있다.

이처럼 상수를 공개할 때의 문제점을 피하려면 어떻게 해야할까?

## 선택지

- 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가한다.

예시로 모든 숫자 Primitve type의 Boxing 클래스가 대표적으로, Integer의 MIN_VALUE와 MAX_VALUE가 이런 예다.

- 열거(ENUM) 타입으로 만들어 공개한다.

열거 타입으로 나타내기 적합한 상수라면 다음과 같이 작성할 수 있을 것이다.

```java
public enum Day{ MON, TUE, WED, THU, FRI, SAT, SUN};
```

- 인스턴스화할 수 없는 유틸리티 클래스([item4](https://voyager003.github.io/wiki/java/effective_item4/))에 담아 공개한다.

```java
public class PhysicalConstants{
      private PhysicalConstants(){}; // 인스턴스화 방지
      public static final double AVOGARDROS_NUMBER = 6.022_140_857e23;
      public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
      public static final double ELECTRON_MASS = 9.109_383_56e-3;
}
```

유틸리티 클래스에 정의된 상수를 클라이언트에서 사용하려면 'PhysicalConstatns.AVOGADROS_NUMBER'와 같이 클래스 이름까지 함께 명시해야 한다. 빈번히 사용한다면 정적 임포트(static import)하여 클래스 이름은 생략할 수 있다.

결론적으로, item22의 핵심은 인터페이스는 타입을 지정하는 용도로만 사용하고, 상수 공개용 수단으로 사용하지 말자라는 것이다.

## 참고자료

- 이펙티브 자바 3판
