---
layout  : wiki
title   : 다른 타입이 적절하다면 문자열 사용을 피하라 
summary : 
date    : 2024-02-06 09:40:49 +0900
updated : 2024-02-06 10:12:12 +0900
tag     : java effectivejava
resource: 9D/44ED8E-D116-41A3-83B3-5A829177D327
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 문자열(String)을 사용하지 않아야 하는 사례

- 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
    - 입력받을 데이터가 진짜 문자열일 때만 사용하는 게 좋다.
    - 만약 데이터가 수치형이라면 int, float와 같은 수치 타입을, 예/아니오 질문의 답이라면 열거 타입이나 boolean으로 변환해야 한다.
- 열거 타입을 대신하기에 적합하지 않다.
    - 상수 열거 시에는 문자열보다는 열거 타입이 월등히 낫다. [item34](https://voyager003.github.io/wiki/java/effective_item34/)
- 혼합 타입을 대신하기에 적합하지 않다.
    - 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 좋지 않다.
    ```java
    String compoundKey = className + "#" + i.next();
    ```
    - 위 코드의 경우 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 오류 가능성이 커진다.
    - 또한 equals, toString, compareTo 메서드 제공이 불가능하며, String이 제공하는 기존에만 의존해야 한다.
    - 이는 priavte 정적 멤버 클래스로 선언한 전용 클래스를 만드는 편이 낫다.
- 권한을 표현하기에 적합하지 않다.
    ```java
    public class ThreadLocal {
        private ThreadLocal() {} // 객체 생성 불가

        // 현 스레드 값을 키로 구분해 저장
        public static void set(String key, Object value);

        // 현 스레드의 값을 반환
        public static Object get(String key);
    }
    ```
    - 스레드 구분용 문자열 키가 전역 이름공간에서 공유되고 있다.
    - 의도대로 동작하려면 각 클라이언트가 고유 키를 제공해야 한다.
    - 같은 키를 쓰기로 결정한다면, 같은 변수를 공유하여 기능을 못하며, 보안에도 취약하다.
    
    ```java
    public final class ThreadLocal<T> {
        public ThreadLocal();
        public void set(T value);
        public T get();
    }
    ```

    - ThreadLocal을 매개변수화 타입으로 선언하여 Type-safe를 보장한다.
    - 이는 문자열 기반 API의 문제를 해결하고, 키 기반 API보다 빠르다.
    
## 참고자료
- 이펙티브자바 3판

