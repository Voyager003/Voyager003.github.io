---
layout  : wiki
title   : 메서드 시그니처를 신중히 설계하라 
summary : 
date    : 2024-01-15 09:39:59 +0900
updated : 2024-01-15 10:10:27 +0900
tag     : java effectivejava
resource: 92/962A40-FC63-4EF3-AB17-7815E129A69B
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## API 설계 요령

### 메서드 이름을 신중하게 짓기

- 표준 명명 규칙을 따른다. (item68)
- 쉽게 이해 가능하고, 같은 패키지에 속한 다른 이름들과 일관되도록 짓는게 최우선
- 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용 (컨벤션)
- 긴 이름을 피한다.

### 편의 메서드 많이 만들지 않기

- 메서드가 너무 많은 클래스는 사용, 문서화, 테스트, 유지보수가 어렵다.
- 클래스나 인터페이스는 각 기능을 완벽히 수행하는 메서드로 제공해야 한다.
- 자주 쓰일 경우에만 별도의 약칭 메서드를 두고, 확신이 서지 않으면 만들지 않는다.

### 매개변수 목록은 짧게 유지하기

매개변수 목록은 4개 이하가 적당하며, 같은 타입의 매개변수 여러 개가 연달아 나오는 경우는 특히 해롭다. 

이유는 매개변수 순서를 기억하기 어려울뿐더러 순서를 바꿔 입력해도 컴파일되고 실행되지만, 의도와 다르게 동작한다. 이를 위해 다음 지침을 소개한다.

- 여러 메서드로 쪼갠다. 
    - 쪼개진 메서드 각각은 원래 매개변수 목록의 부분집합을 받는다. (직교성을 높임 = 공통점이 없는 기능들이 잘 분리되어 있음을 의미)
    - 이 때 API가 다루는 개념의 추상화 수준에 맞게 쪼개야 한다.

- 매개변수 여러 개를 묶는 도우미 클래스 생성
    - [정적 멤버 클래스](https://voyager003.github.io/wiki/java/effective_item24/) 로 둔다. 
    - 매개변수 몇 개를 독립된 하나의 개념으로 볼 수 있는 경우에 적합

- 빌더 패턴
    - 매개 변수가 많을 때, 특히 그 중 일부는 생략해도 괜찮은 경우 적합
    - 모든 매개변수를 하나로 추상화한 객체를 정의하고, setter를 호출해 필요한 값을 설정
    - 각 setter 메서드는 매개변수 하나 혹은 연관된 몇 개만 설정
    - 매개변수 설정 뒤, execute 메서드를 호출해 앞서 설정한 매개변수들의 유효성을 검사
    - 객체를 넘겨 원하는 계산을 수행

### 매개변수의 타입은 클래스보다 인터페이스

- 매개변수로 적합한 인터페이스가 있으면, 인터페이스를 직접 사용하자.
- 예컨대 메서드에 Map을 사용하면, HashMap, TreeMap, ConcurrentHashMap 등 어떠한 Map 구현체도 인수로 받을 수 있다.
- 인터페이스 대신 클래스를 사용하면, 클라이언트에게 특정 구현체만 사용하도록 제한되기 때문에 유연성이 떨어짐

### boolean보다 원소 2개짜리 열거 타입

```java
public enum TemperatureScale {
    FAHRENHEIT, CELSIUS 
}
```

코드를 읽고 쓰기가 쉬워지며, 선택지를 추가하기도 쉽다.

또한 온도 단위에 대한 의존성을 개별 열거 타입 상수의 메서드 안으로 리팩토링하여 넣을 수 있다.

```java
public enum TemperatureScale {

    FAHRENHEIT {
        @Override
        public double toCelsius(double fahrenheit) {
            return (fahrenheit - 32) * 5 / 9;
        }
    },
    CELSIUS {
        @Override
        public double toCelsius(double celsius) {
            return celsius;
        }
    };

    public abstract double toCelsius(double value);
}
```

## 참고자료 

- 이펙티브자바 3판

