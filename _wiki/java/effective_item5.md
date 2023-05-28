---
layout  : wiki
title   : 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 
summary : 
date    : 2023-05-28 15:31:37 +0900
updated : 2023-05-28 17:35:26 +0900
tag     : java effectivejava
resource: 3C/7CEE26-1C2C-44F1-89BD-5218235AC8B0
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

객체지향 설계에서 많은 클래스가 하나 이상의 자원에 의존한다. item5는 의존상태의 잘못된 예로 두 가지를 든다.

## 자원을 직접 명시한 경우 

```java
// 정적 유틸리티 클래스
public class SpellChecker1 {

    private static final Lexicon dictionary = new KoreanDictionary();

    private SpellChecker1() {
    }

    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}
```

- item4의 정적 유틸리티 클래스로 구현한 SpellChecker1 클래스이다.
- SpellChecker1은 KoreanDictionary 클래스를 직접 생성하여 dictionary 변수에 할당하고 있다.
- 이는 다른 종류의 사전으로 사용한다고 했을 때, 교체하기 힘들며, SpellChecker1을 테스트하고자 할 때, dictionary도 같이 테스트하게 되는 문제점이 있다.

```java
// 싱글톤
public class SpellChecker2 {

    private final Lexicon dictionary = new KoreanDictionary();

    private SpellChecker2() {
    }
    
    public static SpellChecker2 INSTANCE = new SpellChecker2();

    public static boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public static List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}
```

- item3의 싱글톤 방식을 사용한 SpellChecker2 클래스이다.
- 인스턴스 생성없이 static으로 직접 할당을 하는 위의 두 방식은 dictionary를 바꾸기 쉽지않다.
- setDictionary()를 만들어 정적 멤버를 변경한다면 멀티 스레드 환경에서 버그를 유발하기 쉽다.
- 이처럼 어떤 클래스가 사용하는 리소스에 따라 행동을 달리해야 하는 경우 위의 두 방법은 부적절하다.

## 의존 객체 주입

```java
public class SpellChecker3 {
    private final Lexicon dictionary;

    public SpellChecker3(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        throw new UnsupportedOperationException();
    }

    public List<String> suggestions(String typo) {
        throw new UnsupportedOperationException();
    }
}

// 구현
public static void main(String[] args) {
    Lexicon lexicon = new KoreanDictionary();
    
    SpellChecker3 spellChecker = new SpellChecker3(lexicon);
    spellChecker.isValid("test");
}
```

- 의존성 주입방식은 위의 두 방법의 문제점을 해결한다. 
- 인스턴스 생성 시, 생성자에 필요한 자원을 넘겨받아 사용하면되고, dictionary가 final 키워드로 설정되어있어 새로운 참조값을 받을 수 없는 불변의 상태를 만들 수 있다.
- 또한 lexicon이라는 인스턴스를 테스트하려면 테스트용 lexicon으로 교체할 수 있다.
    
    ```java
    class KoreanDictionary implements Lexicon {}
    
    class TestDictionary implements Lexicon {}
    ```
    - SpellChecker3 안에 있는 내용만 유닛 테스트가 가능해진다.

## Spring의 의존성 주입

```java
// Lexicon interface
public interface Lexicon {}

// SpellChecker
@Component
public class SpellChecker {
    
    private Lexicon lexicon;
    ...
    }
}

// KoreanDictionary
@Component
public class KoreanDictionary implements Lexicon {...} 

// config
@Configuration
@ComponentScan(basePackageClasses = Config.class)
public class Config {...}

// 사용
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Config.class);
SpellChecker spellChecker = applicationContext.getBean(SpellChecker.class);
spellChecker.isValid("test");
```

- Spring boot를 사용한다면 위처럼 직접 작성하여 사용하지는 않지만, Spring의 의존성 주입 방식은 다음과 같다.
- @Component를 통해 Spring Container에 Bean을 등록하고 Config 설정의 ComponentScan을 통해 인스턴스를 사용하는 쪽에서는 Bean을 받아(getBean) 사용한다.

## 참고자료

- 이펙티브 자바 3판
- https://www.youtube.com/watch?v=24scqT2_m4U&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=5&ab_channel=%EB%B0%B1%EA%B8%B0%EC%84%A0 
