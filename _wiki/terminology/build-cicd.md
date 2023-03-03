---
layout  : wiki
title   : 빌드와 배포
summary : 
date    : 2023-02-28 20:43:36 +0900
updated : 2023-03-03 22:53:14 +0900
tag     : terminology
resource: AD/FFE35F-C082-4791-B7BE-EC015E3E602D
toc     : true
public  : true
parent  : [[/terminology]]
latex   : false
---
* TOC
{:toc}

## Build(빌드)

> 소스코드 파일을 컴퓨터나 휴대폰에서 실행할 수 있는 독립(standalone) 소프트웨어 가공물로 변환하는 과정 및 그에 대한 결과물을 일컫는다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fd3s6bS%2FbtqNRJdufUI%2FfUfigVgPdDFR8rIAVMdQu1%2Fimg.png)

**Build**는 인간이 이해할 수 있게 작성된 '소스 코드'를 기계어(binary)로 번역하여 '실행 가능한 파일로 만드는 과정'이다. 

이러한 빌드 과정은 크게 3가지 방식으로 분류된다.

### Compile

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlANHp%2FbtqNU1kYGLA%2FZ5ksDLwxxBxmqTOhgYlb40%2Fimg.png)

Compile은 프로그래머가 작성한 소스코드를 한꺼번에 번역하여 실행파일로 만드는 방식이다. 크게 4가지 단계로 진행된다.

- Preprocessing(전처리) 
  - 소스코드에 포함 된 매크로나 지시자 등을 소스코드가 실행되기 전 전처리하는 과정이다.
  - C언어의 #include, #define이 그 예이다.
    
- Compilation(컴파일)
  - 컴파일러를 통해 low-level language(저급 언어)로 번역한다.
  - 보통 Assembly language로 번역된다.
  
- Assemble(어셈블)
  - 컴파일러로 번역한 어셈블리어를 기계가 이해할 수 있는 기계어로 번역한다.
  - 이 때, 번역된 파일을 Object file(목적 파일)이라고 한다.

- Linking(링크)
  - 위의 과정을 통해 번역된 기계어(Object file)를 하나로 연결하는 과정이다.
  - Object file들과 필요한 라이브러리를 연결시켜 최종적으로 실행 가능한 파일로 만든다.


### Interpreted

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbO2Oo5%2FbtqOeBdcHxz%2FWlUKczXrK02kNeE787Aa50%2Fimg.png)

인터프리트 방식은 소스코드를 한꺼번에 번역하는 것이 아닌 한 명령 단위로 해석하여 즉시 실행하는 방법이다.

- 인터프리트 방식은 Object file을 생성하지 않고 바로 실행되는 것이 특징이다.
- 컴파일 과정없이 인터프리터를 통해 결과를 바로 볼수 있어 프로그램 디버깅에 유리하다.

### Hybrid

![alt](https://img1.daumcdn.net/thumb/R1920x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdm5iRN%2FbtqOnyVFdv9%2FakMnSPl9xg5WVxD4eqr6Ok%2Fimg.png)

컴파일 방식과 인터프리트 방식을 혼합한 방식이다. 

컴파일 방식은 실행 가능한 파일이 플랫폼에 의존적이지만 실행속도가 빠르고, 인터프리트 언어는 느리지만 플랫폼에 독립적이고 인터프리트만 있으면 실행이 가능하다고 했다. 

상기된 단점을 상호 보완하여 만든 방식으로 Byte Code Language라고도 한다. 과정을 살펴보면

- 고급 언어로 작성된 소스 코드를 byte code로 변환한다.
- byte code를 Virtual Machine(VM) 프로그램으로 바이트코드를 기계가 이해할 수 있는 언어로 변환한다.

Java를 예로 들자면 Java 소스 코드(.java)를 컴파일하면 byte code(.class)로 변환하고, 이 바이트 코드를 JVM이 설치된 컴퓨터는 어디서든 동일한 결과를 
내도록 실행할 수 있다.

이와 같은 Build 과정을 거쳐 실행 가능한 소프트웨어 결과물을 얻는다.

## Deployment(배포)

> 최종 사용자에게 소프트웨어를 전달하는 과정이다. 

프로그래밍에서 Deployment는 3가지로 나뉘는데

1. Release: 같은 제품을 새롭게 만드는 것(예: 새로운 버전을 배포, 새로운 아이피 번호 부여)

2. Deploy: 프로그램 등을 서버와 같은 기기에 설치하여 서비스 등을 제공하는 의미

3. Distribute: 제품을 사용자들이 사용할 수 있도록 서비스 등을 제공하는 의미

주로 쓰이는 의미는 서버에 반영한다는 것을 의미한다.

- IDE에서 JAVA로 코딩을 하고 코드 완성 후 Run 버튼 눌러서 코드 실행 (빌드과정(컴파일))
- 정상 실행 후, 이를 .war 파일로 뽑아서(빌드) -> Web server에 올리거나(배포)
- Jsmooth 같은 tool을 사용하여 exe 파일로 추출하여(빌드) 사용자에게 올린다.(배포)

## 참고자료

- https://st-lab.tistory.com/176
- https://ko.wikipedia.org/wiki/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4_%EB%B9%8C%EB%93%9C - build란?
- https://www.bmc.com/blogs/software-deployment-vs-release/ 
- https://itholic.github.io/qa-compile-build-deploy/ - 배포 의미