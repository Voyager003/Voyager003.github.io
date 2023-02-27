---
layout  : wiki
title   : Configuration proxy 패턴 
summary : 
date    : 2023-02-27 11:59:34 +0900
updated : 2023-02-27 12:00:20 +0900
tag     : spring
resource: A2/BC8BEB-6237-413F-92E8-54227A973823
toc     : true
public  : true
parent  : [[/Spring]]
latex   : false
---
* TOC
{:toc}

## @Configuration과 @Bean

Spring은 일반적으로 Component Scan을 통한 자동 Bean을 등록하는 방법을 권장한다. 

이 때, 자동이 아닌 수동으로 Spring Bean을 Spring container에 등록해야하는 경우 @Bean 애노테이션으로 직접 Bean을 등록한다.

수동으로 Bean 등록 시, @Configuration 애노테이션을 함께 사용하는데 이유를 살펴보자. 

## @Configuration의 Proxy Pattern

먼저 [ProxyPattern](https://refactoring.guru/design-patterns/proxy)이 무엇인지 간단히 알아보자.

> 다른 객체에 대한 접근을 제어하기 위한 대리자 혹은 자리표시자(책임자)역할을 하는 객체를 제공한다.

프록시패턴은 객체에 대한 접근을 제어하는 대리자 역할을 하는데, 코드를 통해 @Configuration이 어떤 역할을 하는지 살펴보자.

```java
@Configuration
public class ConfigurationEx {

    @Bean
    public configTest configTest() {
        return new configTest();
    }

    @Bean
    public configOne configOne() {
        return new configOne(configTest());
    }

    @Bean
    public configTwo configTwo() {
        return new configTwo(configTest());
    }
}
```

ConfigurationEx 클래스에 @Configuration을 명시함으로써, Spring Bean을 등록하여 Spring Container에 의해 처리할 수 있게됐다.

순수 Java코드로 configTest, configOne, configTwo를 호출한다고하면, 호출횟수만큼 configTest 객체를 세 번 생성할 것이다.

Spring은 불필요한 객체 생성을 방지하고자 @Configuration이 명시된 클래스를 객체로 생성할 때, **CGLib 라이브러리**이 바이트 코드를 조작하여
Proxy Pattern을 적용한다.

@Configuration이 명시된 클래스에 등록된 Spring Bean은 여러 번 호출해도 항상 동일 객체를 반환하는 Singleton을 보장한다.

이 때, Java 객체가 아닌 ConfigurationEx$$EnhancerBySpringCGLIB$$... 과 같은 객체 접근을 대리하는 객체를 생성한다.

따라서 @Configuration 없이 @Bean으로만 등록한다면, 객체를 호출할 때마다 새로운 객체를 생성하여 Singleton을 보장하지 않을 뿐더러, 프록시 패턴을 적용하지 
않은 순수 객체를 반환한다. 

## proxyBeanMethods

수동 @Bean 등록 시, 의도적으로 Singleton이 아닌 매번 다른 객체가 생성되기를 원할 수도 있다.

```java
@Configuration(proxyBeanMethods = false)
public class ConfigurationEx {

    @Bean
    public configTest configTest() {
        return new configTest();
    }

    @Bean
    public configOne configOne() {
        return new configOne(configTest());
    }

    @Bean
    public configTwo configTwo() {
        return new configTwo(configTest());
    }
}
```

@Configuration에 proxyBeanMethods의 값을 false로 준다면, Singleton 객체가 아니라 메서드 호출마다 새로운 객체를 생성한다.

## CGLib 성능이슈?

![alt](https://gmoon92.github.io/md/img/aop/jdk-dynamic-proxy-and-cglib/cglib2.png)

기존 CGLib은 구현을 위해 패러미터가 없는 default 생성자가 필요하고, Proxy 메서드 호출 시, taget의 생성자가 2번 호출된다는 단점이 존재했다.

이러한 단점을 개선하기 위해 Spring 4.0부터 Objensis 라이브러리의 도움을 받아 문제점을 개선할 수 있었다. 

## 참고자료

- https://refactoring.guru/design-patterns/proxy - 프록시 패턴
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html#proxyBeanMethods()
- https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html - CGLib

