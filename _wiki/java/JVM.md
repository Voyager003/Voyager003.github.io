---
layout  : wiki
title   : JVM과 Java 코드가 실행원리
summary : 
date    : 2023-02-18 23:35:34 +0900
updated : 2023-03-25 01:06:18 +0900
tag     : java jvm
resource: B4/0B5237-B7DD-4389-805A-8B799BE6AC14
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study) 과정을 따라 Java 기초 다지기

## Java Virtual Machine(JVM)

> JVM(Java Virtual Machine)은 컴퓨터에서 Java 프로그램은 물론 Java 바이트코드로 컴파일되는 다른 언어로 작성된 프로그램을 실행할 수 
> 있게 해주는 가상 시스템이다. [^1]

- Write once, run anywhere, 1995년 썬 마이크로시스템즈가 만든 Java의 표어로 JVM을 통해 OS나 컴퓨터 장비에 구애받지 않고 사용할 수 있는 이유이다.
- JVM은 OS위에서 작동하여 java 프로그램과 OS 사이의 중계자 역할을 하게되어 OS에 의존하지않고 실행할 수 있다.

## Compile과 실행

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F6ztxH%2FbtrThM2SzWC%2Fnge6925CJA5UdaFkJpke4k%2Fimg.png)

- JVM은 고급 언어로 작성된 소스코드(.java)를 바이트코드(.class)로 변환하는데 크게 다음과 같은 과정을 거친다.
  - 어휘 분석 : public, class(키워드), "Hello World"(리터럴), +(연산자)를 수집하여 모인 어휘소를 하나의 Stream으로 만든다.(Token stream)
  - 구문(Syntax) 분석 : 어휘 분석을 통해 얻은 Token stream이 Java 문법에 맞는지 확인한다. 
  - 의미 분석 : 타입 검사 및 타입 변환을 진행한다.
  - 중간코드 생성 : 구문 분석과 의미분석을 마치면 중간 코드(바이트코드)를 생성한다.
- 생성이 끝나면, 바이트 코드를 JIT 컴파일러가 CPU가 해독할 수 있는 bit 단위로 쓰인 컴퓨터 언어로 만든다.

## 바이트코드

> VM(가상 머신)이 이해할 수 있는 언어로 변환된 코드로 코드의 명령어 크기가 1byte이기 때문에 바이트 코드라고 불린다.

- JVM은 바이트 코드를 읽어 컴퓨터가 이해할 수 있는 언어로 변환한다. 
- JVM 기반 언어인 Ruby, Scala, Kotlin 등 언어도 실행 전, Complie time에서 바이트 코드로 번역되는 과정을 가진다.

## JIT 컴파일러

> 프로그램을 실제 실행하는 시점에 기계어로 번역하는 컴파일 기법

<img width="571" alt="스크린샷 2023-03-24 오후 11 17 00" src="https://user-images.githubusercontent.com/85725033/227548188-a4677a8e-80f8-4c49-80f0-dea958d4ddb9.png">


- Java는 Complie과 Interpeter 방식 모두 사용한다.
- 기계어로 변환된 코드를 캐싱하여 재사용시 컴파일을 하지 않도록 한다.
- Runtime에 바이트 코드를 기계어로 변환하는 컴파일을 하여 이미 캐시에 있는 기계어는 변환하지 않도 다시 사용하지 않는 컴파일 방식이다.
- 이 때, 캐시 공간은 작기 때문에 JVM은 내부에서 자주 수행되는 코드를 선별한다.
  - JIT 컴파일러는 C1, C2 컴파일러로 나뉜다.
  - C1 컴파일러는 Runtime에 바이트 코드를 기계어로 변환하는 과정만을 수행한다.
  - C2 컴파일러는 기계어 변환 후 캐시에 저장하는 과정을 수행한다.

## JVM 구성 요소

![alt](https://upload.wikimedia.org/wikipedia/commons/d/dd/JvmSpec7.png)

### Class Loader

> Class Lodaer는 JRE의 일부로 Runtime에 class를 동적ㅇ로 JVM에 로드한다.

<img width="863" alt="스크린샷 2023-03-24 오후 11 30 07" src="https://user-images.githubusercontent.com/85725033/227553768-45ee78c5-cd59-4cc7-b5f3-3e9eda37955d.png">

- Loading : .class(바이트코드)를 읽고, 내용에 따라 바이너레 데이터를 만들고 메서드 영역에 저장
- Linking
  - Verifying : JVM에서 사용이 가능한 형태인지 검증
  - Preparation : Type이 필요로 하는 Memory 할당
  - Resolving : 심볼릭한(명확하게 정의되지 않은) 메모리 참조를 메서드 영역에 있는 타입으로 직접 참조
- Initialization : Static 변수의 값 할당 및 SuperClass 초기화 후 class 초기화 진행

### JVM Memory

<img width="724" alt="스크린샷 2023-03-25 오전 12 07 29" src="https://user-images.githubusercontent.com/85725033/227563841-90a1a6e4-0edf-4b54-b5af-5f1833a33be0.png">

- Method Area : 모든 class 수준(class 명, superlcass, method, variable)의 데이터가 저장된다.
- Heap Area : 모든 인스턴스 Object(class, array)가 저장되는 공간으로 MultiThread에서 접근이 가능한 공유 자원이다.
- Stack Area : 각 Thread마다 개별 Stack 영역이 존재한다. Method가 호출될 때 마다 Stack에 Stack frame이라는 memory에 쌓인다.
- PC Register : 각 Thread마다 개별적으로 현재 실행중인 명령문의 address를 가진다.
- Native Method Stack : native 메서드 정보를 가진 Stack

### Execution Engine(실행 엔진)

> Class Lodaer를 통해 Runtime data 영역에 할당된 byte code를 실행한다.

상단에서 설명한 인터프리터와 JIT 컴파일러가 이에 해당한다.

## JDK, JRE

<img width="647" alt="스크린샷 2023-03-25 오전 12 57 06" src="https://user-images.githubusercontent.com/85725033/227577549-57d13c54-76c4-4799-a459-8d403d7fe68a.png">

- JDK : Java Devlopement Kit
  - JDK는 자바 애플리케이션을 개발하기 위한 환경을 지원한다. 
  - JRE를 포함할 뿐만 아니라 컴파일러(javac), javadoc, jar 등 개발에 필요한 도구들을 포함하고 있다. 

- JRE : Java Runtime Environment로 JRE는 자바 실행환경(Java Runtime Environment)의 약자이다.
  - JVM 뿐만 아니라 Java binaries, Java 클래스 라이브러리 등을 포함하고 있어 Java 프로그램 실행을 지원한다. 
  - 컴파일러나 디버거(Debugger) 등의 도구는 포함하지 않는다. 따라서 자바 프로그램을 개발하는 것이 아니라 실행하기만 원한다면 JRE를 설치한다.

  
## 참고자료

- https://docs.oracle.com/en/java/javase/11/gctuning/introduction-garbage-collection-tuning.html - GC 튜닝 가이드
- https://d2.naver.com/helloworld/1230 - naver d2, JVM Internal
- https://docs.oracgle.com/javase/specs/jvms/se8/html/jvms-5.html - class loader

## 각주

[^1]: A Java virtual machine (JVM) is a virtual machine that enables a computer to run Java programs as well as programs 
written in other languages that are also compiled to Java bytecode. (wiki) 

