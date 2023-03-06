---
layout  : wiki
title   : JDBC 트랜잭션 처리
summary : 
date    : 2023-03-06 14:33:10 +0900
updated : 2023-03-06 16:59:08 +0900
tag     : java jdbc
resource: A6/927D26-89E9-4266-ADCA-691B0B21DB18
toc     : true
public  : true
parent  : [[/DataBase]]
latex   : false
---
* TOC
{:toc}

## Transaction

> 데이터베이스 관리 시스템 또는 유사한 시스템에서 상호작용의 단위 [^wiki]

A라는 사람이 B라는 사람에게 1,000원을 지급하고 B가 그 돈을 받은 경우, 이 거래 기록은 더 이상 작게 쪼갤 수가 없는 하나의 트랜잭션을 구성한다.
만약 A는 돈을 지불했으나 B는 돈을 받지 못했다면 그 거래는 성립되지 않는다. 이처럼 A가 돈을 지불하는 행위와 B가 돈을 받는 행위는 별개로 분리될 수 없으며 하나의
거래내역으로 처리되어야 하는 단일 거래이다. 

이런 거래의 최소 단위를 트랜잭션이라고 한다. 트랜잭션 처리가 정상적으로 완료된 경우 커밋(commit)을 하고, 오류가 발생할 경우 원래 상태대로 롤백(rollback)을 한다.

## JDBC의 Transcation flow

<img width="755" alt="스크린샷 2023-03-06 오후 4 05 24" src="https://user-images.githubusercontent.com/85725033/223041420-02fcacb0-cb5f-4502-92aa-0f2e679aff6a.png">

### DB connection

```java
private final DataSource datasource;

public MemberRepository(Datasource datasource){
    this.dataSource = datasource;
}

...

Connection con = null;

...
try {
    con.setAutoCommit(false); // 트랜잭션 시작
    con = getConnection();    
    
    ...
        
    con.commit(); // 성공시 커밋
} catch (Exception e) {
    con.rollback(); // 실패 시 롤백
}

```

client의 요청에 따라 DB와 connection 하는 과정으로 예시의 경우는 datasource로 connection을 획득한다.

### Statement 생성 및 SQL 전송

```java
public Member save(Member member) throws SQLException {
    
    // 실행할 SQL문
    String sql="insert into member(member_id, money) values(?, ?)";
}

...

PreparedStatement pstmt = null;

...

try {
    con = getConnection();
    pstmt = con.prepareStatement(sql);
    pstmt.setString(..);
    pstmt.setInt(..);
    pstmt.excuteXXX();
} catch {
        ...
}
```

Statement 클래스는 SQL문을 실행하는 역할을 한다.

Java는 SQL문을 이해할 수 없기 때문에, .prepareStatement를 통해 String으로 선언한 sql문을 Java가 이해할 수 있는 객체로 만든 뒤,
.excuteXX(excuteUpdate, excuteQuery 등..) 메서드로 sql문을 실행한다.

### Result 받기

```java
try {
    con = getConnection();
    pstmt = con.prepareStatement(sql);
    pstmt.setString(1, memberId);
    rs = pstmt.executeQuery();
    
    if (rs.next()) {
        Member member = new Member();
        member.setMemberId(rs.getString("member_id"));
        member.setMoney(rs.getInt("money"));
        
        return member;
        } else {
        throw new ...
} catch {
        ...
    }
}
```

ResultSet은 excute 메서드를 통해 query를 실행하면 ResultSet 타입으로 반환하여 결과값을 저장한다.

### DB Connection 해제

```java
catch (SQLException e) {
 rollback();
 throw e;
 } finally {
 close(con, pstmt, rs);
 }
```

트랜잭션을 마치면, DB와의 connection을 해제해야한다. 이는 리소스가 누수되는 것을 방지하기 위함이다.

close 메서드로 JDBC 리소스를 해제한다.

## 참고자료

- https://velog.io/@suyyeon/JDBC-Statement#-%EC%9E%90%EB%B0%94-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8%EC%97%90%EC%84%9C-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-%ED%98%B8%EC%B6%9C-%EC%88%9C%EC%84%9C-
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-1/dashboard - 김영한님의 강의

[^wiki]:https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4_%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98
