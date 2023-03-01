---
layout  : wiki
title   : jar, war
summary : 
date    : 2023-03-01 12:27:30 +0900
updated : 2023-03-01 23:04:04 +0900
tag     : java terminology
resource: 26/D31005-7D8C-41B7-BD5B-DFE1679DDA7A
toc     : true
public  : true
parent  : [[/terminology]]
latex   : false
---
* TOC
{:toc}

## 개요

IDE에서 Java소스 코드를 Build 했다면, 배포(Deploy)를 해야한다.

배포 전에, Build한 코드를 묶어 배포용 파일을 만들어주는 패키징 과정이 필요하다.

Java에서는 war와 jar 방식으로 패키징하는데 특징을 살펴보자.


## 배포 방식

<img width="545" alt="스크린샷 2023-03-01 오후 9 59 12" src="https://user-images.githubusercontent.com/85725033/222146167-279171aa-2359-4c19-a8d8-f21c56151bba.png">

### jar

> Java ARchive의 약자로, 엔터프라이즈 Java Bean (클래스 파일) 및 EJB 배치 디스크립터를 포함하는 EJB 모듈은 .jar 확장을 갖는 JAR 파일로 압축된다.

- JAVA 어플리케이션이 동작할 수 있도록 자바 프로젝트를 압축한 파일이다.
- Class (JAVA리소스, 속성 파일), 라이브러리 파일을 포함한다.
- JRE(JAVA Runtime Environment)만 있어도 실행 가능하다 (java -jar 프로젝트네임.jar)

Spring boot guide의 dafault는 jar이며, 내장 WAS(Spring boot의 Tomcat)을 써야하는 환경이라면 jar 방식을 이용한다.

### war

> Web application ARchive의 약자로 서블릿 클래스 파일, JSP 파일, 지원 파일, GIF 및 HTML 파일을 포함하는 
> 웹 모듈은 확장자가 .war (웹 아카이브) 인 JAR 파일로 패키지된다.

- Servlet / Jsp 컨테이너에 배치할 수 있는 웹 애플리케이션(Web Application) 압축파일 포맷이다.
- 웹 관련 자원을 포함한다. (JSP, Servlet, JAR, Class, XML, HTML, Javascript)
- 사전 정의된 구조를 사용한다. (WEB-INF, META-INF)
- 별도의 웹서버(WEB) or 웹 컨테이너(WAS) 필요하다.
- war도 jar파일의 일종으로 웹 애플리케이션 전체를 패키징 하기 위한 JAR 파일이다.

JSP로 화면을 구성하거나 외장 WAS를 써야하는 환경이라면 war 방식을 이용한다.

### ear

> Enterprise ARchive의 약자로 .jar 및 .war은 확장자가 .ear (enterprise archive)인 
> JAR 파일로 패키지되어 Application Server에 배치된다.

EAR 파일을 실행하려면 완전한 Java EE (Java Platform, Enterprise Edition) 또는 WebSphere 또는 JBoss와 같은 
Jakarta Enterprise Edition (EE) 호환 애플리케이션 서버가 필요하다.

## 파일 작성

JDK에는 jar.exe라는 특수 유틸리티가 포함되있으며, 이는 웹, 엔터프라이즈 및 Java 응용 프로그램을 해당 유형으로 패키징하고 압축한다
## 참고자료

- https://www.javatpoint.com/spring-boot-packaging - 패키징
- https://stackoverflow.com/questions/1594667/war-vs-ear-file - jar, war ear difference
- https://simuing.tistory.com/entry/JAVA-EAR-JAR-WAR-%EC%B0%A8%EC%9D%B4%EC%A0%90
- https://mongsil1025.github.io/til/server/warjar/
