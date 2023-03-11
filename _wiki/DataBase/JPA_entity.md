---
layout  : wiki
title   : JPA 내부 동작
summary : 
date    : 2023-03-10 21:35:05 +0900
updated : 2023-03-11 16:51:30 +0900
tag     : java jpa 
resource: 89/E1FBC6-F58D-49F2-80EF-4041DB18003A
toc     : true
public  : true
parent  : [[/DataBase]]
latex   : false
---
* TOC
{:toc}

## JPA(Java Persistence API)

> 자바 퍼시스턴스 (API/Java Persistence API) 또는 자바 지속성 API(Java Persistence API, JPA)는 
> 자바 플랫폼 SE와 자바 플랫폼 EE를 사용하는 응용프로그램에서 관계형 데이터베이스의 관리를 표현하는 Java API이다.

Java 진영의 ORM 기술 표준으로 인터페이스를 제공하며 실제 구현체로 Hibernate, EcliplseLink 등이 있다.

## JPA의 내부 동작

<img width="734" alt="스크린샷 2023-03-11 오후 4 08 53" src="https://user-images.githubusercontent.com/85725033/224470582-620cdb30-430c-4ee1-b75d-e1c7feeb91d8.png">

### Entity(엔티티)

> Entity는 Persistence Context를 가진 객체로 DB의 테이블에 대응하는 클래스

```java
@Entity
public class Item {
//...
}
```

@Entity가 명시된 class는 JPA에서 관리하며 entity라 한다. DB에 item 테이블을
만들고, 이에 대응되는 Item.java 클래스를 만들어 @Entity 어노테이션을 붙이면 이 class가 entity가 되는 것이다.
class 자체나 생성한 인스턴스도 entity라고 부른다.

### Entity Manager Factory

> Entity manager instance를 관리하는 주체

애플리케이션 실행 시 한 개만 만들어지며 사용자로부터 요청이 오면 Entity manager를 생성한다.

SpringContainer는 Entity Manager instance를 사용 시, proxy 패턴을 이용하여 주입해줌으로서 Thread Safe를 보장한다.

### Entity Manager

> Persistence Context에 접근하여 Entity에 대한 DB 작업을 제공한다. 

내부적으로 DB Connection을 사용해 DB에 접근한다.

## Persistence Context(영속성 컨텍스트)

> Entity를 영구 저장하는 환경 

Entity Manager를 통해 Entity를 저장 혹은 조회 시, Entity manager가 영속성 컨텍스트에 Entity를 보관하고 관리한다.

### Entity Life Cycle

<img width="492" alt="스크린샷 2023-03-11 오후 4 28 42" src="https://user-images.githubusercontent.com/85725033/224471381-5942dc77-199c-435a-ae45-0030a9f4e062.png">

1) 비영속(new/transient) : Persistence Context와 관계가 없는 상태

```java
Item item = new Item();
```

entity 객체를 생성했지만, Persistence Context에 저장하지 않은 상태이다.

2) 영속(managed): Persistence Context에 저장된 상태

```java
em.persist(item);
```

Entity manager를 통해 entity를 Persistence Context에 저장한 상태로 관리 대상이 된다.

3) 준영속(detached): Persistence Context에 저장되었다가 분리된 상태

```java
em.detach(item);

em.clear();

em.close();
```

Persistence Context가 관리하던 entity로 더 이상 관리되지 않는다면 준영속 상태가 된다.

datach()메서드를 호출하여 준영속 상태로 만든다.

4) 삭제(removed): 삭제된 상태

```java
em.remove(item);
```

entity를 Persistence Context와 DB에서 삭제한다.

## 참고자료

- https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94_%ED%8D%BC%EC%8B%9C%EC%8A%A4%ED%84%B4%EC%8A%A4 
- https://velog.io/@neptunes032/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EB%9E%80