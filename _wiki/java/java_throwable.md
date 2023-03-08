---
layout  : wiki
title   : Java 예외 클래스 
summary : 
date    : 2023-03-07 20:33:23 +0900
updated : 2023-03-07 21:31:03 +0900
tag     : java
resource: C7/8F4668-FE8A-4305-BDB7-CD9E06CA8A4E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Exception(예외)

> 프로그램이 Java 프로그래밍 언어의 의미적 제약 조건을 위반할 때, JVM은 이 오류를 예외로 프로그램에 알린다. 

예외는 다음 이유로 인해 예외가 발생한다.

## 예외의 종류

<img width="637" alt="스크린샷 2023-03-08 오전 12 31 47" src="https://user-images.githubusercontent.com/85725033/223469451-6a4d8302-8cd4-41a9-962b-39dc616b54b0.png">

### Throwable(Checked)

> Exception 과 Error class는 Throwable의 직접적인 sub class이다.

**Throwable**는 예외처리 최상위 클래스이다.

### Error(Unchecked)

> 프로그램이 일반적으로 복구되지 않을 것으로 예쌍되는 모든 예외의 super class이다.

**Error**는 메모리 부족 및 심각한 시스템 오류와 같은 애플리케이션의 복구 불가능한 시스템 예외로, 개발자는 오류가 발생하도록 하고 예외를 잡으려고 해선 안된다.

### Exception(Checked)

> 프로그램이 복구하고자 하는 모든 예외의 super class이다.

애플리케이션 로직에서 사용할 수 있는 실질적 최상위 예외 클래스이다.

Exception과 하위 예외는 모두 Compiler가 Check하지만, 하기에 설명된 Runtime Exception은 예외로 한다.

#### Runtime Exception(Unchecked)

Compiler가 체크하지 않는 unchecked 예외이다. nullpointerExcpetion, IllegalArgument.. 등 예외를 포함한다.

## 참고자료

- https://docs.oracle.com/javase/specs/jls/se11/html/jls-11.html#jls-11.1.1 - Java docs
- https://sites.google.com/site/myyclassnotes/java/exceptionhandlinginjava - 예외 계층 이미지


