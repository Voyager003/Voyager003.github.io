---
layout  : wiki
title   : 비검사 경고를 제거하라 
summary : 
date    : 2023-10-06 16:55:49 +0900
updated : 2023-10-06 19:56:58 +0900
tag     : java effectivejava
resource: D9/8B5547-86AE-4DBB-B82A-DBBAB1907428
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 비검사 경고(Unchecked assignment)

```java
Set<String> set = new HashSet();

// Raw use of parameterized class 'HashSet'

// 다이아몬드 연산자를 사용
Set<String> set = new HashSet<>();
```

제네릭 타입을 사용하면 비검사 형변환, 비검사 메서드 호출, 비검사 매개변수화 가변인수 타입,, 비검사 변환 경고 등 많은 컴파일러 경고를 볼 수 있다. 

컴파일러가 알려준대로 수정하면 경고가 사라지지만, 컴파일러가 알려준 타입 매개변수를 명시하지 않고 Java 7부터 지원하는 다이아몬드 연산자('<>')를 사용하면 컴파일러가 올바른 실제 타입 매개변수를 추론한다.

## @SuppressWarnings("unchecked")

비검사 경고를 제거할 수 없지만 type-safe하다고 확신할 수 있다면 @SuppressWarning("unchecked") 애노테이션을 명시해 경고를 숨길 수 있다.

하지만, type-safe을 검증하지 않은 채 경고를 숨긴다면, 컴파일은 되지만 런타임에는 ClassCastException 오류가 발생할 수도 있다.

### 사용법

개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 명시할 수 있지만, 항상 가능한 좁은 범위에 적용하자. 보통은 변수 선언, 짧은 메서드 혹은 생성자가 될 것이다.

```java
    public <T> T[] toArray(T[] a) {
        if (a.length < size) {
            @SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
            return result;
        }
        
        System.arraycopy(elements, 0, a, 0, size);
        if (a.length > size) {
            a[size] = null;
        }
        return a;
    }
```

예시의 경우 생성한 배열(T[])과 매개변수로 받은 배열의 타입이 모두 같아 올바른 형변환이기 때문에, type-safe하다고 볼 수 있어 애노테이션을 명시했다.

이처럼 한 줄이 넘는 메서드나 생성자에 달린 애노테이션은 지역변수 선언쪽으로 옮기는 것이 좋다. 

결과적으로 깔끔하게 컴파일되고 비검사 경고를 숨기는 범위도 최소로 좁힐 수 있었다. 추가적으로 @SuppressWarnings를 명시할 때 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.

비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 내포하고 있으니 최선을 다해 제거하자!

## 참고자료

- 이펙티브 자바3판

