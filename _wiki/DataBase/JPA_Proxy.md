---
layout  : wiki
title   : JPA Hibernate의 Proxy
summary : 
date    : 2023-03-17 11:28:25 +0900
updated : 2023-03-17 15:57:09 +0900
tag     : jpa java
resource: 1C/D4B364-7FE6-4613-B247-D833391FBEED
toc     : true
public  : true
parent  : [[/DataBase]]
latex   : false
---
* TOC
{:toc}

## 개요

```java
// 회원과 팀을 함께 출력하는 경우
public void printUserAndTeam(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("회원 이름: " + member.getUsername());
}

// 회원만 출력하는 경우
public void printUser(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("회원 이름: " + member.getUsername());
}
```

entity 조회 시, member라는 entity만을 출력하고자 하는 경우, em.find()로 조회한다면, member와 연관된 Team이라는 entity까지 조회되는 일이 생긴다. 

실제 서비스같은 경우, 위의 예시처럼 두 entity만이 연관되어 있지않고 수 십개가 연결되어있을 것이다. 이 때 연관된 모든 entity들이 조회된다면 성능상의 문제가 발생
할 것이다.

JPA는 이러한 문제를 해결하기 위해, entity가 실제 사용되기 전까지 DB 조회를 지연하는 방법을 제공하는데 이를 Lazy Loading(지연로딩)이라 한다.

이 Lazy Loading을 사용하려면 실제 entity 객체 대신 DB 조회를 지현할 수 있는 가짜 객체를 이용하는데, JPA 구현체인 Hibernate가 Proxy 객체를 통해
Lazy Loading을 구현하도록 도와준다.

## JPA Proxy

<img width="594" alt="스크린샷 2023-03-17 오후 3 29 13" src="https://user-images.githubusercontent.com/85725033/225829624-8e94ddb7-c569-4aba-b550-7a7c8683c5aa.png">

em.find(entity)는 DB를 통해 실제 entity 객체를 조회하는 메서드인데, em.getReference() 메서드를 통해 DB 조회를 미루는 Proxy entity 객체를 조회할 수 있다.

만들어진 Proxy 객체는 Lazy Loading하고자 하는 class를 상속받아 만들어지며, 이 객체는 실제 객체의 target(참조값)을 보관하고 Proxy 객체를 호출하면 실제 객체 메서드를 호출하게 된다.

## Proxy 초기화

최초 Lazy Loading 시에는 target이 없기 때문에, 실제 객체가 사용될 때 DB를 조회하여 target을 채우게 되는데 이를 Proxy 객체를 **초기화**한다고 한다.

Proxy 초기화 과정을 살펴보면

<img width="793" alt="스크린샷 2023-03-17 오후 3 39 26" src="https://user-images.githubusercontent.com/85725033/225831299-666dcda6-38c5-400e-a62d-219b7e3a4cd6.png">

1) Proxy 객체에 member.getName()을 호출하여 data 조회

2) 실제 entity가 생성되있지 않다면 Persistence Context entity 생성을 요청

3) Persistence Context가 DB를 죄회하여 실제 entity를 생성

4) Proxy 객체는 생성된 실제 entity 객체의 참조를 target에 보관

5) Proxy 객체가 실제 entity 객체의 getName()을 호출하여 객체를 반환

이 때, Proxy 객체는 처음 사용할 때 한 번만 초기화되고, 실제 entity가 바뀌지 않는다.

또한 Persistence Context에 찾은 entity가 이미 있다면 DB를 조회할 필요가 없기때문에 em.getReference()를 호출하더라도 실제 entity를 반환하게 된다.

## 참고자료

- https://www.inflearn.com/course/ORM-JPA-Basic/dashboard - 김영한님의 강의
- https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/ - JPA proxy



