---
layout  : wiki
title   : 트랜잭션 추상화와 동기화 
summary : 
date    : 2023-03-07 13:48:26 +0900
updated : 2023-03-07 16:54:53 +0900
tag     : java jdbc
resource: A5/267162-D30A-4FE8-82EA-D24048979795
toc     : true
public  : true
parent  : [[/DataBase]] 
latex   : false
---
* TOC
{:toc}

## 기존 JDBC기반 코드의 문제점

```java
@RequiredArgsConstructor
public class MemberService { 
    
    private final DataSource dataSource;
    private final MemberRepository memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        Connection con = dataSource.getConnection();
        try {
            con.setAutoCommit(false); //트랜잭션 시작
            businesslogic(...);
            con.commit(); //성공 시 commit
        } catch (Exception e) {
            con.rollback(); //실패 시 rollback
            throw new IllegalStateException(e);
        } finally {
            release(con);
        }
    }
    
    // 비즈니스 로직
    private void businesslobic(){
        ...
    }
}
```

- 문제점 1.
  - 트랜잭션 처리를 담당하는 코드가 JDBC 기술에 의존하고 있다.
  - 이는 추후, JPA, Mybatis와 같은 기술로 변경한다고 한다면, Service Layer의 코드를 모두 변경해야하는 점이 발생한다.
  - 이를 추상화된 인터페이스에 의존하도록 변경한다면, 사용 기술이 바뀌어도 구현체만 바꿔주면 Service Layer의 코드를 변경할 필요가 없어질 것이다.  

- 문제점 2.
  - 같은 트랜잭션을 유지하기 위해서 connection을 패러미터로 넘겨야한다.
  - 이 때, 동일한 기능도 트랜잭션을 유지하지 않아도 되는 경우와 트랜잭션용 기능을 분리할필요가 있다.

- 문제점 3.
  - 트랜잭션을 적용하는 try catch 문에서 반복되는 코드가 있다.

- 문제점 4.
    - 트랜잭션은 비즈니스 로직을 수행하는 Service Layer에서 시작된다.
    - Service Layer는 핵심 비즈니스 로직만을 수행하도록 작성되어야 유지보수가 용이하다.
    - 위의 경우, 트랜잭션을 처리하는 JDBC의 코드(java.sql.Connection, SQLException 등..)가 Service Layer에 포함되어있어 변경이 필요하다.

상기된 문제점을 개선해보자.

### 문제점 1 solution

<img width="605" alt="스크린샷 2023-03-07 오후 2 57 53" src="https://user-images.githubusercontent.com/85725033/223333391-194cf7a8-55f7-4502-95fb-3aba961572c1.png">

특정 트랜잭션 기술에 의존하는 것이 아닌, TxManger라는 추상화된 인터페이스에 의존하도록 한다. 

이를 통해, Service Layer의 코드를 변경하지 않고, 원하는 구현체를 의존성 주입을 통해 주입한다면 OCP 원칙을 지킬 수 있다.

Spring은 **PlatformTransactionManager** 인터페이스를 통해 추상화 기능을 제공한다.

### 문제점 2 solution

트랜잭션 시작과 끝까지 같은 DB connection을 유지해야 한다. 문제점이 제기된 코드는 같은 커넥션을 동기화하기 위해 패러미터로 connection을 전달했다. 
(Connection con = dataSource.getConnection()의 con을 전달)

<img width="603" alt="스크린샷 2023-03-07 오후 3 13 46" src="https://user-images.githubusercontent.com/85725033/223336074-548600f0-7d32-4afa-b708-e6fac72f87d8.png">

Spring은 ThreadLocal을 사용한 트랜잭션 동기화 매니저를 통해 Multi Thread 상황에도 안전하게 connection을 동기화할 수 있으며, 필요에 따라 connection을
획득할 수 있다. 

상기된 solution으로 code를 개선해보자.

#### Service

```java
@RequiredArgsConstructor
public class MemberService {

    private final PlatformTransactionManager transactionManager;
    private final MemberRepository memberRepository;

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        
        // 트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
        
        try {
            businesslogic(...);
            transactionManager.commit(status); // 성공 시 commit
        } catch (Exception e) {
            transactionManager.rollback(status); // 실패 시 rollback
            throw new IllegalStateException(e);
        } finally {
            release(con);
        }
    }
    
    // 비즈니스 로직
    private void businesslogic(){
        ...
    }
}
```

Service Layer에서 transactionManager.getTransaction() 메서드를 호출하여 트랜잭션을 시작하고, 트랜잭션 매니저(PlatformTransactionManager
transactionManager)를 인터페이스로 주입하고 JDBC 기술 구현체인 DatasourceTransactionManager 구현체를 주입받은 뒤, connection을 생성한다.

이 후, 비즈니스 로직을 실행하여 Repository를 호출한다. 비즈니스 로직을 모두 수행하면 트랜잭션을 종료해야 하는데, 이 때, 동기화 매니저를 통해 얻은 
동기화된 connection을 획득하면 DB에 트랜잭션을 commit하거나 rollback을 한다.

트랜잭션이 종료되고 전체 resource를 정리하는 과정으로 마무리된다. Connection pool을 고려하여 con.setAutocommit(true)로 되돌리고, close()를 호출하여
connection을 종료하고 Connection pool을 사용하는 경우 pool에 반환된다.

#### Repository

```java
public class MemberRepository {
    
    private final DataSource dataSource;
    
    public MemberRepositoryV3(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    public Member save(Member member) throws SQLException {
        String sql = "insert into ~";
        Connection con = null;
        PreparedStatement pstmt = null;
        
        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(...);
            pstmt.setInt(...);
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }
    
    public void update(String memberId, int money) throws SQLException {
        ...
    }

    private Connection getConnection() throws SQLException {
        Connection con = DataSourceUtils.getConnection(dataSource);
        log.info("get connection={} class={}", con, con.getClass());
        return con;
    }
    
    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        DataSourceUtils.releaseConnection(con, dataSource);
    }
}
```

- DatasourceUtils.getConnection() 메서드는 트랜잭션 동기화 매니저가 관리하고 있는 커넥션이 있다면 해당 connection을 반환하고, 없다면 새로운 connection을 
반환한다.

- DatasourceUtils.releaseConnection()은 con.close()를 사용하여 커넥션을 직접 닫아서 connection이 유지되지 않는 문제를 개선한 기능이다.
    - connection을 바로 닫는 것이 아닌 트랜잭션을 사용하기 위한 동기화된 connetion을 close하지 않고 유지시킨다.
    - 동기화 매니저가 관리하는 connection이 없다면, 그 때 connection을 close한다.


### 문제점 3 solution

```java
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
    business;ogic(...);
    transactionManager.commit(status); 
} catch (Exception e) {
    transactionManager.rollback(status); 
    throw new IllegalStateException(e);
}
```

트랜잭션을 획득하고 사용하는 로직에서 try, catch 문을 포함한 commit, rollback 코드가 반복되는 것을 확인할 수 있다.

이를 **template callback pattern**을 활용해 문제를 해결해보자. 

![스크린샷 2023-03-07 오후 4 28 17](https://user-images.githubusercontent.com/85725033/223353728-017ea801-535d-4e23-8a79-22e5920255d1.png)

Spring에서 제공하는 TransactionTemplate은 트랜잭션의 시작 종료 시점을 명시적으로 결정할 수 있는데, TransactionManager를 주입받아 트랜잭션 속성을
설정하고, execute 메서드에서 로직을 실행한다.

이를 코드에 적용해본다면

```java
@RequiredArgsConstructor
public class MemberService {

    private final TransactionTemplate txTemplate;
    private final MemberRepository memberRepository;
    
    public MemberService(PlatformTransactionManager transactionManager, MemberRepository memberRepository) {
        this.txTemplate = new TransactionTemplate(transactionManager);
        this.memberRepository = memberRepository;
    }

    public void accountTransfer(String fromId, String toId, int money) throws SQLException {
        
        txTemplate.excuteWithoutResult((status)->{
            try{
                businesslogic(...);
            }catch (SQLException e){
                throw new IllgealStateException(e);
            }
        });
    }
    
    // 비즈니스 로직
    private void businesslogic(){
        ...
    }
}
```

transaction template으로 transaction을 시작하도록 개선하여 commit, rollback 코드를 제거했다.

transactino template을 통해 비즈니스 로직이 정상 수행되면 commit, Exception이 발생하면 rollback한다. 

### 문제점 4 solution

Service Layer에 순수한 비즈니스 로직 코드만을 남겨야한다. 이를 Spring이 제공하는 AOP 기능을 사용하여 proxy 패턴을 적용해보자.

```java
@RequiredArgsConstructor
public class MemberService {
    
  private final MemberRepository memberRepository;

  @Transactional
  public void accountTransfer(String fromId, String toId, int money) throws SQLException {
    businesslogic(...);
  }

  private void bizLogic(String fromId, String toId, int money) throws SQLException{
    ...
  }
}
```

Spring은 @Transactional 애노테이션을 지원하는데 이는 트랜잭션 프록시가 트랜잭션을 처리하는 로직을 가져가고, 트랜잭션을 시작하여 실제 서비스를 대신
호출한다. (Spring Configuration에서 사용한 CGlib 방식과 같은 프록시 패턴)

애노테이션을 인식하여 트랜잭션 프록시를 적용하는 방식을 Declarative Transaction Management(선언적 트랜잭션 관리)라고 한다. 

## 참고자료

- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard - 김영한님의 강의 




