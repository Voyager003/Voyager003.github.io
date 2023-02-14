---
layout  : wiki
title   : 내가 보려고 만든 Spring Annotation
summary : 
date    : 2023-02-12 22:07:08 +0900
updated : 2023-02-12 22:07:34 +0900
tag     : spring
resource: E9/C3812F-2E9A-4004-B48C-954A2C90841E
toc     : true
public  : true
parent  : [[/spring]]
latex   : false
---
* TOC
{:toc}

## Annotation(애노테이션)

Annotation(@)은 클래스와 메서드에 추가하여 다양한 기능을 부여하는 역할을 한다.

코드가 실행되는 중에 **Reflection**을 통해 추가 정보를 획득하여 기능을 실행한다.

Spring Framework는 Annotation을 활용하여 해당 클래스가 어떤 역할인지 정하거나, Bean을 주입하는 등 다양한 역할을 수행한다.

## [Stereotype Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/package-summary.html)

![alt](https://www.oreilly.com/api/v2/epubs/9781788299459/files/assets/6fd97786-73e6-4b3d-841d-8954bd1658b4.png)

> **Stererotype Annotation**은 아키텍처에서 타입, 메서드의 역할을 나타내는 Annotation이다.

### [@Component](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)

> Annotation이 명시된 class가 component임을 나타낸다.

```java
@Component
public class HelloController {
    
    public HelloController() {
        System.out.println("hello");
    }
}

// 사용자 지정 Bean id 등록
@Component("id")
public class HelloController{
    
    public HelloController() {
        System.out.println("hello");
    }
}
```

- Class에 @Component를 명시하여 Spring Bean에 등록한다.
- Bean 등록 시, Bean의 id를 따로 지정하지 않는다면 class의 이름의 첫 글자를 소문자로 바꾼 id가 Bean id로 사용된다. (위의 경우는 helloController)
- id를 지정하고 싶다면 아래의 코드와 같다.

### [@Repository](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Repository.html)

> Repository, Data Access Object 역할을 수행하는 class를 나타낸다.

- DB 데이터에 접근하기 위해 생성하는 DAO(Data Access Object) 관련 Bean을 자동등록 대상으로 만들 때 사용한다.
- @Component와 역할은 같지만(구체화된 역할), 추가적으로 검사 되지 않은 예외(DAO 메소드에서 발생)를 Spring DataAccessException으로 변환할 수 있다.

### [@Service](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)

> Business logic, 계산, 외부 API 호출과 같은 일부 서비스를 수행하기 위해 사용하는 class를 나타낸다.

- Service Layer에서 사용하는 Bean이라는 것을 구분하기위해 사용한다.
- 역시 @Component와 역할이 같다. (구체화된 역할)

### [@Controller](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)

> Spring Controller임을 나타낸다.

- Spring MVC 혹은 Spring Webflux의 controller를 식별하는데 사용한다.
- classpath scanning을 통해 구현 클래스를 자동으로 감지할 수도록 한다.
- view 반환을 주목적으로 하며, API로 사용하는 경우 @ResponseBody를 사용해 객체를 반환한다.
- 일반적으로 @RequestMapping을 기반으로 한 handler-method와 함께 사용한다.

### [@RestController](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html)

- @Controller + @ResponseBody를 같이 사용한 것과 같다.
- Spring에서 Controller 중 View로 응답하지 않는 controller를 의미한다.
- json, xml과 같은 data 반환을 주목적으로 view가 필요없는 API만 지원하는 서비스에서 사용한다.
- HttpResponse로 바로 응답이 가능하다.
- Stereotype Annotation은 아니지만 Controller와 차이점을 확인하기 위해 작성했다.

## [Context Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/package-summary.html)

> JSR-250, component scanning, Java기반 메타데이터를 포함한 Application Context에 대한 annotation을 지원한다.

### [@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)

> Spring Container에 해당 class가 Bean을 구성하고 있는 class임을 나타낸다.

- 


### [@Bean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)

> Spring Container(Application Context)에 의해 관리될 Bean provider로써 메서드를 선언하는 데 사용한다.

```java
@Configuration
public class Appconfig {    
    
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }   
}

@Configuration
public class Appconfig {

    @Bean(name="id")
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

- ArrayList를 Bean으로 등록하기 위해서는 별도로 해당 라이브러리 객체를 반환하는 메서드를 만들고 @Bean Annotation을 사용하면 된다.
- @Bean에 값을 지정하지 않는다면 메서드 이름을 camelCase로 변경한 것이 Bean id로 등록된다. (위의 경우 objectmapper)
- 아래 코드와 같이 name값을 지정한다면 원하는 id를 Bean으로 등록할 수 있다.
- @Component와 마찬가지로 목적이 명확하지 않은 Bean을 생성할 때 사용하는 애노테이션인데 차이점이 있다.
    - @Bean의 @Target({ElementType.METHOD, ElementType.ANNOTATIONTYPE}) 
    - @Component의 @Target({ElementType.TYPE})
    - @Bean은 개발자가 컨트롤이 불가능한 외부 라이브러리들을 등록하고자 할 때 사용된다.
    - 위의 경우, ObjectMapper class에 @Component를 선언할 수 없어 인스턴스를 생성하는 메서드를 만들고 해당 메서드에 @Bean을 선언했다.
    - 반대로 직접 컨트롤 가능한 class들의 경우 @Component를 사용한다.
    - 즉, 각자 선언할 수 있는 타입이 정해져있어 해당 용도 외에는 컴파일 에러를 발생시킨다.

### @ComponentScan

## 참고자료

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/index.html - Spring docs
- https://springframework.guru/spring-framework-annotations/ - Spring 부가설명
- https://jojoldu.tistory.com/27 - Bean, Component 차이
- https://www.oreilly.com/library/view/spring-5-design/9781788299459/9a9a3413-3d2e-4aa4-8993-87c5219ce060.xhtml - 이미지
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard - 김영한님의 강의


