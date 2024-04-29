---
layout  : wiki
title   : 지연 초기화는 신중히 사용하라 
summary : 
date    : 2024-04-29 11:09:51 +0900
updated : 2024-04-29 23:14:36 +0900
tag     : java effectivejava
resource: E5/173889-93C4-4879-9238-8267385C464B
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 지연 초기화(lazy initialization)

지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법으로, 값이 전혀 쓰이지 않으면 초기화도 일어나지 않는다.

정적 필드와 인스턴스 필드 모두에 사용할 수 있으며, 주로 최적화 용도로 쓰이지만 클래스와 인스턴스 초기화 시 발생하는 순환 문제를 해결화는 효과도 있다.

item83의 조언은 이 지연 초기화를 필요하기 전까지 하지 말라이다. 

### 지연 초기화의 문제점 

지연 초기화는 클래스, 인스턴스 생성 시의 초기화 비용은 줄어들지만, 필드에 접근하는 비용은 커진다.

때문에 초기화하려는 필드 들 중 초기화가 이뤄지는 비율에 따라, 초기화된 각 필드를 얼마나 호출하느냐에 따라 지연 초기화가 실제로는 성능을 느려지게 할 수도 있다.


### 필요성

그럼에도 필요한 경우가 있다. 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은데 그 필드를 초기화하는 비용이 큰 경우이다. 이를 알 수 있는 유일한 방법은 지연 초기화 적용 전후의 성능을 측정해보는 것이다.

## 멀티스레드 환경에서의 지연 초기화

지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화가 필요하다.

대부분의 상황에서 일반적인 초기화가 지연 초기화보다 나은데, 예시로 살펴보자.

```java
private final FieldType field = computeFieldValue();
```

코드는 인스턴스 필드를 final 한정자로 선언하여 초기화한 것으로 일반적인 초기화 방법이다.

```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null) 
        field = computeFieldValue();
    return field;
}   
```

지연 초기화가 초기화 순환성을 깨뜨릴 것 같다면 synchronized를 명시한 접근자를 사용하자.

이상의 두 관용구는 정적 필드에도 똑같이 적용된다.

### 지연 초기화 홀더 클래스

```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() {
    return FieldHolder.field;
}
```

성능때문에 정적 필드를 지연 초기화 해야한다면 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자.

getFIeld가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서 FieldHolder 클래스 초기화를 촉발한다. 이 방법은 getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않아 성능이 느려질 이유가 없다는 것이다.

일반적인 VM은 오직 클래스를 초기화할 때만 필드 접근을 동기화할 것이다. 

클래스 초기화가 끝난 뒤에는 VM이 동기화 코드를 제거하여, 그 이후 부터 검사나 동기화 없이 필드에 접근하게 된다.

### 이중검사 관용구(double check)

성능 때문에 인스턴스 필드를 지연 초기화 해야한다면 이중검사를 권한다.

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null) { // 첫 번째 검사(lock X)
        return result;
    }
    
    synchronized(this) {
        if(field == null) // 두 번째 검사(lock O)
            field = computeFieldValue();
        return field;
    }
}       
```

이중검사 관용구는 필드의 값을 두 번 검사하는 방식으로 한 번은 동기화 없이 검사하고 두 번째는 동기화하여 검사한다. 이는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다. 

이 때 필드가 초기화된 이후로는 동기화하지 않으므로 해당 필드는 volatile로 선언해야 한다.

### 단일검사 관용구

반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화 해야하는 경우, 이중검사 관용구에서 두 번째 검사를 생략한 단일검사 관용구를 사용할 수 있다.

```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null) {
        field = result = computeFieldValue();
    return result;
}
```

## 정리

대부분의 필드는 지연시키지 말고 즉시 초기화해야한다.

성능, 위험한 초기화 순환을 막기위해 지연 초기화를 써야한다면 상황에 따른 지연 초기화 방식을 도입하자

## 참고자료

- 이펙티브자바 3판



