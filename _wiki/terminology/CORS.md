---
layout  : wiki
title   : CORS(Cross-Origin Resource Sharing)
summary : 
date    : 2023-05-10 14:59:46 +0900
updated : 2023-05-10 21:22:52 +0900
tag     : terminology
resource: B8/5EB05A-F97C-402C-A0F3-8A205F3C6337
toc     : true
public  : true
parent  : [[/terminology]]
latex   : false
---
* TOC
{:toc}

## CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)란?

> 추가 HTTP 헤더를 사용하여, 한 출처에서 실행 중인 웹 애플리케이션이 다른 출처(origin)의 선택한 자원에 접근할 수 있는 권한을 부여하도록 브라우저에 알려주는 정책(체제)

- 여기서 언급된 웹의 출처는 URL의 Scheme(protocol), Host(domain), port를 의미한다.
- 웹 애플리케이션은 resouce가 자신의 출처와 다를 때, 교차 출처 HTTP 요청을 실행한다.
- 웹 브라우저는 보안 상의 이유로 기본적으로 SOP(동일 출처 정책, Same-Origin Policy)를 따른다. 여기서 보안상의 이유란 다음과 같다.
    - XSS(Cross-Site Scripting) : 악의적인 사용자가 웹페이지에 script 코드를 삽입하여 다른 브라우저에서 실행하도록 하는 것
    - CSRF(Cross-Site Request Forgery) : 악의적 웹사이트에서 사용자의 브라우저를 이용해 다른 웹사이트에 요청을 보내는 것
    - 데이터 유출 : 웹페이지에 load된 resource와 script가 노출
- CORS는 이 SOP를 우회하여 다른 도메인에 있는 resource에 접근할 수 있도록 하는 메커니즘이다.

### SOP(동일 출처 정책, Same-Origin-Policy)
  
> 동일한 출처에서만 resource를 공유할 수 있다

SOP는 동일 출처 서버에 있는 resource는 자유로이 가져올 수 있지만, 다른 서버에 있는 resource는 상호작용이 불가능하다는 의미이다.

여기서 같은 출처, 다른 출처를 어떻게 구분하는지 짚고 넘어가자면, Scheme, Host, port 3가지 요소가 동일하면 된다. 예를 통해 알아보자.

- https://voyager003.github.io/

블로그의 URL이다. 요소로 나눠보자면, 

1. Scheme : https:// 
2. Host : voyager003.github.io 
3. Port : 4000(4000이라고 가정)

이제 같은 출처와 다른 출처로 인정되는 예시를 들어보자.

- 같은 출처
  - https://voyager003.github.io/wiki -> scheme, host, port 모두 동일
  - https://user:password@voyager003.github.io/ -> 역시 scheme, host, port 모두 동일
- 다른 출처
  - http://voyager003.github.io/ -> scheme이 다름
  - https://rome003.github.io/ -> host가 다름
  - https://voyager003.github.io:8088 -> port가 다름

이처럼 출처를 비교하는 로직은 서버에 구현된 스펙이 아닌 **브라우저에 구현되어 있는 스펙**이다.

이는 브라우저를 통하지 않고, 서버 간 통신을 할 때는 정책이 적용되지 않음을 의미한다.
  
## 해결 방법

프론트엔드를 Vue js, 백엔드로 Spring boot를 사용하여 이를 연동하는 상황이며, vue js와 spring의 port는 각각 8089, 8085라 가정한다.

### Access-Control-Allow-Origin 세팅

서버에서 Access-Control-Allow-Origin Header에 알맞은 값을 세팅해주는 방식이다. 

기본적으로 

#### @CrossOrigin 애노테이션

```java
@RestController
@RequestMapping("/api")
public class MyController {

    @CrossOrigin(origins = "http://example.com", methods = {RequestMethod.GET, RequestMethod.POST})
    @GetMapping("/data")
    public ResponseEntity<String> getData() {
        // 데이터를 가져오는 로직
        return ResponseEntity.ok("Data response");
    }

    @CrossOrigin(origins = "*", allowedHeaders = {"Authorization"})
    @PostMapping("/save")
    public ResponseEntity<String> saveData(@RequestBody DataDto data) {
        // 데이터 저장 로직
        return ResponseEntity.ok("Data saved");
    }
}
```

- @CrossOrigin 애노테이션은 특정 컨트롤러 클래스와 메서드에 CORS 설정을 적용할 수 있다.
- 클래스에 선언한다면, 모든 메서드에 적용한다.
- 'origins'는 허용할 출처(도메인)을 지정한다. 기본값은 모든 도메인("*")이다. 
- 'methods'는 허용할 HTTP 메서드를 지정한다. 기본값은 모든 HTTP 메서드이다.
- getData() 메서드는 'http://example.com' 도메인에서 오는 GET 및 POST 요청을 허용한다.
- saveData() 메서드는 모든 도메인에서 오는 POST 요청을 허용하며, 'Authorization' Header만 허용한다.

#### WebMvcConfigurer 설정을 통해 적용하는 방식

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://example.com")
                .allowedMethods("GET", "POST")
                .allowedHeaders("Authorization")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

- Spring의 Configuration을 통해 프로젝트 전역으로 적용하는 방식이다.
- WebMvcConfigurer의 메서드인 addMapping 메서드를 오버라이딩한 뒤, CORS를 적용할 URL 패턴 및 메서드, header를 지정할 수 있다.
- 위의 설정은 '/api/'로 시작하는 경로에 대해서는 http://example.com 도메인에서 오는 GET 및 POST 요청이 허용된다.
- 또한 Authorization hedaer와 자격증명도 허용되며, 사전 검증 요청의 유효 기간은 3600초(1시간)로 설정했다.

### Webpack Dev Server로 리버스 프록싱

```javascript
// vue.conifjg.js

const { defineConfig } = require("@vue/cli-service");
module.exports = defineConfig({
  transpileDependencies: true,

    // npm run build target 디렉토리 지정
    outputDir: "../src/main/resources/static",

    devServer: {
        proxy: {
          '/api': {
            target: "http://localhost:8085",
            changeOrigin: true,
            ws: false
          }
        }
    }
});
```

로컬 환경에서 '/'로 시작하는 URL로 보내는 요청에 대해 브라우저는 localhost:8089/api로 요청을 보낸 것으로 인식하고 있지만, webpack이 http://localhost:8085로 요청을 대행하여 프록싱한다. 이는 CORS 정책을 지킨 것처럼 브라우저를 우회하여 원하는 서버와 통신할 수 있게 되었다.

주의점은 실제 프로덕션 환경에서도 클라이언트 애플리케이션의 소스를 나르는 출처와 API 서버의 출처가 같은 경우에만 사용하는 것이 좋다. 이유는 애플리케이션을 빌드하고 서버에 올리고나면 webpack 서버가 구동하는 환경이 아니기 때문에 원치 않는 API 요청을 보내기 때문이다. 


## 참고자료
- https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- https://evan-moon.github.io/2020/05/21/about-cors/ 

