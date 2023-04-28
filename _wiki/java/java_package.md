---
layout  : wiki
title   : Java package 
summary : 
date    : 2023-04-28 16:45:08 +0900
updated : 2023-04-28 16:45:22 +0900
tag     : java
resource: BA/64F324-7DA3-4435-8E7D-AEC1EEA63F1F
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/7) 7주차 과제

## package 키워드

> 관련된 클래스와 인터페이스를 논리적으로 그룹화하는 방법

```java
// 선언 예시 : package package명;
package com.javastudy.packagestudy;

```

- 관련 클래스를 구조화 및 묶음으로써 FQCN(Fully Qualified Class Name)의 고유성을 보장하기 위해 사용한다.
  - 이는 클래스를 유일하게 만들어주는 '식별자' 역할을 한다
  - 예를 들어, String의 클래스 패키지는 'java.lang'이며 FQCN은 'java.lang.String'이 된다.
- '.'을 구분자로한 계층구조로 형성되어 있다.
- 모든 클래스는 반드시 하나의 패키지에 속하며, 패키지 미선언 시, Java에서 기본적으로 제공하는 unnamed package에 속한다.
- 숫자로 시작해서 안된다.
- _, $ 를 제외한 특수문자를 사용해서는 안된다.
- java로 시작하는 패키지는 자바 표준 api 에서만 사용하므로 사용해서는 안된다.
- 모두 소문자로 작성하는 것이 관례이다.
- 소스코드에서 주석과 공백을 제외한 첫 줄에 한 번만 선언되어야 한다.

### Built-in Package(빌트-인 패키지)

> JDK, JRE에 내장되어 제공하는 패키지

- java.lang 혹은 java.util같은 패키지는 기본적으로 내장되어있기 때문에 import하지 않아도 클래스를 불러오는데, 이를 빌트-인 패키지라 한다.
- 종류는 다음과 같다.
  - **java.lang** : language support 클래스들을 포함하는 패키지
  - primitive type, 수학 연산의 정의가 되는 클래스가 포함
  - 자동 import되기 때문에 해당 패키지의 클래스를 바로 사용할 수 있다.
  - **java.io** : 입출력 기능을 지원하는 클래스들을 포함하는 패키지
  - **java.util** : 자료구조 구현을 위한 유틸리티 클래스를 포함하는 패키지
  - **java.applet** : Applets을 생성하기 위한 클래스들을 포함하는 패키지
  - **java.awt** : GUI 컴포넌트를 구현하기 위한 클래스들을 포함하는 패키지
  - **java.net** : 네트워크 기능을 지원하기 위한 클래스를 포함하는 패키지

## import 키워드

> 다른 패키지에 있는 클래스를 사용해야할 때, 패키지명을 생략하기 위해 사용한다

```java
// import package.class명;
import com.javastudy.packagestudy.pack;
import com.javastudy.packagestudy.*; // packagestudy 패키지에 해당하는 모든 클래스를 사용

// static import
import static java.lang.Math.random;

//main 함수에서 실행
public class Packagetest {

    public static void main(String[] args) {
        System.out.println(random()); // static 미사용 시, Math.random()
    }
}
```

- 같은 패키지에 있는 클래스들은 import문을 선언하지 않아도 패키지명이 생략 가능하다.
- 이는 컴파일러에게 소스코드에 사용된 클래스의 패키지 정보를 제공한다.
- static import 시, 메서드 혹은 변수를 패키지, 클래스명을 생략하고 접근가능하도록 한다.

## Classpath

> Java 컴파일러, JVM이 사용자정의 클래스, 패키지의 위치를 지정해주는 패러미터

```java
// 명령어 -classpath 클래스패스 경로
// 명령어 -cp 클래스패스 경로

javac -classpath C:\java\jdk1.8.0 classpath.java
```

- 클래스패스를 지정하지 않으면, 기본적으로 현재 디렉토리가 클래스패스로 지정된다.
- ';'을 구분자로 여러 경로를 지정할 수 있다.

### Classpath 환경변수

- 환경 변수는 OS에 지정하는 변수로 JVM과 같은 애플리케이션들이 환경변수 값을 참고하여 동작한다.
- Java의 경우 클래스 패스로 CLASSPATH를 사용하는데, 값 지정 시 옵션을 사용하지 않아도 된다.
- 최근에는 OS 상의 환경변수로 클래스패스 설정을 지양하고 IDE 혹은 Build tool를 통해 클래스 패스를 설정한다.

## 접근 지시자(Access Modifier)

> 외부로부터 접근 제한을 막기 위한 역할

- 인스턴스 생성을 막기위해 생성자를 호출못하게 하거나, 특정 메서드를 호출할 수 없도록 제한한다.
- 이 기능을 제한하기 위해 접근지시자를 이용한다. 
- 제한이 큰 순서대로 나열하면 다음과 같다.
  - public : 모든 패키지에서 제한없이 호출할 수 있다. 기본 생성자, 필드, 메서드가 public이라면 클래스도 public 제한을 가진다.
  - protected : 같은 패키지에 속하는 클래스에서 호출할 수 있도록 하며, 다른 패키지에 속한 클래스가 해당 클래스의 자식 클래스라면 호출할 수 있다.
  - default : 접근 제한자를 생략하면 default이며, 같은 패키지에서 제한없이 호출할 수 있지만, 다른 패키지에서 호출할 수 없다.
  - private : 오직 클래스 내부에서만 사용하도록 제한한다.

## 참고자료

- https://gintrie.tistory.com/67