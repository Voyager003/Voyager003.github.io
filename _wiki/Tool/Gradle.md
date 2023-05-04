---
layout  : wiki
title   : Gradle build tool 
summary : 
date    : 2023-05-04 21:07:38 +0900
updated : 2023-05-04 22:34:24 +0900
tag     : gradle
resource: 6A/1871D5-0678-4DD6-ADF3-A3C84114207E
toc     : true
public  : true
parent  : [[/Tool]]
latex   : false
---
* TOC
{:toc}

## Gradle

- [Gradle](https://github.com/gradle/gradle)은 Java, Scala, Kotlin, C/C++ 및 Groovy를 비롯한 언어 및 플랫폼에서 빌드 자동화를 지원하는 
빌드 도구이다.
- XML 기반의 Ant, Maven와 달리 JVM에서 동작하는 Groovy 기반의 DSL(Domain Specific Language)를 사용한다.
- Maven의 pom.xml을 Gradle 용으로 변환할 수 있으며, Maven의 중앙 저장소도 지원하여 라이브러리를 그대로 사용할 수도 있다.

## 초기화 작업

```java
$ gradle init

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 3

Select implementation language:
  1: C++
  2: Groovy
  3: Java
  4: Kotlin
  5: Scala
  6: Swift
Enter selection (default: Java) [1..6] 3

Select build script DSL:
  1: Groovy
  2: Kotlin
Enter selection (default: Groovy) [1..2] 1

Select test framework:
  1: JUnit 4
  2: TestNG
  3: Spock
  4: JUnit Jupiter
Enter selection (default: JUnit 4) [1..4]

Project name (default: demo):
Source package (default: demo):


BUILD SUCCESSFUL
2 actionable tasks: 2 executed
```

- init은 다음과 같은 구조로 새 프로젝트를 생성한다.

```java
├── gradle 
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew 
├── gradlew.bat 
├── settings.gradle 
└── lib
    ├── build.gradle 
    └── src
        ├── main
        │   └── java 
        │       └── demo
        │           └── Library.java
        └── test
            └── java 
                └── demo
                    └── LibraryTest.java
```

- Gradle 빌드에 권장되는 사용법은 Gradle Wrapper를 사용하는 것이다.
    - Wrapper는 각 OS에 맞춰 build를 수행하도록하는 배치 스크립트이다.
    - Gradle Wrapper의 스크립트는 명시된 Gradle 버전을 호출하고, 자동으로 설치하여 빌드를 실행한다.
      - **gradle-wrapper.jar**는 wrapper 파일로, gradlew나 gradlew.bat 파일이 프로젝트 내에 설치하는 이 파일을 사용하여 gradle task를 실행하기 때문에 로컬 환경의 영향을 받지 않는다.
      - 이는 실제로 Wrapper 버전에 맞는 구성들을 로컬 캐시에 다운로드 받는다.
      - **gradle-wrapper.properties** 는 Wrapper의 설정 파일로, 이 파일의 wrapper 버전 등을 변경하면 task 실행시, 자동으로 새로운 Wrapper 파일을 로컬 캐시에 다운로드 받는다.
- gradlew 파일은 unix용 실행 스크립트이다.
  - gradle로 컴파일 혹은 빌드 시, > gradle build 실행 시, 로컬에 설치된 gradle을 사용한다.
  - 이 경우 Java나 Gradle이 설치되어 있어야 하고, 새로받은 프로젝트의 Gradle 버전과 로컬에 설치된 Gradle 버전이 호환되지 않으면 문제가 발생할 수 있다. 
  - 따라서 Wrapper를 사용하면 다음과 같이 실행한다. > ./gradlew build
- gradle.bat 파일은 window용 실행 배치 스크립트이다.
  - window에서 실행 가능하다는 점만 제외하면 gradlew와 동일하다.
- build.gradle은 의존성 및 플러그인 설정을 위한 스크립트 파일이다.
- settings.gradle 파일은 프로젝트의 구성 정보를 기록하는 파일이다.
  - 하위 프로젝트들이 어떤 관계로 구성되어 있는지 서술한다.
  - gradle은 이를 참조하여 프로젝트를 구성한다.

## 참고자료

- https://github.com/gradle/gradle - gradle 레포
- https://docs.gradle.org/current/samples/sample_building_java_libraries.html