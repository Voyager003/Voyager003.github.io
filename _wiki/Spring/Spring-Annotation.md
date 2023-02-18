---
layout  : wiki
title   : 내가 보려고 만든 Spring Annotation
summary : 
date    : 2023-02-12 22:07:08 +0900
updated : 2023-02-18 14:22:23 +0900
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

Annotation은 클래스와 메서드에 추가하여 다양한 기능을 부여하는 역할을 한다.

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

- Spring의 Component Scan 기능이 @Component가 명시된 클래스를 찾아 자동으로 Spring Container에 Bean을 등록한다.
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
- 역시 @Component와 역할이 같다. (구체화된 역할)

### [@RestController](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html)

- @Controller + @ResponseBody를 같이 사용한 것과 같다.
- Spring에서 Controller 중 View로 응답하지 않는 controller를 의미한다.
- json, xml과 같은 data 반환을 주목적으로 view가 필요없는 API만 지원하는 서비스에서 사용한다.
- API로써 기능을하는 controller 메서드를 만들 때는 주로 return하는 값을 그대로 HTTP Body에 return하는 방식을 사용한다.
- HttpResponse로 바로 응답이 가능하다.
- Stereotype Annotation은 아니지만 Controller와 차이점을 확인하기 위해 작성했다.

## [Context Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/package-summary.html)

> JSR-250, component scanning, Java기반 메타데이터를 포함한 Application Context에 대한 annotation을 지원한다.

### [@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)

> 하나 이상의 class에 @Bean을 선언하고 실행 시, Bean definition 및 서비스 요청을 생성하기 위해 Spring Container에 처리될 수 있음을 나타낸다. 

- 임의의 class를 만들어 @Bean을 붙인다고 Spring Container에 등록되지 않는다.
- @Bean을 사용하는 class에 @Configuration을 명시하여 해당 class에서 Bean을 등록한다는 것을 명시하는 역할을 한다.
- Spring Container는 @Configuration이 명시된 class를 자동으로 Bean으로 등록하고 해당 클래스를 파싱하여 @Bean이 있는 메서드를 찾아 Bean을 생성한다.

### [@Bean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)

> Spring Container(Application Context)에 의해 관리될 Bean provider로써 메서드를 선언하는 데 사용한다.

- @Configuration과 함께 사용하여 수동으로 Spring Container에 Bean을 등록하는 방법이다.
- 1개 이상의 @BEan을 제공하는 class의 경우 @Configuration을 명시해줘야 Singleton이 보장된다.

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

### [@ComponentScan](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html)

> @Compoent와 같은 Stererotype Annotation가 명시된 class를 scan하여 Bean으로 등록한다.

## [Beans factory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/package-summary.html)

> annotation 기반 bean configuration을 지원하는 package이다.

### [@Autowired](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html)

> field, setter methods, constructor에 명시하며, object dependency(객체 종속성)을 주입한다.

- 무조건적인 객체에 대한 의존성을 주입한다.
- Controller에서 DAO나 Service에 관한 객체들을 주입 시킬 때 많이 사용한다.
- Type을 확인하고 찾지못하면 Name에 따라 주입한다.
- 이 때, Name으로 강제하고 싶다면 @Qualifier애 명시한다.

### [@Qualifier](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Qualifier.html)

- @Autowired와 같이 쓰이며, 같은 타입의 Bean 객체가 있을 때 해당 아이디를 적어 원하는 Bean이 주입될 수 있도록 한다.
- 같은 타입의 Bean이 두 개 이상이 존재하는 경우에 Spring이 어떤 Bean을 주입해야 할지 알 수 없어 Spring Container를 초기화하는 과정에서 예외를 발생시킨다.
- 이 때, @Qualifier를 @Autowired와 명시하여 정확히 어떤 Bean을 사용할지 지정하여 특정 의존 객체를 주입할 수 있도록 한다.

## [Bind Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/package-summary.html)

> request를 컨트롤러, 핸들러 메서드에 binding하기 위한 Annotation과 parameters(매개변수)를 method arguments에 binding한다.

### [@RequestMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html)

> 요청 처리 class의 메서드에 web request에 mapping하기 위한 annotation이다.

```java
@Controller                   
@RequestMapping("/item") // /user로 들어오는 요청을 모두 처리

public class ControllerEx {
    @RequestMapping(method = RequestMethod.GET)
    public String getUser(Model model) {
        //  GET method, /item 요청을 처리
    }
    @RequestMapping(method = RequestMethod.POST)
    public String addUser(Model model) {
        //  POST method, /item 요청을 처리
    }
    @RequestMapping(value = "/info", method = RequestMethod.GET)
    public String addUser(Model model) {
        //  /id/info 요청을 처리
    }
}
```

- Controller나 Controller의 method에 적용한다.
- Spring의 컨트롤러 혹은 그 메서드의 URI를 정의한다.
- 요청을 받는 형식인 GET, POST, PATCH, PUT, DELETE를 정의하기도 한다.
- 요청 형식 미지정시, 자동으로 GET으로 설정한다.
- @RequestMapping에 대한 모든 mapping 정보는 RequestMappingHandlerMapping, RequestMappingHandlerAdapter를 통해 지원한다.

### [@RequestBody](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestBody.html)

> 요청 data(JSON, XML)를 바로 Class나 model로 mapping한다.

- HTTP Body에 전달되는 data를 메서드의 인자와 매칭해 data를 받아서 처리할 수 있는 Annotation이다.
- 클라이언트가 보내는 HTTP 요청 본문(JSON 및 XML)을 Java object로 변환한다. 

```java
@Controller                   
@RequestMapping("/id")      
public class ControllerEx {
    
    @RequestMapping(method = RequestMethod.POST)
    public String addId(@RequestBody User user) {
        //  POST method, /id 요청을 처리
    }
}
```

### [@RequestParam](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html)

> 메서드 매개 변수가 웹 요청 매개 변수에 binding되어야 함을 명시하는 주석이다.

- URL에 전달되는 parameter를 메서드의 인자와 매칭해 파라미터를 받아서 처리할 수 있는 Annotation이다. 
- Json 형식의 Body를 **MessageConverter**를 통해 Java 객체로 변환한다.

```java
@Controller                  
@RequestMapping("/id")      
public class ControllerEx {
    
    @RequestMapping(method = RequestMethod.GET)
    public String getUser(@RequestParam String id, @RequestParam(name="xx") String age {

    }
}
```

### [@PathVariable](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html)

> 메서드 praremeter가 URI 템플릿 변수에 binding되어야 함을 명시하는 주석이다.

```java
@GetMapping("/user/{userId}")
public int listUsersInvoices(@PathVariable("userId") int user{
}
```

- method parameter 앞에 사용하여 해당 URL에서 {value}를 변수로 받아 온다.
- /user/{userId} URI가 전달될 때 {userId}는 @PathVariable(“userId”)로 받는다.

## 참고자료

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/index.html - Spring docs
- https://jojoldu.tistory.com/27 - Bean, Component 차이
- https://www.oreilly.com/library/view/spring-5-design/9781788299459/9a9a3413-3d2e-4aa4-8993-87c5219ce060.xhtml - 이미지
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1/dashboard - 김영한님의 강의


