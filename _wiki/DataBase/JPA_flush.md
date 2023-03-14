---
layout  : wiki
title   : JPA flush(플러시)
summary : 
date    : 2023-03-14 16:28:43 +0900
updated : 2023-03-14 23:39:12 +0900
tag     : jpa java
resource: B6/1C958A-D05E-4D11-8099-438CABF17C8D
toc     : true
public  : true
parent  : [[/DataBase]]
latex   : false
---
* TOC
{:toc}

## Flush

> Persistence Context의 변경 내용을 DB에 반영하는 것

트랜잭션 과정에서 commit을 호출하면 Persistence Context에 있는 객체가 DB에 반영된다. 

이 때, commit은 DB 트랜잭션 과정에서 DB변동 사항을 반영하는 것을 확정하는 메서드로 실제 query를 전송하는 메서드는 별개로 존재하는데, 
이를 수행하는 메서드가 flush이다. 

## 동작

<img width="914" alt="스크린샷 2023-03-14 오후 9 08 28" src="https://user-images.githubusercontent.com/85725033/224996608-168de474-beab-4aac-a645-87458e4bdb6a.png">

과정을 살펴보면

1) JPA는 entity를 Persistence Context에 보관할 때, 최초 상태를 복사하여 **스냅샷**을 저장한다.

2) Entity와 스냅샷을 비교하여 변경된 점을 감지한다.

3) 수정된 Entity를 쓰기 지연 SQL 저장소에 등록한다.

4) 쓰기 지연 SQL 저장소의 Query를 DB에 전송한다. (INSERT, UPDATE, DELETE..)

5) flush() 메소드 호출 뒤, commit() 메소드를 호출하여 DB에 반영한다.

## Flush가 호출되는 경우

Flush가 호출되는 경우를 살펴보자.

### Transaction commit

DB 변경 내역을 query로 전달하지 않고 트랜잭션만 commit한다면, DB에 데이터가 반영되지 않을 것이다.

그래서 트랜잭션은 commit 전에 flush를 호출하여 Persistence Context의 변경 내용을 DB에 반영해야한다.

JPA는 default로 트랜잭션 commit시 flush를 자동으로 호출한다. (Entitymanager.setFlushMode(FlushModeType.COMMIT))

### JPQL query 실행

```java
em.persist(member1);
em.persist(member2);

// jpql 실행
query = em.createQuery("select m from ...");
```

member1과 member2를 Persistence Context에 저장하고, jqpl을 실행하는 예이다.

이 때, query를 실행하고 flush를 호출하지 않는다면 DB 변경내역을 확인할 수 없을 것이다.

즉 query를 실행하기 직전에 Persistence Context를 flush해서 변경 내용을 DB에 반영해야한다. 

이러한 경우 트랜잭션 commit과 마찬가지로, JPA는 이를 자동으로 호출한다.

### 직접 수동으로 호출

```java
em.flush();
```

수동으로 호출하는 경우는 Test 코드 작성 외에 자주 사용하지 않는다.

## 참고자료

- https://www.inflearn.com/course/ORM-JPA-Basic/dashboard - 김영한님의 강의
