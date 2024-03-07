---
layout  : wiki
title   : 메서드가 던지는 모든 예외를 문서화하라
summary : 
date    : 2024-03-07 09:31:19 +0900
updated : 2024-03-07 09:39:28 +0900
tag     : java effectivejava
resource: 49/143807-BC2B-4CB0-9DB9-E253A8F49306
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 메서드 예외 문서화

메서드가 던지는 예외는 해당 메서드를 올바르게 사용하는 데 중요한 정보이다. 

item74에서 제시하는 문서화 방법은 다음과 같다.

- 검사 예외는 항상 따로따로 선언
- 각 예외가 발생하는 상황을 JavaDoc의 @throw 태그를 사용해 문서화
- 공통 상위 클래스하나로 통틀어서 선언(Exception, Throwable 던지는 작업)하는 일을 피하기
    - 유일한 예외는 main() 메서드
- 비검사 예외를 문서화하여 오류들이 무엇인지 알려 메서드를 성공적으로 수행하기 위한 전제조건을 알림 
- 메서드가 던질 수 있는 예외를 @throws 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 않기
- 한 클래스에 정의된 많은 메서드가 같은 이유로 예외를 던진다면 그 예외를 클래스 설명에 추가
    - NullPointerException이 그 예로 클래스 무선화 주석에 인수로 null이 넘어오면 예외를 던진다라고 명시

## 참고자료

- 이펙티브자바 3판

