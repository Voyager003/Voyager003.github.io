---
layout  : wiki
title   : 일반적으로 통용되는 명명 규칙을 따르라 
summary : 
date    : 2024-02-20 09:34:07 +0900
updated : 2024-02-20 09:45:31 +0900
tag     : java effectivejava
resource: DA/893F7A-4D4D-4671-9B0A-28B7FF062257
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java 언어 명세

Java는 명명 규칙이 잘 정립되어 있으며 [명세](https://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html) 에 기술되어 있다.

## 철자 규칙

> 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다.

특별한 이유가 없는 한 반드시 따라야 다른 개발자들의 오해로 인한 실수가 생기지 않는다.

### 패키지 명명

- .으로 구분하여 계층적으로 이름을 지음

- 보통 인터넷 도메인 이름을 역순으로 사용
    - ex) com.google

- 표준 라이브러리 및 선택적 패키지는 java와 javax로 시작

- 이름은 보통 8자 이하의 짧은 단어
    - ex) utilities -> util
    - 약어의 경우, abstract window toolkit -> awt와 같이 가능

### 클래스, 인터페이스 명명

- 열거타입과 애너테이션도 클래스 혹은 인터페이스로 봄

- 각 단어는 대문자로 시작

- 널리 통용된 줄임말(max, min 등)을 제외하고는 단어를 줄여쓰지 않기

- 약자의 경우는 첫글자만 대문자로 할지 약간의 논쟁이 있으나, 첫글자만 대문자로 하는게 알아보기 좋음
    - ex) HttpUrl vs HTTPURL

- 전체를 대문자로 하면 어디서 끊어 읽어야 하는지에 대한 구분이 어렵다.

### 메서드, 필드 이름 명명

- 첫 글자를 소문자로 하는 것 이외에는 클래스, 인터페이스 명명법과 동일
    - ex) remove(), ensureCapacity()

### 상수 필드 명명

- 상수 필드는 enum 의 열거 이름들 static final 접근 제어자로 표현되는 필드를 의미

- 모두 대문자로 쓰며, 단어 사이는 밑줄로 구분
    - ex) VALUES, NEGATIVE_INFINITY

### 지역 변수 명명

- 지역 변수에 한해서는 문맥에서 의미를 쉽게 유추할 수 있는 경우에 한해 약어를 사용

- 입력 매개변수도 지역 변수의 일종

### 타입 매개변수(제네릭) 명명

T: 임의의 타입
E: 컬렉션 원소의 타입
K, V: 맵의 키와 값
X: 예외
R: 메서드의 반환 타입
T, U, V, T1, T2, T3: 임의의 타입 시퀀스

## 문법 규칙

### 클래스와 인터페이스

- 객체를 생성할 수 있는 클래스(열거타입 포함)라면, 단수명사 혹은 명사구를 사용
    - ex) Thread, PrioirtyQueue, ChessPiece

- 객체를 생성할 수 없는 클래스라면, 보통 복수형 명사로 지음
    - ex) Collections, Collectors

- 인터페이스 이름은 able 혹은 ible 로 끝나는 형용사로 짓거나 클래스와 똑같이 지음
    - ex) Runnable, Iterable, Accssible, Closeable

### 어노테이션

- 어노테이션은 지배적인 규칙 없이 명사, 동사, 전치사, 형용사 두루 쓴다.
    - ex) BindingAnnotation, Inject, ImplementedBy, Singleton

### 메서드

- 동작을 수행하는 메서드는 동사나 동사구로 지음
    - ex) append, drawImage

- boolean 값을 반환하는 메서드는 is나 has로 시작하고, 명사, 명사구, 형용사로 기능하는 아무 단어나 구로 끝나도록 지음
    - ex) isDigit, isProbablePrime, isEmpty, isEnable

- 반환 타입이 boolean 이 아니거나, 속성을 반환하는 메서드는 명사, 명사구 혹은 get으로 시작하는 동사구로 지음
    - ex) size, hashCode, getTime

- 객체의 타입을 바꿔서 다른 타입의 같은 내용의 객체를 반환할 때는 보통 toType 으로 지음
    - ex) toString, toArray

- 객체의 내용을 다른 뷰로 보여주는 메서드는 asType 형태로 지음
    - ex) asList

- 객체의 값을 기본 타입 값으로 반환하는 메서드의 이름은 보통 typeValue 형태로 지음
    - ex) intValue

- 정적 팩터리의 경우, from, of 등의 네이밍을 씀
    - ex) valueOf, list.of

### 필드

- 필드 이름은 클래스, 인터페이스, 메서드 이름에 비해 명확하고 덜 중요
- 
- boolean 타입의 필드 이름은 보통 boolean 접근자 메서드에서 앞 단어를 뺀 형태
    - ex) initialized, composite

- 다른 타입의 필드라면 명사, 명사구를 사용
    - ex) height, digits, bodyStyle

## 참고자료

- 이펙티브자바 3판

