---
layout  : wiki
title   : public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라 
summary : 
date    : 2023-08-06 23:32:30 +0900
updated : 2023-08-07 09:50:20 +0900
tag     : java effectivejava
resource: 7D/8F3A74-D71B-487E-BCE1-1371A857B061
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## public 필드를 가진 클래스

```java
class Point {
    public double x;
    public double y;
}
```

item16에서 말하는 위 클래스의 문제점은 다음과 같다.

- 변수 x, y는 public으로 선언되어 데이터 필드에 직접 접근이 가능하여 캡슐화의 이점을 제공하지 못한다.

- API를 수정하지 않고서는 내부 표현을 바꿀 수 없다.

- 불변식을 보장할 수 없다.

- 외부에서 필드 접근 시, 부수 작업을 수행할 수 없다. 여기서 부수 작업이란 인스턴스의 상태를 변경하거나 동작을 수행하는 것을 의미한다.

이를 해결하기 위해서는 어떤 방법을 사용하는게 좋을까?

## 접근자와 변경자(mutator) 메서드를 활용한 캡슐화

```java
@Getter @Setter
class Point{
    private double x;
    private double y;

    public Point(double x, double y){
        this.x = x;
        this.y = y;
    }
}    
```

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공하여 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성이 있다.

하지만 package-private(패키지 바깥 코드를 손대지 않고 데이터 표현 방식 변경가능), private 중첩 클래스(이 클래스를 포함하는 외부 클래스까지로 제한)의 경우 데이터 필드를 노출한다고 해도 그 클래스가 표현하려는 추상 개념만 올바르게 표현한다면 문제없다. 

접근자 방식보다 훨씬 깔끔하며, 패키지 바깥 코드 신경쓰지 않고 데이터 표현 방식을 바꿀 수 있다.

## public 클래스의 불변 필드

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_DAY = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute){
        if(hour < 0 || hour >= HOURS_PER_DAY){
            throw new IllegalArgumentException("시간 : "+ hour);
        }
        if(minute < 0 || minute >= MINUTES_PER_DAY){
            throw new IllegalArgumentException("분 : "+ hour);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

이처럼 public 클래스의 필드가 불변(final)이라면 불변식은 보장할 수 있게되어 직접 노출할 때의 단점은 줄어들지만, 여전히 단점이 존재한다.

먼저 내부 표현 변경 시, API 수정이 필요하다. 예를 들어 Time 클래스가 지금은 시간과 분으로 표현하고 있지만, 이를 시간과 초로 변경하게 된다면 minute 필드를 second로 변경해야한다. 이는 클래스 외부 사용자들에게 영향을 미치게 된다.

또한 필드를 읽을 때 부수 작업을 수행할 수 없다. hour와 minute 필드는 인스턴스 생성 시 생성자에서 초기화되며 이후에는 변경될 수 없는 불변 필드이다.

Time의 hour와 minute의 값을 읽으려면 해당 필드에 접근해 값을 가져오면 된다. 부수 작업을 수행할 수 없다는 것은 이러한 필드를 읽을 때 값 자체만을 가져오며 추가적 동작이 필요하지 않다는 것을 의미한다.

## 참고자료

- 이펙티브 자바 3판
 
