---
layout  : wiki
title   : Spring MVC 패턴
summary : 
date    : 2023-02-03 10:03:15 +0900
updated : 2023-02-03 10:04:23 +0900
tag     : spring 
resource: 94/82E0CA-DA93-4842-8446-89282CACE321
toc     : true
public  : true
parent  : [[/Spring]]
latex   : false
---
* TOC
{:toc}

[이전 글](https://voyager003.github.io/wiki/Spring/Spring-mvc/)과 이어지는 내용입니다. 

## MVC pattern

하나의 Servlet이나 JSP만으로 비즈니스 로직과 View rendering까지 모두 처리하게 된다면, 하나의 코드에 너무 많은 역할이 과중되어 유지보수가 어려운 코드가 
될 것이다. 

이 때문에 Web Application의 유지보수에 용이하도록 MVC 패턴이 도입됐다.

![alt](https://developer.mozilla.org/en-US/docs/Glossary/MVC/model-view-controller-light-blue.png)

MVC 패턴의 핵심은 '관심사의 분리'로 크게 3가지 역할로 구분한다.

**Model**에서 data와 비즈니스 로직을 관리하고, **View**는 사용자(client)에게 결과를 화면으로 보여주고, **Controller**는 Model과 VIew 사이에서
Model이 어떻게 처리할지 알려주는 역할을 한다.

![alt](https://velog.velcdn.com/images%2Fjunhok82%2Fpost%2F950f13c6-8a86-42c7-8eaa-cf3eb5ea1d53%2Fimage.png)

Java는 화면을 rendering하는 JSP를 View template으로 사용하여 View의 역할을 하도록하고, Servlet을 Controller의 역할을 하도록 코드를 설계했다.

MVC Pattern의 도입으로 view와 controller의 역할을 분리한 것은 좋았지만, 요청받는 페이지가 늘어날수록 controller 내에 중복된 코드가 발생하게 됐다.

예를 들어 View로 이동하는 Forward나 View 주소를 가르키는 ViewPath도 중복되고, 응답을 보낼 필요도 없는 경우에 Servlet내 response 코드가 사용되지 않는
경우이다. 이러한 공통적인 처리를 위해 도입된 방법이 FrontController이다.

### FrontController

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbmIMRX%2Fbtq4rdbwCtY%2FO88ME95Pvv9svLKftxsaU0%2Fimg.png)

FrontController의 도입으로 

1) Front Controller가 URL mappring 정보에서 적합한 controller를 조회하여 호출하도록 처리하고

2) 기존에는 View로 이동하는 Forward 코드가 중복되었기때문에, 별도로 view를 처리하는 객체를 생성하여 컨트롤러에서 ModelAndView를 반환하면 
Front Controller에서 렌더링 하도록 처리

3) Front Controller를 제외한 나머지 controller는 Servlet 기술을 몰라도 동작될 수 있도록 Servlet 종속성 제거

4) View의 이름을 전부 명시하지 않고 논리 이름만 반환해도 되도록 처리 (ViewResolver)

위의 과정을 통해 Servlet 하나로 client의 요청을 받고 그에 맞는 하위 controller를 호출해주는 패턴이 도입됐다.

하지만 이 FrontController에도 맹점이 있었다. 예를 들어 A 개발자는 ModelView를 반환하는 controller를 사용하고 싶고, B 개발자는 view name을
반환하는 controller를 사용하고 싶어한다고 상황이 발생할 수도 있다. 이를 위해 여러 controller와 호환 가능하도록 front-controller와 
controller 사이에 adapter가 도입됐다.

### Adapter

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzcptA%2FbtrDQd7Wp7f%2FJyVFZckw3BN4vkUXkoVFBk%2Fimg.png)

Handler adapter 도입으로 다양한 종류의 controller(adapter)를 호출할 수 있게 됐다. 이를 통해

URI로 controller를 찾고 (HandlerMapping 과정), Handler mapping에서 찾은 handler(controller)를 실행하게 된다.

이러한 기능들을 Spring에서는 자동 등록하여 쉽게 사용할 수 있도록 하는데 살펴보자. 

## Spring MVC

![alt](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/images/mvc-contexts.gif)

Spring은 위에서 설명했던 Front-controller의 역할과 Adapter의 역할을 구현한 **Dispatcher Servlet**이 대신하게 된다.

### Dispatcher Servlet

**Dispatcher Servlet**은 HTTP protocol로 들어오는 모든 요청을 받아 적합한 controller에 위임해주는 front-controller의 역할을 한다.

글에서 설명은 안했지만 과거에는 모든 servlet을 URL 매핑을 위해 web.xml에 모두 등록해줘야했지만, dispatcher-servlet이 해당 application으로
들어오는 모든 요청을 handling하고 공통 작업을 처리하면서 편리하게 사용할 수 있게됐다. 이로써 개발자들은 controller를 구현만 해둔다면 알아서 적합한 
controller로 위임해주는 구조가 됐다.

#### Dispatcher Servlet 동작 방식

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbImFbg%2FbtrGzZMTuu2%2FCkY4MiKvl5ivUJPoc5I3zk%2Fimg.png)

Dispatcher Servlet은 크게 다음과 같은 과정으로 이루어진다.

1) client 요청을 Dispatcher Servlet에서 받기

Servlet context에서 filter를 지나 Spring context에서 Dispatcher Servlet에서 요청을 받는다.

2) 요청 정보를 통해 요청을 위임할 controller(hadler)를 찾기

Dispatcher Servlet은 요청 정보를 바탕으로, 요청을 처리할 controller를 찾고 해당 메소드를 호출한다.
이 때, HandlerMapping의 구현체 중 하나인 RequestMappingHandlerMapping가 controller를 찾는다.

해당 객체는 내부에서 HashMap으로 (요청 정보, 처리 대상)을 관리하고 있어서, 요청 정보를 Key로 사용하여 요청을 처리할 대상을 찾아온다. 
요청을 처리할 대상은 controller를 가지고 있는 HandlerMethod 객체이다. 그리고 찾아온 HandlerMethod를 HandlerMethodExecutionChain으로 감싸서 
반환하는데, controller 요청을 넘겨주기 전에 처리해야 하는 인터셉터 등을 포함하기 위함이다.

3) 요청을 controller로 위임할 핸들러 adapter를 찾아서 전달

Dispatcher Servlet은 컨트롤러로 요청을 직접 위임하는 것이 아니라 HandlerAdapter를 통해 어댑터 패턴을 적용해 controller로 요청을 위임한다.

4) 핸들러 어댑터가 컨트롤러로 요청을 위임

핸들러 어댑터가 컨트롤러로 요청을 넘기기 전에 공통적인 전처리 과정이 필요합니다. 요청에 매칭되는 인터셉터들도 실행을 시키고, 
@RequestParam, @RequestBody 등으로 파라미터를 준비하는 ArgumentResolver도 실행하는 등의 다양한 공통 작업들이 수행됩니다. 
이러한 전처리 작업들이 완료되면 파라미터 값들과 함께 컨트롤러로 요청을 위임합니다.

5) 비지니스 로직을 처리

6) controller가 반환값을 반환

비즈니스 로직이 처리된 후에는 컨트롤러가 반환값을 반환한다. 응답 데이터를 사용하는 경우에는 주로 ResponseEntity를 반환하게 되고, 
응답 페이지를 보여주는 경우라면 String으로 View의 이름을 반환할 수도 있다.

7) handler adapter가 반환값을 처리

handler adaplter는 controller부터 받은 반환값을 응답 처리기인 ReturnValueHandler가 후처리한 후에 Dispatcher Servlet으로 돌려준다. 
만약 컨트롤러가 ResponseEntity를 반환하면 HttpEntityMethodProcessor가 MessageConverter를 사용해 응답 객체를 
직렬화하고 응답 상태(HttpStatus)를 설정한다. 만약 컨트롤러가 View 이름을 반환하면 View를 반환하기 위한 준비 작업을 처리한다.

8) 서버의 응답을 client로 반환

Dispatcher Servlet을 통해 반환되는 응답은 다시 필터들을 거쳐 클라이언트에게 반환된다. 
이때 응답이 데이터라면 그대로 반환되지만, 응답이 화면이라면 View의 이름에 맞는 View를 찾아서 반환해주는 ViewResolver가 적절한 화면을 내려준다.

## 참고자료
- https://developer.mozilla.org/ko/docs/Glossary/MVC#%EC%9B%B9%EC%97%90%EC%84%9C%EC%9D%98_mvc - MVC 이미지
- https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/mvc.html - Spring docs
- https://mangkyu.tistory.com/18 - Dispatcher servlet (망나니 개발자님의 블로그)
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard - 김영한님의 강의