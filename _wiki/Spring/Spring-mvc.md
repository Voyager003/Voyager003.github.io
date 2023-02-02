---
layout  : wiki
title   : Servlet과 JSP와 MVC패턴 등장 배경
summary : 
date    : 2023-01-31 16:39:50 +0900
updated : 2023-02-03 00:25:19 +0900
tag     : spring java
resource: 73/B2CCBC-0CB6-495F-9B41-E432E84D645D
toc     : true
public  : true
parent  : [[/Spring]]
latex   : false
---
* TOC
{:toc}

## MVC Pattern 및 Spring MVC 등장배경

![alt](https://developer.mozilla.org/ko/docs/Learn/Getting_started_with_the_web/How_the_Web_works/client-server.jpg)

Web의 동작 방식을 생각해보자. 간단하게 설명한다면 Client(브라우저)가 request하면 TCP/IP protocol을 통해 socket을 연결하고 HTTP request message를 읽고, HTTP message Body에 파싱한 뒤에
Business logic을 수행하고 HTTP response message를 생성하여 socker을 종료하는 과정을 거칠 것이다. 

이러한 복잡한 과정(TCP/IP, HTTP protocol..)에서 개발자들이 Business logic에 집중할 수 있도록, 하단에 설명될 개념들이 등장하게 되었는데 살펴보자.

## Servlet(서블릿)

그렇게 하여 등장한 기술인 **서블릿(Servlet)** 은 Java EE의 표준 API 중 하나로(javax.servlet.Servlet) 
HTML을 java 코드로 구현하여 문자열 stream("")을 통해 처리하고, 동적인 web 페이지를 작성할 수 있게됐다.

Servlet의 등장으로 HTTP Request message를 읽고 response를 만들어주는 역할을 하여 개발자들이 business logic에 집중할 수 있게 됐다.

### Web Server & WAS & Web Container

Servlet Container 설명 이전에 Web Server, Web Application Server(WAS), Web Container에 대해 혼동하고 있는 부분이있어 정리했다.

#### Web Server

**Web Server**는 단순히 요청에 대한 data를 정적 페이지(HTML, CSS)로 client로 보낸다. 초창기의 인터넷은 정적 데이터에 대한 수요가 많았기 때문에
WAS를 따로 나누지 않고 Web Server라는 개념을 통칭해서 사용했다.

하지만 기술의 발전과 사용자의 증가로 data를 수정하여 client에 보내줘야하는 경우가 생겨나면서, web 서버와 따로 서버를 나누게 되었는데
이를 Apllication Server라 한다. 주로 DB와 같이 사용되면서 Server side code를 통해 동적 데이터를 처리하게 됐다.

#### Web Application Server(WAS)

**Web Applcation Server(WAS)** 는 Application Server의 종류로 J2EE(Java 2 Enterprise Edition)의 스펙을 구현하여 
애플리케이션을 개발하기 위한 기술과 환경을 제공하며 Servlet이나 JSP로 작성된 애플리케이션을 실행하는 소프트웨어이다. (Apache Tomcat이 이에 해당한다.)

Web Server에서 동적인 data를 반환하려면 요청에 알맞는 프로그램(A)이 필요할 뿐만 아니라, 프로그램(A)에 적절히 넘겨 주는 중간자 역할을 
는 프로그램(B)이 필요한데 **CGI 프로그램(B)** 이 그 역할을 하게된다. 

하지만 CGI는 특별한 라이브러리나 도구가 아닌 Web server와 외부 프로그램 사이에서 정보를 주고 받는 방법, 즉 표준 스펙이자 Interface를 의미한다.
Java는 CGI와 유사한 방법으로 구현한 Servlet이라는 프로그램이 CGI의 역할을 하게된다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc1SJFH%2Fbtq2lpKGwSt%2F1xJiKtDckbXpeJOJSFaLMK%2Fimg.png)

WAS(Web Application Server)는 일부 Web Server 기능과 Web Container로 함께 구성된다. 앞단의 Web Server는 
HTTP 요청을 받아 Web Container로 넘겨준다. Web Container 이를 내부 프로그램 로직의 처리에 따라서 data를 만들어서
Web Server로 다시 전달한다. Java는 이를 Servlet을 통해서 처리하기 때문에 Servlet Container라고도 부른다.


#### Web Container(Servlet Container)

**Web Container(Servlet Container)** 는 Servlet을 만들었다고 해서 스스로 작동하지 않는다. 이 Servlet 객체의 생성, 소멸을 관리해주는 역할을 
Servlet Container가 하게된다.

- Servlet Container의 동작

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fl4kFd%2Fbtq2kst8KtB%2FXklp332Q59q7Eb1gsTKBa1%2Fimg.png)

크게 다음과 같은 흐름으로 진행되는데

1) Web Container는 요청을 받으면 Class loader를 통해 Servlet Class를 로드. (1번만)

2) Servlet Class가 로드되면 instance를 생성 (1번만 생성하고 Singleton으로 관리한다.)

3) instance를 생성한 뒤 init()을 호출하여 Servlet을 초기화한다. ( 1번만 초기화한다.)

4) Web Container는 요청이 들어올 때마다 thread를 통해 Service()를 호출한다.

5) Web Container가 종료되기 전에 Destroy()를 호출하여 특정 Servlet instance에 대한 resource 반납을 진행
 
resource를 효율적으로 사용하기 위해 매번 같은 로직을 거쳐 같은 결과를 산출하는 작업(doPost, doGet과 같은 HTTP 요청)은
init()에서 처리하는 것을 확인할 수 있다.

위의 과정과 같이 Servlet instance를 생성하고 구현체를 삭제하는 과정까지를 Servlet의 생명주기(Life Cycle)라고 한다.

---

전반적으로 Servlet에 대해 살펴봤는데, WAS가 Servlet Container를 포함하고 있어 WAS와 Servlet Container 두 개념이 헷갈리는데 이유는 아마 Apache Tomcat의 존재때문일 
것이다. 

Tomcat이 Servlet Container이면서도 WAS의 역할을 어느정도 하기 때문에 WAS로 간주하기도 하는데,
tomcat은 J2EE 스펙을 전부 구현하고 있지는 않기 때문에 Servlet container이지만 WAS라고 하기에는 약간 제한적이라고 말할 수 있다.

또한 WAS만 쓰면 되지 왜 Web Server를 따로 쓰느냐는 의문이 생길 수 있다. 

WAS와 Web Server는 둘 다 정적인 파일을 처리할 수가 있지만 WAS에서 응답을 처리할 때에는 외부 프로그램, Servlet Container 등의 존재 때문에 
부하가 많이 걸리게 되고 하나의 WAS에 너무 많은 요청이 몰리게 되면, 처리할 data가 많아져서 CPU에 부하가 오게 된다.

이 때문에 상대적으로 가벼운 Web Server에서
정적 요청을 처리하고 이를 여러 WAS로 구성을 하여 요청을 분산시킬 필요가 있는데, 이러한 요청을 분산시키는 기능을 Load Balancing이라 한다.

## JSP(Java Server Pages)

Servlet은 java 코드안에 html 태그를 삽입한다는 점때문에 코딩하는데 불편함을 유발했는데, 이를 위해 반대로 HTML안에 java코드를
삽입하게 된 JSP가 등장하게 됐다.

```html
<%@ page contentType="text/html;charset=UTF-8" %>
<% 
    String param1 = request.getParameter("param1");
    String param2 = request.getParameter("param2");
    int result = 0;
    if(param1 != null && param2 != null) {
        result = Integer.parseInt(param1) + Integer.parseInt(param2);
    }
%>

<html>
    <body>
        <form id=="f1" action="">
            <input type="text" name="param1" value="<%=param1%>">
            +  <input type="text" name="param2" value="<%=param2%>">
            =  <%=result%><br>
            <input type="submit" value="run">
        </form>
    </body>
</html>
```

JSP는 위와 같은 코드로 HTML 파일안에 <% %>같은 태그를 통해 java 코드를 삽입하여 web 서버에서 동적으로 web page를 생성하도록 
하지만 이러한 JSP는 logic 코드와 design이 jsp 한 파일에 모두 섞여있어 코드를 유지보수하는데 어려움을 겪게됐다.

Servlet Container 설명으로 글이 너무 길어져서 이어서 다음 글에 MVC 패턴과 함께 설명하겠다.

## 참고자료

- https://tecoble.techcourse.co.kr/post/2021-05-23-servlet-servletcontainer/ - Servlet
- https://tecoble.techcourse.co.kr/post/2021-05-24-apache-tomcat/ - Web server, was, container 차이
- https://lob-dev.tistory.com/entry/CGI-Servlet-Servlet-Container - Servlet flow
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard - 김영한님의 강의