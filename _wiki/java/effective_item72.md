---
layout  : wiki
title   : 표준 예외를 사용하라 
summary : 
date    : 2024-03-04 09:49:45 +0900
updated : 2024-03-04 10:12:07 +0900
tag     : java effectivejava
resource: 07/430453-D597-4D60-B8EA-601E263E5F01
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 표준 예외

Java 라이브러리는 대부분 API에서 쓰기에 충분한 예외를 제공한다. 

표준 예외를 재사용한다면 익숙해진 규약을 따르기 때문에 API 사용 시 다른 사용자들 익히고, 접근하기 쉽다. 또한 예외 클래스의 수가 적을수록 메모리 사용량이 줄어들며 클래스를 적재하는 시간도 적게 걸린다.

item72에서 설명하는 가장 많이 재사용되는 예외로 IllegalArgumentException, StateException이 있다. 각각 호출자가 인수로 부적절한 값을 넘기는 경우와 객체의 상태가 호출된 메서드를 수행하기 적합하지 않을 때 던지는 예외이다. 

그 외에 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때 던지는 Concurrent
ModificationException과 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던지는 UnsupportedOperationException 등이 있다.

이를 고려하여 Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말도록 하자. 이는 여러 성격의 예외들을 포괄하는 클래스로 안정적인 테스트가 힘들다.

API 문서를 참고하여 예외가 어떤 상황에 던져지는지 고려하여 예외가 던져지는 맥락과 부합할 때 재사용하도록 하자.

## 참고자료

- 이펙티브자바 3판

