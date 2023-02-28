---
layout  : wiki
title   : 빌드와 배포
summary : 
date    : 2023-02-28 20:43:36 +0900
updated : 2023-02-28 23:31:28 +0900
tag     : java
resource: AD/FFE35F-C082-4791-B7BE-EC015E3E602D
toc     : true
public  : true
parent  : [[/terminology]]
latex   : false
---
* TOC
{:toc}

## Build(빌드)

> 소스코드 파일을 컴퓨터나 휴대폰에서 실행할 수 있는 독립(standalone) 소프트웨어 가공물로 변환하는 과정 및 그에 대한 결과물을 일컫는다.

Build 과정은 Compile과 Link를 포함한다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fd3s6bS%2FbtqNRJdufUI%2FfUfigVgPdDFR8rIAVMdQu1%2Fimg.png)

**Compile**은 인간이 읽고 이해할 수 있는 소스 코드를 기계가 읽을 수 있는 언어(binary)로 변환하는 것을 의미하며, 이러한 작업을 수행하는 프로그램을 
컴파일러라고 한다. Java의 경우 JDK에서 볼 수 있는 javac가 컴파일러이다. 

컴파일러는 특정 프로그래밍 언어로 만들어진 문서를 **목적코드**로 변환시키고, 컴파일을 통해 만들어진 파일을 Object 파일이라 한다.

프로젝트 내 여러 개의 소스 코드가 생성되고 A라는 소스 코드에서 B라는 소스 코드에 존재하는 함수(메서드)를 호출하는 경우가 존재하는데, 이때 A와 B 소스파일 각각 
컴파일만 하면 A가 B에 존재하는 함수를 찾질 못하기 때문에 호출할 수가 없다.

이 때, A와 B를 연결해주는 **Link**라는 작업이 필요하다. 여러 개로 분리된 소스 코드들을 컴파일한 결과물에서 최종 실행 가능한 파일을 만들기 위해 필요한 부분을
찾아서 연결해주는 작업을 링크라고 한다.

링크는 정적 링크(static link)와 동적 링크(dynamic link)가 있다.

- 정적 링크는 컴파일된 소스파일을 연결해서 실행 가능한 파일을 만드는 것
- 동적 링크는 프로그램 실행 도중 프로그램 외부에 존재하는 코드를 찾아서 연결하는 작업 
  - Java의 JVM이 프로그램 실행 도중 필요한 클래스를 찾아서 클래스 패스에 로드해주는 것이 예이다.

이와 같은 Build 과정을 거쳐 실행 가능한 소프트웨어 결과물을 얻어낸다.

## Deploy(배포)

**배포**는 최종 사용자에게 소프트웨어를 전달하는 과정이다. 

작성한 코드를 빌드하고, 빌드된 실행 가능한 파일을 사용자가 접근할 수 있는 환경에 배치한다면 배포가 된 것이다.

이 때 Java의 실행 가능한 파일은 .war와 .jar가 있으며, 이 파일을 WAS 서버에 올리면 배포이다.


## 참고자료

- https://st-lab.tistory.com/176
- https://ko.wikipedia.org/wiki/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4_%EB%B9%8C%EB%93%9C - build란?
- https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC - 컴파일러
