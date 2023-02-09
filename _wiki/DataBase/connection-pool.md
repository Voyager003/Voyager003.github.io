---
layout  : wiki
title   : JDBC Connection pool(커넥션 풀)
summary : 
date    : 2023-02-09 23:23:55 +0900
updated : 2023-02-09 23:24:42 +0900
tag     : java jdbc db
resource: C0/5B43E9-03C9-48C6-B3FC-F943CC6F1E8F
toc     : true
public  : true
parent  : [[/DataBase]]
latex   : false
---
* TOC
{:toc}

## JDBC DriverManager

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc1SSfH%2FbtrOvc0bTQf%2FaU0eOAD6ya6nQS92Ueqro0%2Fimg.png)

Java에서는 JDBC 인터페이스를 각 DB벤더에 맞도록 구현해서, 개발자가 사용하는 벤더의 DB에 맞도록 구현하여 JDBC 드라이버를 제공한다. 

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbjObmM%2FbtrOwjLyLZ8%2F12KNkGojNOmTmIP6CQwIgK%2Fimg.png)

그 중, JDBC 인터페이스의 DriverManager는 각 벤더에 맞는 DB의 드라이버들을 관리하여 커넥션을 획득하는 기능을 제공한다.


DriverManager의 getConnection() 메서드를 통해 벤더의 드라이버에 커넥션을 요청하고, 라이브러리에 등록된 드라이버의 목록을 자동 인식하여
정보를 넘겨서 커넥션을 획득할 수 있는지 확인한다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FI8TzU%2FbtrOw6kkjAu%2FutzG56tkHzM5IHL1sycYH0%2Fimg.png)

API를 살펴보면, url(jdbc:Driver 종류://IP:포트번호/DB명), user(DB의 ID), password(DB의 비밀번호)를 받도록 되어있다.

각 벤더의 드라이버는 처리할 수 있다면 실제 DB에 연결하여 커넥션을 획득하고 이 커넥션 구현체가 클라이언트에 반환된다.
(이 때 항상 새로운 커넥션을 획득)

## Connection Pool

DB커넥션 과정을 요약하면 아래와 같다.

1) 애플리케이션 로직이 DB 드라이버를 통해 커넥션을 조회

2) DB의 드라이버는 DB와 TCP/IP의 3-way-handshake 과정을 통해 커넥션을 연결

3) 커넥션이 연결된다면 아이디와 패스워드 및 정보를 DB에 전달

4) 받은 정보를 통해 내부 인증을 완료 

5) 내부에 DB 세션을 생성하고 커넥션 생성이 완료되었다는 응답을 보냄

6) 커넥션 객체를 클라이언트에 반환

이러한 커넥션 과정을 클라이언트의 요청마다 새로 만드는 과정은 복잡하고 시간이 더 소요되기 때문에, 사용자 
응답 속도에 영향을 주게된다. 이러한 점을 해결하는 방법이 바로 Connection Pool로 커넥션을 미리 생성해두고 
Pool(단어 그대로 수영장 풀과 동일)에 커넥션을 풀어놓는 방법이 고안됐다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcDmZrC%2FbtrOwww7YTG%2FZ4DuSTaKVhfnOGFFvEO71K%2Fimg.png)

애플리케이션 시작 시점에 필요한 만큼 커넥션을 풀에 보관하는데, 이 때 풀에 들어있는 커넥션은 TCP/IP 과정을 거쳐 DB와 커넥션이 연결되어 있는 상태로 바로 
SQL을 DB에 전달할 수 있다. 

Spring Boot 2.0부터는 기본 커넥션 풀로 hikariCP를 제공하고 대부분 이를 사용한다. Thread가 커넥션을 요청하면 HikariCP의 경우 이전에 사용했던 커넥션이 
존재하는지 확인하고 이를 우선적으로 반환하게 되는 특징이 있다.

---

주의해야할 점은 'Connection Pool이 커지면 성능이 무조건 좋아지는가'라는 점이다.

결론부터 말하면 커넥션의 주체는 Thread이기 때문에 그렇지 않다.

Thread Pool이 Connection Pool보다 작다면 (Thread < Connection)

Thread Pool이 트랜잭션을 처리하는 스레드가 사용하는 커넥션 외에 남는 커넥션은 실질적으로 메모리만 차지하게 되고

Thread, Connection Pool이 모두 크기가 증가한다면 스레드의 증가로 더 많은 *Context Switching이 발생하게 된다.

이는 Connection Pool을 아무리 늘리더라도 Context Switiching으로 인한 오버헤드가 발생하기 때문에 성능적인 한계가 존재한다는 것이다.

결론적으로 WAS에서 커넥션 풀을 크게 설정하면 메모리 소모가 크지만 사용자가 대기하는 시간이 줄어들고, 
적게 설정한다면 대기 시간이 길어지기 때문에 서비스의 특징, 애플리케이션과 DB의 서버 스펙을 고려한 성능테스트를
통해 적절한 커넥션 풀의 수를 결정해야 한다. (hikariCP의 경우 default는 10개)

*Context Switching : 여러개의 프로세스가 실행되고 있을 때 기존에 실행되던 프로세스를 중단하고 다른 프로세스를 실행하는 것.

## DataSource 적용

위에서 DriverManager와 ConnectionPool라는 커넥션을 얻는 방법을 살펴봤다. 
애플리케이션 로직에서 DriverManager를 사용하다가 hikariCP와 같은 커넥션 풀을 사용하도록 변경한다면 
애플리케이션 로직의 코드를 변경해야한다.

이는 객체지향의 관점에서 좋지 않기 때문에(OCP 위반) Java에서는 
javax.sql.Datasource라는 인터페이스를 제공한다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb0coyB%2FbtrOyiEm03g%2FZMbTfJZKFAYaK4ONA2G4b1%2Fimg.png)

이 DataSource는 커넥션을 획득하는 방법을 추상화하는 인터페이스로 핵심 기능은 커넥션 조회이다.

대부분의 커넥션 풀은 DataSource의 인터페이스를 이미 구현해놓았기 때문에 hikariCP나 DBCP2의 코드에 의존하는 것이 아닌 DataSource 
인터페이스에만 의존하도록 애플리케이션 로직을 작성하면 된다.

이 때 위에서 설명한 DriverManager는 인터페이스를 사용하지 않는데 이를 DataSource에서 사용할 수 있도록 DriverManagerDatasource라는 클래스를 제공한다.

## Datasource를 이용한 Connection

### DriverManager

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FFCBBu%2FbtrOzs7AzxW%2F7bH7z8wk89kmfDiqLHC67K%2Fimg.png)
![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FyLm24%2FbtrOwNZGmYR%2FLmCkwkqVHCYVL9EzYs5Bj1%2Fimg.png)

DriverManagerDatasource를 이용해 dataSource 객체 생성 시 필요한 패러미터를 넘기고, getConnection()을 통해 호출했다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcnT1XZ%2FbtrOwOK3wBO%2FJzagvAil1fMIYIijHYxHy1%2Fimg.png)

DriverManager를 설명할 때, 커넥션을 호출하면 항상 새로운 커넥션을 획득한다고 했는데 connection=con0과 connection=con1을 통해 확인할 수 있다.

### Connection Pool

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbYFX3d%2FbtrOvbUz1Yq%2FRASyBNfMBD3SKx26IRB1Jk%2Fimg.png)

hikradiCP를 이용해 데이터 소스를 생성한 뒤에, 위와 동일하게 useDataSource로 커넥션을 호출했다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FD4nLB%2FbtrOxR762Kf%2FdXL7CZ9Axn0lu1pVVT1ie1%2Fimg.png)
![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fca6HyW%2FbtrOwxpnfw3%2Fsm8YJ4vQyIy71ZaiiF6EXk%2Fimg.png)

테스트의 결과를 살펴보면 Poolsize를 10으로 설정해놓고(total=10) con1과 con2(active=2)를 커넥션을 꺼내서 사용했기 때문에 idle=8(대기 상태)인 것을
확인할 수 있고, log를 살펴보면 hikariproyconnectino@~~이라는 인스턴스를 확인할 수 있다.

이 때 hikariCP는 별도의 스레드를 사용해서 커넥션 플에 커넥션을 채우는데, 이는 풀에 커넥션을 채우는 것이 
상대적으로 오래 걸리는 일이기 때문에(TCP/IP handshaking 과정) 별도의 스레드를 사용해 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않는다.


## 참고자료
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard - 김영한님의 강의


