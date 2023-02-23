---
layout  : wiki
title   : Spring Web Layer와 용어 정리
summary : 
date    : 2023-02-22 16:46:06 +0900
updated : 2023-02-23 09:44:41 +0900
tag     : spring java
resource: AD/0E90B0-DEC3-49A0-BCBD-069D7EFCDC1F
toc     : true
public  : true
parent  : [[/spring]]
latex   : false
---
* TOC
{:toc}

## 개요

실무나 프로젝트에서 dto, dao이나 domain과 같은 패키지를 언제, 어떻게 나누는지에 대해 의문이 생겼는데, 인프런 강의 중
[영한님의 답변](https://www.inflearn.com/questions/16046/%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%ED%8F%B4%EB%8D%94-%EA%B5%AC%EC%A1%B0%EC%99%80-%EA%B0%95%EC%9D%98-%EC%9D%BC%EC%A0%95%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC-%EC%A7%88%EB%AC%B8%EC%9D%B4-%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4)을
보고 정리 차원에서 글을 작성했다.


<img width="648" alt="스크린샷 2023-02-22 오후 4 48 45" src="https://user-images.githubusercontent.com/85725033/220556294-d0957a27-581f-47d6-8997-fefb129a7d13.png">

## Spring Web Layer

먼저 Spring Web Layer(웹 계층)를 한 번 살펴보자.


<img width="783" alt="스크린샷 2023-02-22 오후 10 28 09" src="https://user-images.githubusercontent.com/85725033/220633657-b7329d1c-6563-44c9-a5f6-4111eba100a4.png">

### Presentation Layer(Web Layer)

- 브라우저 상(클라이언트)의 Web 클라이언트 요청 및 응답을 처리한다.
- Service Layer와 DAO 계층에서 발생하는 Exception을 처리한다.
- @Controller 클래스가 이 계층에 속한다.

### Service Layer(Business Layer)

- Transcation 관리
- Presentation Layer와 Data Access Layer를 연결하는 역할로 두 Layer가 직접적으로 통신하지 않도록 한다.
- @Service 구현 클래스가 이 계층에 속한다.

### Data Access Layer(Repository Layer)

- DataBase와 같이 데이터 저장소에 접근하는 영역이다.
- DB의 Table과 매칭되는 class로 Entity class라고도 한다.
- JPA를 사용한다면 Repository가 Data Access Object(DAO)역할을 한다고 볼 수 있다. (하단의 내용을 참고)

### DTO(VO)

- Data Transfer Object는 Layer 간의 data 교환을 위한 객체를 의미한다.

### Domain Object(Domain Model)

- domain이라고 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해하고 공유할 수 있도록 단순화 시킨 것을 말한다.

## 코드를 통해 살펴보기

<img width="775" alt="스크린샷 2023-02-22 오후 10 32 34" src="https://user-images.githubusercontent.com/85725033/220634604-43cd8977-5e6a-4666-872e-0ae7022403eb.png">

회원(member)을 등록, 조회하는 기능과 아직 데이터 저장소가 선정되지 않았다는 상황을 전제로 그림의 순서대로 각 Layer의 역할을 알아보자.

### DTO

```java
public class Member {

    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

먼저 domain 로직 중 하나인 DTO이다. data 교환을 위한 순수 Java객체로 data를 주고 받을 포맷이다. 

비즈니스 로직을 갖지않으며 순수 getter, setter 메서드를 가진다. 

회원가입 시스템의 경우, 회원 등록을 위한 id와 name을 가진다.

### Controller

```java
@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService){
        this.memberService = memberService;
    }
    
    // createMemberForm에서 Post로 넘긴 정보를 @PostMapping("/members/new")로 받아 처리
    @PostMapping("/members/new")
    public String create(MemberForm form){

        Member member = new Member();
        member.setName(form.getName());
        memberService.join(member);

        return "redirect:/";
    }

    @GetMapping("/members/new")
    public String createForm(){
        return "members/createMemberForm";
    }

    @GetMapping("/members")
    public String list(Model model){
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
}
```

Client의 요청을 받고 그 요청에 대해 업무를 수행하는 Service를 호출한다.

memberService 클래스의 join(회원가입)을 수행하고 결과를 가지고 화면을 생성하도록 View에 전달한다.

### Service

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }

    // 회원 가입
    public Long join(Member member) {

        validateDuplicateMember(member); 
        memberRepository.save(member);
        return member.getId();
    }
    
    //중복 회원 검증
    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
                .ifPresent(m -> {
                    throw new IllegalStateException("Already Existed member");
                });
    }
    
    // 회원 조회
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }
}
```

회원 가입(join), 회원 조회(findMembers) 등 비즈니스 단 기능들을 수행하며, 직접 DB에 접근하지 않는다.


### Repository

```java
@Repository
public class MemoryMemberRepository implements MemberRepository{
    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;
    
    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(),member);
        return member;
    }
    
    @Override
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream()
                .filter(member -> member.getName().equals(name))
                .findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }

}
```

직접 DB에 접근하여(예시의 경우 store HashMap) dto 객체를 저장, 저장, 삭제 등의 역할을 수행한다.

Service Layer 대신 DB에 접근하고 중간 다리 역할을 한다.

### Domain(Entity)

```java
@Entity 
public class Member {
    
    @Id
    @GenerateValue(strategy = GenerationType.IDENTITY)
    private final Long id;
    private final String name;

    public Member() {
    }

    public Member(Long id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

실제 DB table에 mapping되는 클래스로 실제 DB에 저장되는 내용들을 구현하는 class이다. 이를 기준으로 table이 생성되고 schema가 변경된다.

Entity는 데이터베이스 영속성(persistent)의 목적으로 사용되는 객체로 요청(Request)이나 응답(Response) 값을 전달하는 클래스로 사용하는 것은 설계 상 좋지 않다.

예시에서는 간단하게 Hashmap으로 DB를 구현했지만, MySQL과 같은 DB와 JPA를 사용한다고 가정한다면 코드는 위와 같은 형태로 구현할 수 있을 것이다.

## 혼동할 수 있는 point

### 도메인 로직, 비즈니스 로직?

<img width="807" alt="스크린샷 2023-02-23 오후 12 27 44" src="https://user-images.githubusercontent.com/85725033/220815178-46487e0a-a706-461d-a11c-d7271bf59ba2.png">

먼저 Problem space와 Solution space를 구분해보자.

**Problem space**는 domain, Problem domain, Core domain과 같은 용어로 표시되는데, 이 모든 용어들은 **소프트웨어가 해결하려는 문제, 구축목적**을
나타낸다. 

예를 들어, Toss같은 앱은 송금 및 주식 거래 등 서비스를 제공한다. 그렇다면 토스의 domain은 이러한 금융 관련 업무를 수행하는 송금, 주식거래 기능이 될 것이다.

**Solution space**는 비즈니스 로직, 도메인 로직과같은 용어가 포함된다. 이는 domain에 대한 솔루션을 나타낸다. 

다시 예를 들어보자. Toss라는 앱의 domain은 송금시스템이다. 그렇다면 도메인 로직 중 하나는 송금하려는 사용자의 잔액을 확인하고, 
잔액이 없다면 오류 메시지를 띄우는 메시지를 보여주는 코드가 도메인 로직이 될 것이다.

### 그렇다면 domain model(도메인 로직, 비즈니스 로직)과 Service Layer(서비스 로직)는 같은 일을 하는 것아닌가?

domain model에서 설명 했지만, 주요 차이는 의사결정(decision-maker)이 주된 척도가 된다. 

Toss 앱으로 친구에게 송금을 한다고 가정하고 단계로 나눠, 의사결정이 언제 이루어지는지 알아보자.

1) 계좌의 잔액을 확인한다. -> 도메인인 '송금'이 가능한지 의사결정을 한다.

2) 잔액이 없다면 오류 메시지를 띄운다. -> UI 메시지 출력

3) 송금을 마친 사용자의 잔액을 감소시킨다. ->  '송금' 서비스 수행

4) 사용자의 잔액을 Database에 저장한다. -> Persistence


**1.** 과 **3.** 단계를 보면 '송금'이라는 도메인에 대한 의사 결정을 하고있다. 따라서 도메인 로직이라고 할 수 있다.

반면 **2.** 와 **4.** 단계는 도메인 로직이 의사결정을 할 수 있도록 data를 제공하고, 그 결과를 DB나 UI에 반영한다. 따라서 서비스 로직이라고 할 수 있다.

도메인 로직에서 구현한 메서드는 주로 Service Layer에서 사용되며, 때문에 도메인 로직은 Service Layer가 아닌 domain model에 담는다고 한다.

의사 결정을 한다는 말이 굉장히 추상적이기 때문에, 실제 코드를 작성하면서 어느 Layer에 포함되어야 하는지 학습이 더 필요할 것 같다.

### domain(entity)과 DTO를 분리하는 이유?

- DTO가 일회성으로 데이터를 주고받는 용도로 사용되는 것과 다르게 Entity의 생명주기(Life Cycle)가 다르다.
- DB table과 mapping되는 Entity(domain)가 변경되면 여러 class에 영향을 미칠 것이다.
- 따라서 view와 통신하는 DTO class는 자주 변경되기 때문에 역할을 철저히 분리해야한다.

### DAO와 Repository 비교

DAO는 persistence 로직인 Entity Bean을 대체하기 위해 만들어진 개념이다. DAO가 비록 객체-지향적인 인터페이스를 제공하려는 의도를 가지고 있다고 하더라도 실제 
개발 시에는 하부의 persistence 메커니즘이 DB라는 사실을 숨기려고 하지 않는다. 
DAO의 인터페이스는 데이터베이스의 CRUD 쿼리와 1:1 매칭 되는 세밀한 단위의 오퍼레이션을 제공한다. 

반면 Repository는 메모리에 로드된 객체 컬렉션에 대한 집합 처리를 위한 인터페이스를 제공한다.
Repository에서 제공하는 하나의 오퍼레이션이 DAO의 여러 오퍼레이션에 매핑되는 것이 일반적이다.
따라서 하나의 REPOSITORY 내부에서 다수의 DAO를 호출하는 방식으로 REPOSITORY를 구현할 수 있다.

요약하면 Repotiroy는 entity 객체를 보관하고 관리하는 저장소이고, DAO는 데이터에 접근하도록 DB접근 관련 로직을 모아둔 객체이다. 
둘다 개념의 차이일뿐 실제로 개발할 때는 비슷하게 사용된다.

이 부분은 아직 JPA를 학습하지 않아서, 좀 더 공부한 뒤 정리해봐야겠다. 

## 참고자료

- https://www.petrikainulainen.net/software-development/design/understanding-spring-web-application-architecture-the-classic-way/
- https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/ - DTO 사용범위
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-mvc-1 - 김영한님의 강의
- https://enterprisecraftsmanship.com/posts/what-is-domain-logic/ - what is domain logic?
- https://stackoverflow.com/questions/21339657/whats-the-difference-between-service-layer-and-domain-model-layer
- https://www.baeldung.com/java-dao-vs-repository - DAO, Repository