---
layout  : wiki
title   : Spring Security, Vue js, JWT 토큰 기반 인증 방식으로 회원가입, 로그인 구현하기 (2)
summary : 
date    : 2023-08-03 23:56:07 +0900
updated : 2023-08-04 11:34:44 +0900
tag     : springsecurity spring jwt vuejs 
resource: D8/BBC1AF-F8C1-4976-A5ED-F71E68973C62
toc     : true
public  : true
parent  : [[/spring]]
latex   : false
---
* TOC
{:toc}

## 회원가입 기능 구현 

```
// SignupPage.vue
export default {
  name: "SignupPage",
  data() {
    return {
      email: "",
      password: "",
      role: "ROLE_SELLER",
      emailError: false,
      passwordError: false
    };
  },
  methods: {
    signup() {
      const user = {
        email: this.email,
        password: this.password,
        role: this.role
      };
      axios.post("/signup", JSON.stringify(user), {
        headers: {
          "Content-Type": "application/json"
        },
      })
        .then(response => {
          if(response.data.errorCode===400) {
            alert("이미 존재하는 이메일입니다.");
          } else {
            alert("회원가입이 완료되었습니다. 로그인 화면으로 이동합니다.");
            router.replace("/login");
          }
        });
    } ...
```

유효성 검증은 생략하고 설명하겠다. template에서 v-model으로 이메일, 비밀번호, 권한을 받아와 axios를 사용해 post 요청을 보내도록 했다. 

이 때, JSON.stringify() 함수로 JS 객체를 JSON 문자열로 변환하고, Content-Type을 applicatino/json으로 설정한다.

```java
// SignupRequest.java
@Getter
@NoArgsConstructor
public class SignupRequest {

    private String email;

    private String password;

    private Role role;
}

// UserSignupService.java
@Service
@RequiredArgsConstructor
public class UserSignupService {

    private final UserRepository userRepository;
    private final AuthorityRepository authorityRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    @Transactional
    public void Signup(SignupRequest request) throws Exception {

        Set<Authority> roles = new HashSet<>();
        Authority authority = Authority.builder().name(request.getRole()).build();
        authorityRepository.save(authority);
        roles.add(authority);

        userRepository.save(User.builder()
                .email(request.getEmail())
                .password(passwordEncoder.encode(request.getPassword()))
                .roles(roles)
                .build());
    }
}
```

SignupRequest라는 DTO를 만들어 이메일, 비밀번호, 역할을 받도록 했다. Signup()은 이 DTO를 받아 DB에 회원정보를 저장하게 된다.

builder 패턴을 통해 User Entity 인스턴스를 생성하고, 이메일과 BCryptPasswordEncoder로 암호화한 비밀번호 그리고 권한을 저장했다.

```java
@RestController
@RequiredArgsConstructor
public class UserSignupApi {

    private final UserSignupService userSignupService;

    @PostMapping("/signup")
    public ResponseEntity<?> signup(@RequestBody @Valid SignupRequest request) throws Exception {
        userSignupService.Signup(request);
        return ResponseEntity.ok().build();
    }
}
```

컨트롤러에서는 /signup path로 POST요청을 처리한다. 회원가입이 성공적으로 이루어지면 HTTP STATUS 200을 반환한다.

![스크린샷 2023-08-04 오전 12 39 40](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/77c83eec-8b8c-4303-a8cf-d7acd75ccdca)

회원가입에 성공했다면 H2 DB에도 이메일, 암호화된 비밀번호, 권한이 저장된 것을 확인할 수 있다.

## 로그인 기능 구현

```
data() {
    return {
      email: "",
      password: ""
    }
  },
  methods: {
    login() {
      const user = {
        email: this.email,
        password: this.password,
      };
      axios.post("/login", JSON.stringify(user), {
        headers: {
          "Content-Type": "application/json"
        },
      })
        .then(response => {
          if(response.data.errorCode===409) {
            alert("이메일 혹은 패스워드가 잘못 입력되었습니다.");
          } else {
            localStorage.setItem("accessToken", JSON.stringify(response.data));
            alert("로그인 되었습니다!");
            router.replace("/");
          }
        });
    }
  }
};
```

LoginPage 컴포넌트이다. 회원가입과 마찬가지로 template에서 v-model로 이메일과 비밀번호를 받아 유효성 검사에 통과한다면 JWT를 브라우저의 로컬 스토리지에 보관한다. 

JWT를 저장하는 방법은 쿠키에 저장하는 방식과 로컬 스토리지에 저장하는 방식이 있는데, 후자의 방법을 사용한다. 

```java
// LoginRequest.java
@Getter
public class LoginRequest {

    @NotEmpty
    private String email;

    @NotEmpty
    private String password;
}

// JwtResponse
@Getter
public class JwtResponse {

    private String token;
    private String type = "Bearer";
    private Long id;
    private String username;
    private String email;
    private List<String> roles;

    public JwtResponse(String accessToken, Long id, String username, String email, List<String> roles) {
        this.token = accessToken;
        this.id = id;
        this.username = username;
        this.email = email;
        this.roles = roles;
    }
}

// UserLoginAPi.java
@RestController
@RequiredArgsConstructor
public class UserLoginApi {

    private final AuthenticationManager authenticationManager;

    private final UserRepository userRepository;

    private final PasswordEncoder passwordEncoder;

    private final JwtUtils jwtUtils;
    
    @PostMapping("/login")
    public ResponseEntity<?> login(@Valid @RequestBody LoginRequest loginRequest) {

    Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.getEmail(), loginRequest.getPassword()));

    SecurityContextHolder.getContext().setAuthentication(authentication);
    String jwt = jwtUtils.issueJwtToken(authentication);

    UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();

    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.add("Authorization", "Bearer " + jwt);

    List<String> roles = userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList());

    return new ResponseEntity<>(new JwtResponse(jwt,
            userDetails.getId(),
            userDetails.getUsername(),
            userDetails.getEmail(),
            roles), httpHeaders, HttpStatus.OK);
    }
}
```

먼저 로그인 페이지에서 요청한 이메일과 비밀번호를 DTO 형식으로 받는다.

AuthenticationManager는 실제 Authentication 인스턴스를 만들고 인증을 처리하는 인터페이스이다.

이 인터페이스는 authenticate() 라는 메서드만을 가지며, 전달된 인증 인스턴스를 인증하려고 시도하고 성공한다면 부여된 권한을 포함한 인증 인스턴스를 반환한다. (returning a fully populated Authentication object (including granted authorities)

이전 글에서 설명했지만, Authentication라는 보안 컨텍스트는 SecurityContextHolder에 저장되는데, 코드에서는 Authentication의 구현체인 UsernamePasswordAuthenticationToken을 SecurityContextHolder에 저장한다.

이후 JWT를 생성하고 getPrincipal() 메서드로 인증된 사용자의 정보를 가져온다.

마지막으로 Response Header에 JWT를 담아서 반환하고, Body에 사용자의 정보를 담아 STATUS 200을 반환하도록 했다.

## JWT 토큰 확인

로그인을 마쳤으면, 작성한 JWTResponse 형식으로 정보가 전달이 됐는지 확인해보자.

'abc@naver.com'라는 이메일로 로그인을 요청하고, PostMan으로 Response Body에 있는 정보를 보면

![스크린샷 2023-08-04 오전 11 07 20](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/e36affe6-37f8-4f2b-b8fa-0ead0bbacb1e)

다음과 같은 정보를 확인할 수 있다. 반환받은 JWT를 Decode해보면 

<img width="570" alt="스크린샷 2023-08-04 오전 11 11 19" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/ac5953cf-3ce6-431f-af19-1053973ced4b">

다음과 같이 헤더와 페이로드에 작성했던 정보를 확인할 수 있다.

## 권한을 통한 페이지 접근

회원가입과 로그인 과정을 통해 JWT를 발급받았다. 권한을 이용해 페이지에 접근해보자. 그 이전에 작성했던 SecurityConfig를 확인해보자.

```java
@Configuration
@RequiredArgsConstructor
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

... 

public FilterChain filterChain(HttpSecurity http) {
    ... 
    .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers( "/","/login", "/signup").permitAll()
                        .requestMatchers("/assets/**","/favicon.ico", "/index.html").permitAll()
                        .requestMatchers("/api/test/**").permitAll()
                        .anyRequest().authenticated())
                        
                        ...
```

@EnableMethodSecurity 애노테이션은 해당 메서드의 조건에 따라 접근을 제한한다. 간단한 테스트용 컨트롤러를 만들어 확인해보자.

```java
@RestController
@RequestMapping("/api/test")
public class AccessController {

    @GetMapping("/public")
    public String publicAccess() {
        return "Public Content.";
    }

    @GetMapping("/seller")
    @PreAuthorize("hasRole('SELLER')")
    public String sellerAccess() {
        return "Seller Board.";
    }

    @GetMapping("/customer")
    @PreAuthorize("hasRole('CUSTOMER')")
    public String customerAccess() {
        return "Customer Board.";
    }
}
```

먼저 /api/test/public에 접근해보면 

![스크린샷 2023-08-04 오전 11 21 37](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/92682261-f23c-45d2-953c-860fd21ca2cc)

다음과 같이 STATUS 200을 반환하여 정상적으로 접근할 수 있다. requestMatchers에서 /api/test/**에 대한 접근은 permitAll()로 설정했기 때문이다.

이제 /api/test/seller에 접근해보자.

![스크린샷 2023-08-04 오전 11 23 18](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/eab1442a-fad2-4046-bed0-edf1348842e9)


권한이 없기 때문에, 이전에 정의했던 AuthEntryPointJwt 코드가 에러 메시지를 Response Body에 담아 반환한 것이다.

이제 회원가입 및 로그인을 마치고 권한을 가진 사용자가 접근하도록 해보자. 'test@naver.com'이라는 사용자로 로그인을 하고 역할은 ROLE_SELLER로 선택했다.

이후, GET 요청 Header의 Authorization에 Bearer 토큰을 넣어 /api/test/seller에 접근해보자.

![스크린샷 2023-08-04 오전 11 29 00](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/17e12b3d-e487-4205-b587-059855971bdd)


![스크린샷 2023-08-04 오전 11 29 44](https://github.com/Voyager003/Voyager003.github.io/assets/85725033/f435ae2e-fad6-484e-ac65-8f9964a93e63)

페이지에 성공적으로 접근했다! 요청 Header에 Token을 담아 보내면, 권한에 따라 사용자의 접근을 제한할 수 있다.

---

몇 주동안 삽질하면서 공부한 내용이라 머리에 더 잘 남는것 같다. 이제 응용하여 다른 기능도 구현해보자.

