---
layout  : wiki
title   : Spring Security, Vue js, JWT 토큰 기반 인증 방식으로 회원가입, 로그인 구현하기 (1)
summary : 
date    : 2023-07-28 10:31:39 +0900
updated : 2023-08-03 23:03:40 +0900
tag     : springsecurity spring jwt vuejs
resource: 6C/EAB720-5952-496D-9975-1E89FDCCE89B
toc     : true
public  : true
parent  : [[/spring]]
latex   : false
---
* TOC
{:toc}

## 개요

사용자(클라이언트)가 서버에 접근할 때, 사용자가 인증된 사용자인지 확인하는 방법은 여러가지가 있다.

전통적인 웹에서는 Cookie와 Session을 기반으로 사용자의 상태(State)를 추적하고 관리하는 방법을 사용한다.

다른 방법은 토큰(Token) 기반 인증 방식이다. RESTful API를 기반으로 구현된 서버는 사용자의 상태를 별도의 Session 서버를 통해 관리하지 않는다. 이는 RESTful API의 특징 중 하나인 무상태성(stateless)에 기인한 것으로 API 서버는 요청(Payload)에 따라 결과를 응답하는 것에 집중하게 된다.

토큰 기반 인증은 사용자의 인증 정보가 담긴 토큰이 서버가 아닌 사용자에게 있으므로 서버에 저장할 필요가 없다. 서버가 데이터를 유지하고 있으려면 그만큼 자원을 소비해야하지만, 토큰 기반 인증을 사용하게 된다면 클라이언트에서 인증 정보가 담긴 토큰을 생성하고 인증하기때문에 서버 입장에서는 사용자의 인증 정보를 저장하거나 유지하지 않아도 되기 때문에 무상태로 효율적인 검증을 진행할 수 있다.

그 밖에, 확장성과 무결성이 보장된다는 특징도 있지만 이 글의 핵심 목적이 아니기 때문에 따로 찾아보자.

JWT 토큰 방식은 서버의 자원을 소비하지 않아도 되고 확장에 용이한 장점이 있지만, 비밀 키가 노출되거나 토큰이 탈취당하면 생기는 보안취약점이 있는데 Spring Security가 제공하는 인증 및 인가관련 기능을 조합해 사용한다면 단점을 보완하는 효과를 낼 수 있다. 


## 토큰 기반 인증방식 과정

![alt](https://www.vaadata.com/blog/wp-content/uploads/2016/12/JWT_tokens_EN.png)

1) 먼저, 클라이언트(브라우저)가 아이디와 패스워드를 서버에 전달하면서 인증을 요청한다.

2) 서버는 전달받은 정보를 확인해 유효한 사용자인지 검증하고 JWT 토큰을 생성한다.

3) 서버가 클라이언트에게 토큰을 전달하고, 클라이언트가 토큰을 저장한다. (응답)

4) 인증이 필요한 API를 사용할 때 토큰을 함께 보낸다.

5) 서버는 토큰이 유효한지 검증한다.

6) 토큰이 유효하면 클라이언트가 요청한 내용을 처리한다.(응답)

## 기능 구현

### 전체적인 flow

1) 사용자가 회원가입을 한다.(권한이 필요 없음)

2) 서버에서 사용자가 입력한 정보를 DB에 저장한다.

3) 사용자가 회원가입한 이메일과 비밀번호로 로그인 요청을 보낸다.

4) 서버는 사용자가 요청한 이메일, 비밀번호를 DB에서 가져와 복호화 한 뒤에 정보가 일치하는지 확인한다.

5) 일치한다면, 사용자에게 액세스 토큰을 전달한다.

6) 사용자는 전송된 토큰을 로컬 스토리지에 저장한다.

7) 사용자는 서버에 요청을 보낼 때마다 헤더에 토큰을 포함시킨다.

8) 서버는 요청을 받을 때, 토큰이 유효한지 검사하고 유효하다면 API를 호출한다.

### 개발 환경

- Java 17, Spring Boot 3.1.0, Spring Security 6.1.0, gradle.7.6.1
- Vue js 3.0

Spring Security가 제공하는 Form Login을 사용하지 않고, frontend단을 Vue js로 구축하여 backend에서 JSON을 보내고 받는 REST API로만 구성한다.

## build.gradle 설정

```java
// build.gradle

// Spring Security
implementation 'org.springframework.boot:spring-boot-starter-security'
testImplementation 'org.springframework.security:spring-security-test'

// JWT Library
implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```

Spring Security와 JWT 라이브러리 의존성이 필요하다.

## User, Authority Entity 생성

```java
// User.java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "user_info")
public class User {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "user_email", nullable = false)
    private String email;

    @Column(name = "user_password", nullable = false)
    private String Password;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(name = "user_roles",
            joinColumns = @JoinColumn(name = "user_id"),
            inverseJoinColumns = @JoinColumn(name = "role_id"))
    private Set<Authority> roles = new HashSet<>();



    @Builder
    public User(String email, String password, Set<Authority> roles) {
        this.email = email;
        this.Password = password;
        this.roles = roles;
    }
}

// Authority.java
@Entity
@Getter
@Table(name = "authorities")
@NoArgsConstructor
public class Authority {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Enumerated(EnumType.STRING)
    private Role name;

    @Builder
    public Authority(Role name) {
        this.name = name;
    }
}

// Role.java
@Getter
public enum Role {
    ROLE_SELLER,
    ROLE_CUSTOMER
}
```

회원가입 시, 이메일과 패스워드, 역할을 입력받도록 할 것이다. 

이 때, 역할은 ROLE_SELLER(판매자), ROLE_CUSTOMER(구매자)로 구분하여 권한을 부여하도록 한다.

권한은 일반적으로 하나 이상을 가지기 때문에 Set에 권한을 담도록 했다. 

## Repository 구현

위에서 작성한 Entity에 Access하기 위한 Repository가 필요하다. 

Spring Data JPA를 사용하여 Repository를 생성해보자.

```java
// UserRepository.java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);
    Boolean existsByEmail(String email);
}

// AuthorityRepository.java
@Repository
public interface AuthorityRepository extends JpaRepository<Authority, Long> {

    Optional<Authority> findByName(Role role);
}
```

## UserDetails 및 UserDetailsService 구현

<img width="424" alt="스크린샷 2023-08-03 오전 12 54 42" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/ffcc3c25-e979-4184-b559-92fc46145bf1">

### UserDetails

Spring Security 인증 프로세스에서 중요한 역할을 하는 객체가 있는데, 바로 UserDetails와 GrantedAuthority이다.

**UserDetails**는 Security 내에서 사용자의 정보를 담는 인터페이스이며, **GrantedAuthority**는 사용자가 실행할 수 있는 작업을 정의한다.

애플리케이션에서 사용자가 누구인지 알리기 위해 UserDetails를 구현하고, UserDetails에 선언된 메서드를 알아보자.


```java
@AllArgsConstructor
@Getter
public class UserDetailsImpl implements UserDetails {

    private Long id;

    private String email;

    @JsonIgnore
    private String password;

    private Collection<? extends GrantedAuthority> authorities; 

    public static UserDetailsImpl build(User user) {
        List<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority(role.getName().name()))
                .collect(Collectors.toList());

        return new UserDetailsImpl(
                user.getId(),
                user.getEmail(),
                user.getPassword(),
                authorities);
    }

    // 애플리케이션 사용자가 수행할 수 있는 작업을 GrantedAuthority 인스턴스 컬렉션으로 반환
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() { 
        return authorities;
    }

    // 사용자 자격 증명을 반환(패스워드)
    @Override
    public String getPassword() {
        return password;
    }

    // 사용자 자격 증명을 반환(Username)
    @Override
    public String getUsername() {
        return email;
    }

    // 이하 4개 : 사용자 계정을 필요에 따라 활성/비활성화
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        UserDetailsImpl user = (UserDetailsImpl) o;
        return Objects.equals(id, user.id);
    }
}
```

getUsername()과 getPassword()는 사용자의 이름과 암호를 반환하는 메서드이다. 

예시는 username 필드를 만드는 대신, email을 사용하도록 했다. 반환된 username과 password는 애플리케이션 인증 과정에 사용되는 세부 정보이다.

이하 4개의 메서드는 사용자가 애플리케이션 resource에 접근할 수 있도록 권한을 부여하기 위한 메서드이다.

사용자가 작업을 수행할 권리가 있거나 없다고 말해 사용자가 가진 권리를 나타내는 것이 권한이다. getAuthorities()는 사용자에게 부여된 권한을 컬렉션으로 반환하는 메서드이다.

참고로 다음과 같이 UserDetails를 Entity에 implements하여 구현하는 경우도 있다. 

```java
public User implements UserDetails {
    ...
}
```
 
나의 경우, Entity 코드가 복잡해지는 이유와 User에는 JPA Entity의 책임만을 남겨 알아보기 쉽도록 UserDetails를 따로 구현했다.

### UserDetailsService

UserDetails를 구현해 Spring Security가 이해할 수 있는 사용자를 기술했다. 이제 UserDetailsService로 인증 프로세스가 사용자 관리를 위임하도록 해보자.

UserDetailsService 인터페이스의 형태는 다음과 같다.

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

loadUserByUsername()은 주어진 사용자 이름을 가진 사용자의 세부 정보를 얻는 메서드이다.

메서드가 반환하는 사용자는 UserDetails에서 기술한 사용자의 정보로 존재하지않다면 UsernameNotFoundException을 던진다. 작성한 UserDetails로 구현체를 만들어보자. 

```java
@Component
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() ->
                        new UsernameNotFoundException("Email Not Found with email: " + email));
        return UserDetailsImpl.build(user);
    }
}
```

UserRepository에서 email을 통해 사용자의 정보를 조회하고, 존재한다면 User Entity를 불러와 build() 메서드를 통해 User 형식의 인스턴스를 래핑하고 UserUserDetails 인스턴스를 빌드하여 반환한다.

## 요청 필터링 및 내부 동작과정

Spring Security는 Servlet Filter를 기반으로 Controller에 요청이 도착하기 이전 혹은 사용자에게 response가 전달되기 전에 로직을 처리한다.

클라이언트가 애플리케이션에 요청을 전송하고, Servlet Container는 Servlet과 Filter로 구성된 FilterChain을 만들어 요청 URI path 기반으로 HttpServletRequest를 처리하는 것이다.

Jwt Token을 생성하고 검증하는 로직을 처리하는 Filter를 만들어보자.

```java
public class AuthTokenFilter extends OncePerRequestFilter {

    @Autowired private JwtUtils jwtUtils;

    @Autowired private UserDetailsServiceImpl userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        try {
            String jwt = parseJwt(request);
            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {

                String username = jwtUtils.getUserNameFromJwtToken(jwt);
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            throw new IllegalStateException("Cannot set user authentication");
        }
        filterChain.doFilter(request, response);
    }

    // Authorization Header에서 Token을 추출
    private String parseJwt(HttpServletRequest request) {
        String headerAuth = request.getHeader("Authorization");

        if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
            return headerAuth.substring(7);
        }
        return null;
    }
}
```

먼저 Filter는 OncePerRequestFilter를 상속받아 doFilterInternal()를 오버라이드한다. 설명을 살펴보면

> Filter base class that aims to guarantee a single execution per request dispatch, on any servlet container. It provides a doFilterInternal method with HttpServletRequest and HttpServletResponse arguments.

요청 디스패치 당 단일 실행을 보장한다고 적혀있다. 무엇을 의미하는 걸까?

일반적인 Servlet Filter는 HTTP 요청 및 응답을 가로채 처리하는 역할을 하는데, 이 Filter는 Servlet Container에 등록되어 모든 요청에 대해 실행될 가능성이 있다.

즉 같은 요청에 대해 여러 번의 Filter가 실행되어 중복적인 작업이 발생할 수 있다.

Spring Security는 이런 문제를 해결하기 위해 같은 요청에 대해서는 한 번만 실행하도록 보장하는 OncePerRequeistFilter를 제공하고, 이를 사용하도록 권장된다.

다시 코드로 돌아와 인터페이스가 제공하는 doFilterInternal이 수행하는 작업을 보자.

parseJwt()가 Authorization Header에서 접두사인 Bearer를 제거하여 JWT를 추출하고, 요청이 JWT 토큰을 포함한다면 유효성을 검증하고, username을 분석한다.

### SecurityContext

<img width="502" alt="스크린샷 2023-08-03 오후 4 42 45" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/ba3da1dd-63aa-4ba1-9a58-dcadaa725db7">

username에서 UserDetails를 가져와 인증 객체를 만들고 이를 SecurityContextHolder에 저장하게 된다. 이를 저장하는 이유가 무엇일까?

인증 프로세스가 끝나면 현재 인증된 사용자의 이름 혹은 권한을 참조해야하는 Entity에 대한 세부 정보가 필요할 가능성이 있다. 인증 프로세스가 완료된 후에도 이 정보에 접근하기 위해 Authenctication 객체를 저장하는데, 이 인스턴스를 **보안 컨텍스트**라한다. Spring Security에서는 이 보안 컨텍스트를 setAuthentication() 메서드로 SpringContextHolder에 저장한다.

컨텍스트에 담길 수 있는 구현체는 Authentication을 구현하고 있어야 하며, 코드에서 사용한 UsernamePasswordAuthenticationToken 역시 Authentication의 구현체이다.

저장한 이후에 인증된 주체(principal) 정보를 얻어야 한다면 ContextHolder에 접근하는 getContext() 메서드로 접근할 수 있게된다.

## JWT 토큰 생성

이제 JWT 토큰을 생성하는 클래스를 구현해보자.

```java
// application.yml
jwt:
  secretKey: eed49feca6c39f970a2cb61eadabb6cc81448505fde4c6a7ae942a67dcd45015 // 256bit
  expiration: 86400000 // 1일
```

yml파일에 secretKey와 만료 시간(ms)을 설정해야한다. 이 때 secretKey는 HMAC with SHA-256 알고리즘을 사용하기 위해 256bit 이상의 키를 사용해야한다. 충분한 길이의 무작위 Key를 입력해주면 된다.

여기서 HMAC은 HashBased Message Authentication Code의 약어로, 원본 메시지가 변하면 그 해시값도 변하는 Hashing의 특징을 이용해 메시지의 변조 여부를 확인하여 무결성을 제공하는 알고리즘이다.

```java
@Getter
@ConfigurationProperties("jwt")
@AllArgsConstructor
public class JwtProperties {

    private String secretKey;
    private int expiration;
}
```

이제 secretKey와 만료시간을 외부설정을 사용해서 접근하도록 하겠다. 외부 설정은 @Value와 @ConfigurationProperties를 사용하는 법이 있는데, 후자를 사용했다.

```java
@Component
@RequiredArgsConstructor
public class JwtUtils {

    private final JwtProperties jwtProperties;

    private Key key;
    
    @PostConstruct
    protected void init() {
        byte[] keyBytes = jwtProperties.getSecretKey().getBytes(StandardCharsets.UTF_8);
        this.key = Keys.hmacShaKeyFor(keyBytes);
    }

    public String issueJwtToken(Authentication authentication) {
        return Jwts.builder()
                .setHeaderParam(Header.TYPE, Header.JWT_TYPE)
                .setSubject(authentication.getName())
                .setIssuedAt(new Date())
                .setExpiration(new Date((new Date()).getTime() + jwtProperties.getExpiration()))
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }


    public String getUserNameFromJwtToken(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(key).build()
                .parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parse(authToken);
            return true;
        } catch (MalformedJwtException e) {
            return false;
        } catch (ExpiredJwtException e) {
            return false;
        } catch (UnsupportedJwtException e) {
            return false;
        } catch (IllegalArgumentException e) {
            return false;
        }
    }
}
```

먼저 @PostConstruct로 init()을 호출해 Key를 생성한다. 이유는 JWT 발행 시 서버가 재부팅되지 않는 한 유효한데, 클래스가 로드될 때 인스턴스가 초기화되어 새 임의 키를 생성하기 때문에 서명이 일치하지 않는다는 'jwt signature does not match locally computed signature' 에러를 볼 수 있을 것이다.

따라서 의존성 주입이 이루어진 후에 JWT를 발행하여 Key를 생성하도록 했다.

issueJwtToken()은 JWT를 생성하는 메서드이다. 인증 정보를 받아 JWT의 Header, Subject, 발급일과 만료일을 설정하고 서명하여 JWT 토큰을 문자열로 반환한다.

getUserNameFromJwtToken()은 JWT 토큰에서 subject를 추출해 반환한다. validateJwtToken()은 JWT 토큰의 유효성을 검증하는 메서드로 복호화 과정에서 에러 발생 시, 유효하지 않다고 판단한다.

## 인증 예외 처리

```java
@Component
public class AuthEntryPointJwt implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {

        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        final Map<String, Object> body = new HashMap<>();
        body.put("status", HttpServletResponse.SC_UNAUTHORIZED);
        body.put("error", "Unauthorized");
        body.put("message", authException.getMessage());
        body.put("path", request.getServletPath());

        final ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(response.getOutputStream(), body);
    }
}
```

AuthenticationEntryPoint 인터페이스를 구현한 AuthEntryPointJwt는 권한에 대한 인증 실패 혹은 인증되지않은 사용자가 보호된 리소스에 접근하려고할 때 호출되는 메서드를 정의한다.

commence()는 인증 오류 발생 시 호출되며, 위 코드에서는 응답 헤더를 컨텐츠타입을 JSON, 응답 코드를 401(UNAUTORIZED)로 설정하고, 인증 오류에 대한 정보를 JSON 형식을 response body에 담는다.

이를 jackson 라이브러리의 ObjectMapper를 사용해 변환된 JSON을 getOutputStream()을 통해 클라이언트에게 전달하게 된다.

## SecurityConfig 설정

```java
@Configuration
@RequiredArgsConstructor
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    private final UserDetailsServiceImpl userDetailsService;
    private final AuthEntryPointJwt unauthorizedHandler;

    @Bean
    public AuthTokenFilter authenticationJwtTokenFilter() {
        return new AuthTokenFilter();
    }

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManagerBean(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
                .exceptionHandling(exception -> exception
                        .authenticationEntryPoint(unauthorizedHandler))
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .cors(Customizer.withDefaults())
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers( "/","/login", "/signup").permitAll()
                        .requestMatchers("/assets/**","/favicon.ico", "/index.html").permitAll()
                        .requestMatchers("/api/test/**").permitAll()
                        .anyRequest().authenticated())
                .authenticationProvider(authenticationProvider())
                .addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

위에서 설정한 Filter와 인증 정보를 Spring Security에서 사용할 수 있도록 Config 파일을 작성해야 한다.

먼저 애노테이션을 살펴보면 @EnableWebSecurity와 @EnableMethodSecurity가 있다.

@EnableWebSecurity의 역할은 Spring Security를 활성화하고 웹 보안 설정을 구성하는데 사용되며, @EnableMethodSecurity는 @PreAuthorize, @Secured 등 메서드 조건에 따라 접근을 제한할 수 있도록 한다.

Spring boot 2.7.3 이전에는 WebSecurityConfigurerAdapter를 상속받아 configure()를 오버라이딩하여 설정을 구성했지만, 2.7.3 이후에는 @Bean으로 등록하도록 수정됐다.

아까 작성했던 AuthTokenFilter를 Spring Bean에 등록하고, 인증 오류 발생 시 호출될 AuthEntryPointJwt와 PasswordEncoder 역시 Bean으로 등록하자.

### DaoAuthenticationProvider 

<img width="644" alt="스크린샷 2023-08-03 오후 10 51 44" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/8d6942fd-8977-42de-8fe1-bc4839edb397">

DaoAuthenticationProvider는 UserDetails 및 Password Encoder를 사용해 사용자 아이디와 암호를 인증하는 AuthenticationProvider의 구현체이다.

UserDetails에서 UserDetails를 조회하고, PasswordEncoder를 사용해 UserDetails의 암호를 확인한다. 이 때 인증이 성공하여 반환되는 인증은 UsernamePasswordAuthenticationToken이며 UserDetails의 정보를 가진다. 이는 인증 필터에 의해 최종적으로 SecurityContextHolder에 설정된다.

### AuthenticationManager

<img width="634" alt="스크린샷 2023-08-04 오후 12 02 26" src="https://github.com/Voyager003/toy-shoppingmall/assets/85725033/7b29553d-a422-46f8-82cf-0c94e6539f9d">

AuthenticationManager는 Filter로부터 인증 처리를 지시받는 첫 번째 클래스이다.

이름 그대로 일종의 관리자 역할을 하며, AuthenticationFilter에 의해 AuthenticationManager가 동작하고 인증을 처리하면 SecurityContextHolder에 
Authentication 값이 세팅된다.

그림의 ProviderManager는 AuthenticatinoManager의 일반적인 구현체로 AuthenticationProvider 목록을 위임받는다.

위임받은 Provider는 인증 성공, 실패를 결정 역할을 한다.

### SecurityFilterChain

<img width="444" alt="스크린샷 2023-08-03 오후 10 58 19" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/7f86b00e-b06a-474a-b3e8-fffba9bc1426">

FilterChain은 filter가 작동하는 순서가 정의된 filter의 모음이다. 

Spring Security의 아키텍처의 filter는 일반적인 HTTP filter로 다른 HTTP filter와 마찬가지로 doFilter() 메서드를 오버라이딩해 논리를 구현한다. (AuthTokenFilter을 확인해보자.)

<img width="454" alt="스크린샷 2023-08-03 오후 11 05 31" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/d16cbf1d-bec7-4fc3-9325-1581953598df">

FilterChain은 애플리케이션을 구성하는 방법에 따라 더 길어지거나 짧아질 수 있다. 이 때, 순서 번호에 따라 요청에 필터가 적용되는 순서가 결정된다.

코드에서 적용한 Filter를 살펴보자.

- .csrf(AbstractHttpConfigurer::disable) 
    - CSRF 보호를 비활성화하는 필터이다. 

- .exceptionHandling
    - 인증 오류 발생 시 호출될 필터로 구현체로 AuthEntryPointJwt를 등록했다.

- .sessionManagement
    - 세션 관리를 설정하는 필터이다.
    - JWT 기반 인증 방식을 택하기때문에 세션 관리 정책을 사용하지 않도록 설정했다.
    - 세션 생성 정책을 STATELESS로 설정 시, 세션을 사용하지 않는다.

- .cors(Customizer.withDefaults())
    - CORS를 설정하는 필터이다.
    - Vue js로 작성한 클라이언트 코드와 통신하기위해, withDefaults()로 설정하여 기본 CORS 설정을 사용하도록 했다.

- .authorizeHttpRequests
    - 요청에 대한 접근 권한을 설정하는 필터이다.
    - requestMatchers()로 요청에 대한 접근 권한을 설정할 수 있다.
    - permitAll()로 설정 시, 모든 사용자가 접근할 수 있다.
    - anyRequest().authenticated()로 설정 시, permit()으로 허용한 요청을 제외한 모든 요청은 인증된 사용자만 접근할 수 있다.

- .authenticationProvider(authenticationProvider())
    - 인증 절차를 정의하는 필터이다.

- .addFilterBefore()
    - 기존 필터의 앞에 새로운 필터를 추가하기 위해 사용된다. 
    - 이렇게 추가된 필터는 해당 위치에 도달했을 때만 실행되며, 기존 필터들은 그 전에 실행된다.
    - 이로써 새로운 필터는 기존 필터의 처리 결과에 영향을 받아 작동하게 된다.

---

글이 너무 길어져서 다음 포스트에서 작성한 코드를 기반으로 회원가입과 로그인 기능을 실행하여 작동하는지 확인하도록 해보자.

## 참고자료

- Spring Security in Action
- https://docs.spring.io/spring-security/reference/index.html - 스프링 시큐리티 공식문서
- https://stackoverflow.com/questions/13152946/what-is-onceperrequestfilter - OncePerRequestFilter 
- https://stackoverflow.com/questions/42397484/jwt-signature-does-not-match-locally-computed-signature - jwt signature does not match locally computed signature 에러원인
- https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/dao-authentication-provider.html - DaoAuthenticationProvider


