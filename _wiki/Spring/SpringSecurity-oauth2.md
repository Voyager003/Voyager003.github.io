---
layout  : wiki
title   : Spring Security와 OAuth2 작동 흐름
summary : 
date    : 2023-05-23 15:43:11 +0900
updated : 2023-05-24 20:37:12 +0900
tag     : spring springsecurity
resource: 99/56DD9C-E4FE-4333-8794-58A246BFBE95
toc     : true
public  : true
parent  : [[/Spring]]
latex   : false
---
* TOC
{:toc}

## Spring Security 흐름 이해하기

<img width="705" alt="스크린샷 2023-05-23 오후 4 49 11" src="https://github.com/Voyager003/PracticeCode/assets/85725033/42521998-37df-484f-8efc-f8fff2002cbf">

- Spring Security는 Servlet Filter와 이들로 구성된 Filterchain을 사용한다.
- SecurityContextPersistenceFilter부터 시작하여 FilterSecurityInterceptor까지 순서대로 필터를 거친다.
- 필터 실행 시, 화살표로 연결된 클래스를 거치면서 실행하며, 특정 필터를 제거하거나 필터 뒤에 커스텀 필터를 넣는 설정이 가능하다.
- 기본 폼 로그인 사용 시, 진행되는 흐름은 다음과 같다.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*62sgUdt_hB1ZCKNYzvJNaA.jpeg)

1) 사용자가 폼에 아이디, 패스워드 입력 시 HttpServletRequest에 아이디와 패스워드 정보가 전달된다. 이 때 AuthenticationFilter가 요청을 가로채 유효성 검사를 실행

2) 유효성 검사가 끝나면 구현체인 UsernamePasswordAuthenticationToken을 생성 (상태 : 미검증 Authentication)

3) AuthenticationManager의 구현체인 ProviderManager에게 생성한 토큰을 전달

4) AuthenticationManager는 등록된 AuthenticationProvider를 조회하여 인증을 요청

5) 실제 DB로부터 사용자 인증 정보를 가져오는 UserDetailsService를 호출하여 사용자 정보를 전달

6) 전달받은 사용자 정보를 통해 UserDetails DB에서 찾은 사용자 정보인 UserDetails 객체 생성

7) AuthenticationProvider는 UserDetails를 넘겨받고 입력 정보와 UserDetails의 정보를 비교해 인증 처리

8) 인증이 완료되면 사용자 정보를 담은 Authentication 객체를 반환 (상태 : 검증 Authentication)

9) 최초의 AuthenticationFilter에 Authentication이 반환됨

10) 검증된 Authentication 객체를 SecurityContext에 저장 

- 이 때 **6.** 에서 UserDetailsService와 UserDetails의 실질적 인증 과정은 사용자가 입력한 데이터와 UserDetailsService의 loadUserByUsername()이 반환하는 UserDetails 객체를 비교함으로써 동작한다.
- UserDetailsService와 UserDetails 구현을 어떻게 하느냐에 따라 인증 과정이 달라진다.

## OAuth2

OAuth(Openid Authentication)는 인증을 위한 개방형 표준 프로토콜로 Third-party 프로그램에 리소스 오너를 대신해 리소스 서버에서 제공하는 자원에 대한 접근을
위임하는 방법이다. 이해를 위한 용어 몇 가지를 짚고 넘어가자.

- 리소스 오너(Resource Owner) : 리소스 소유자로, 정보를 사용하도록 인증 서버에 허가하는 주체 즉 서비스를 이용하는 사용자이다.
- 리소스 서버(Resource Server) : 리소스 오너의 정보를 가지며, 정보를 보호하는 주체 구글, 네이버, 페이스북 등이 이에 해당한다.
- 인증 서버(Authorization Server) : 클라이언트에게 리소스 오너의 정보에 접근할 수 있는 토큰을 발급하는 애플리케이션
- 클라이언트 애플리케이션(Client Application) : 인증 서버에서 인증을 받고 리소스 오너의 리소스를 사용하는 주체 즉 개발자가 만든 서비스를 의미한다.

OAuth2를 이용한 권한 부여 코드 승인 방식(Authorization Code Grant)의 흐름은 다음과 같다.

<img width="991" alt="스크린샷 2023-05-23 오후 11 35 12" src="https://github.com/Voyager003/Voyager003.github.io/assets/85725033/e4f476fd-f3db-4f69-90a6-b5129ffd6e9b">

1. 클라이언트(애플리케이션)는 Authorization Server로 접근 권한을 요청
    - 이 때, 요청 패러미터에는 client_id, redirect_uri, response_type=code를 포함
2. 클라이언트로부터 접근 권한 요청을 받은 Authorization Server는 소셜 로그인을 할 수 있는 로그인 창을 띄운다.
3. 리소스 오너는 소셜 로그인 창을 통해 로그인을 진행
4. Authorization Server는 리소스 오너로부터 전달받은 데이터가 맞는지 여부를 판단하고 권한 승인 코드를 반환
5. 클라이언트는 권한 승인 코드를 통해 리소스 서버에 보호된 자원을 요청할 수 있는 Access Token을 요청
6. Access Token을 전달받은 클라이언트는 해당 토큰을 통해 리소스 서버에 필요한 요청을 보낸다.

### 내부 작동

1. frontend client에서 엔드포인트에서 요청을 보냄
  - http://localhost:8080/oauth2/authorize/{provider}?redirect_uri=<redirect_uri_after_login>
  - provider : google, facebook, github
  - redirect_uri : OAuth2 provider가 성공적으로 인증을 완료했을 때 redirect 할 URI를 지정 (이 때,OAuth2의 redirectUri 와는 다르다)

2. 엔드포인트로 인증 요청을 받으면, Spring Security의 OAuth2 클라이언트는 user를 provider가 제공하는 AuthorizationUrl로 redirect 시킨다.
   - Authorization request와 관련된 state는 authorizationRequestRepository 에 저장된다 (Security Config에 정의)
   - provider에서 제공한 AutorizationUrl에서 허용/거부가 정해짐
   - 이때 만약 유저가 앱에 대한 권한을 모두 허용하면 provider는 사용자를 callback url로 redirect (http://localhost:8080/oauth2/callback/{provider}) 
   - 이때 사용자 인증코드 (authroization code) 도 함께 갖고있다.
   - 만약 거부하면 callbackUrl로 똑같이 redirect하지만 error가 발생

3. Oauth2 에서의 콜백 결과가 에러인 경우
   - Spring Security는 oAuth2AuthenticationFailureHanlder를 호출 (Security Config에 정의)

4. Oauth2 에서의 콜백 결과가 성공인 경우
   - 사용자 인증코드 (authorization code)도 포함하고 있다면 Spring Security는 access_token에 대한 authroization code를 교환하고, customOAuth2UserService 를 호출(Security Config에 정의)

5. customOAuth2UserService는 인증된 사용자의 세부사항을 검색한 후에 DB에 Create 혹은 동일 Email로 Update 하는 로직을 작성

6. oAuth2AuthenticationSuccessHandler가 호출
    - 사용자에 대한 JWT 인증 토큰을 생성하고 쿼리 문자열의 JWT 토큰과 함께 redirect_uri로 사용자를 보냄

## 인증/인가 로직 구현 시점

<img width="818" alt="스크린샷 2023-05-24 오전 10 47 45" src="https://github.com/Voyager003/PracticeCode/assets/85725033/7c56259c-f830-4bc2-a0b4-2a1cb257ccce">

- Interceptor와 Filter(Spring Security)는 모두 Controller에 request가 도착하기 이전에 로직을 수행하거나, 사용자에게 response가 전달되기 전에 
로직을 처리할 수 있도록 돕는 역할을 한다. 
- 차이가 있다면 Interceptor는 Spring Context에 등록되어 관리되는 반면, Filter는 Servlet Context에 조냊하여 실행되는 시점이 다르다는 점이다.
- 이를 자세하게 구분하면 다음과 같다.
  - Interceptor의 경우 Controller 처리 이전(preHandle), 이후(postHandle), View rendering 이후(afterCompletion)에 로직이 수행된다.
  - 반면 Filter는 Spring 영역의 Dispatcher Servlet 요청에 도착하기 이전과 Dispatcher Servlet을 떠난 이후에 실행된다.
- 이러한 차이로 Interceptor는 특정 요청에만 전/후 로직이 필요할 경우, 입/출력 데이터를 가공할 때 사용하며
- 모든 요청에 로직이 필요한 경우, request/response header와 같은 매개변수를 수정해야할 때 주로 Filter를 사용한다.



## 참고자료
- https://dev.gmarket.com/45
- https://www.callicoder.com/spring-boot-security-oauth2-social-login-part-2/

