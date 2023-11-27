---
layout  : wiki
title   : 명명 패턴보다 애너테이션을 사용하라 
summary : 
date    : 2023-11-26 21:48:39 +0900
updated : 2023-11-27 11:51:22 +0900
tag     : 
resource: 1A/C67A01-A157-43D2-9CB7-099C676CDC1D
toc     : true
public  : true
parent  : 
latex   : false
---
* TOC
{:toc}

## 명명 패턴의 특징

테스트 프레임워크인 JUnit5는 버전 3까지 테스트 메서드의 이름을 test로 시작하게끔 했다. 

이 방법은 실수로 메서드 명을 tsetSaferyOverride로 짓는다면 Junit은 이 메서드를 무시하고 지나치기 때문에 테스트가 통과했다고 오해를 살 수 있다.

또한 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다. 메서드가 아닌 클래스 명을 TestSaferyMechanism으로 지어 Junit에 넘긴다면, 이 클래스에 정의된 테스트 메서드들을 수행해주길 기대했지만 개발자가 의도한 테스트는 전혀 수행되지 않는다.

그리고 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다. 특정 예외를 던져야만 성공하는 테스트가 있다면, 기대하는 예외 타입을 테스트에 매개변수로 전달해야하는 상황이 생긴다. 예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만, 보기도 나쁘고 깨지기도 쉽다. 테스트를 실행하기 전에는 그런 이름의 클래스가 존재하는지 혹은 예외가 맞는지조차 알 수 없다.

## 애노테이션

[애노테이션](https://voyager003.github.io/wiki/java/java_annotation/#annotation%EA%B3%BC-%EC%A0%95%EC%9D%98%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)은 위에서 말한 문제를 해결해주는 개념으로 Junit도 버전 4부터 이를 도입했다.

Test라는 이름의 애노테이션을 정의한다고 가정해보자.

```java
import java.lang.annotation.*;

/**
 * 테스트 메서드임을 선언하는 애노테이션
 * 매개변수 없는 정적 메서드 전용
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

@Test 애노테이션 외 @Retention과 @Target이라는 메타 애노테이션이 명시되어있다. 

@Retention은 @Test가 런타임에도 유지되어야 한다는 의미로, 생략한다면 테스트 도구는 @Test를 인식할 수 없다. 

@Target은 반드시 메서드 선언에서만 사용해야한다고 알린다. 클래스나 필드 선언에는 명시가 불가능하다.

```java
public class Sample {
    @Test 
    public static void m1() {
    } // 성공

    @Test
    public static void m3() {
        throw new RuntimeException("fail");
    }
    ...
    
    /**
     * 정적 메서드가 아님
     */
    @Test
    public void m5() {
    }

    public static void m4(){
    }
}
```

위 코드는 @Test 애노테이션을 실제 적용한 모습이다. 이처럼 애노테이션을 아무 매개변수없이 단순히 대상에 마킹한다는 의미에서 마커(marker) 애노테이션이라 한다.

애노테이션 사용 시 개발자가 Test 이름에 오타를 내거나 메서드 선언 외의 프로그램 요소에 달면 컴파일 오류를 발생시킨다. 

@Test 애노테이션이 Sample 클래스의 의미에 직접적 영향을 주지 않고, 대상 코드의 의미는 그대로 둔 채 테스트 도구에서 특별한 처리를 할 기회를 준다. (추가 정보를 제공)

item39는 RunTests를 예로 든다. 

```java
public class RunTests {
    public static void main(String[] args) throws ClassNotFoundException {
        int cnt = 0;
        int passed = 0;

        Class<?> testClass = Class.forName("ch6.dahye.item39.AnnotationSample");

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                cnt++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException e) {
                    Throwable exc = e.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception e) {
                    System.out.println("잘못 사용한 예 " + m);
                }
            }
        }
        System.out.printf("성공 : %d, 실패: %d%n",
                            passed, cnt-passed);
    }
}
```

RunTest는 명령줄로부터 완전 정규화된 클래스 이름을 받아, 그 클래스에서 @Test 애노테이션이 명시된 메서드를 차례로 호출한다. 테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 InvocationTargetException으로 감싸서 던지고 원래 예외에 담긴 실패 정보를 추출(getCause)해 출력한다.

### 매개변수를 가진 애노테이션

특정 예외를 던져야만 성공하는 테스트를 지원하도록 해보자. 이를 위해서는 새로운 애노테이션 타입이 필요하다.

```java
/**
 * 명시한 예외를 던져야만 성공하는 애노테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

애노테이션의 매개변수 타입은 Class<? extends Throwable>로 와일드카드 타입이다. 

이는 Throwable을 확장한 클래스의 Class 객체라는 뜻으로 모든 예외 타입을 수용한 한정적 타입 토큰의 활용 사례이다. 이 애노테이션을 다룰 수 있도록 main 메서드를 수정하면

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedEx) {
        Throwable exc = wrappedEx.getCause();
        Class<? extends Throwable> excType =
            m.getAnnotation(ExceptionTest.class).value();
        if (excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf(
                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                    m, excType.getName(), exc);
        }
    } catch (Exception exc) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```

애노테이션의 매개변수의 값을 추출해 테스트 메서드가 올바른 예외를 던지는지 확인하는 데 사용한 코드이다. 테스트가 문제없이 컴파일되면 애노테이션 매개변수가 가리킨은 예외가 올바른 타입일 것이고, 컴파일 타임에는 존재했지만 런타임에는 존재하지 않는 경우 TypeNotPresentException을 던질 것이다.

### 배열 매개변수를 받는 애노테이션

추가적으로 여러 개 명시하고 그 중 하나가 발생하면 성공하게 만드는 것도 가능하다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
```

매개변수 타입을 Class 객체의 배열로 수정했다. 원소가 여럿인 배열을 지정 시, 다음과 같이 원소들을 중괄호로 감싸 쉼표로 구분하면 된다.

```java
@ExceptionTest({...Exception.class, ...Exception.class})
```

이제 @ExceptionTest를 지원하도록 테스트 러너를 수정하면,

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
  } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        Class<? extends Throwable>[] excTypes =
            m.getAnnotation(ExceptionTest.class).value();
        for (Class<? extends Throwable> excType : excTypes) {
            if (excType.isInstance(exc)) {
                passed++;
                break;
            }
        }
    if (passed == oldPassed)
        System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```

예외를 배열로 값을 받아와 모든 예외 처리가 성공한 경우에만 테스트를 성공하도록 수정하여 직관적이다.

### 반복 가능한 애노테이션

Java 8부터는 여러 개의 값을 받는 애노테이션을 배열 매개변수를 사용하는 대신 @Repeatable 메타 애노테이션을 명시하여 구현할 수 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

구현 방법을 보면 먼저 @Repeatable을 단 애너테이션을 반환하는 '컨테이션 애너테이션' 을 하나 더 정의하고, @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.

컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야하며, 컨테이너 애너테이션 타입에 적절한 @Retention과 @Target을 명시해야 한다. 이를 적용하면 다음과 같다.

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() { 
    ...
}
```

이 방법을 테스트 러너에 적용하면,

```java
if (m.isAnnotationPresent(ExceptionTest.class)
    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
    tests++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (Throwable wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
        ExceptionTest[] excTests =
                m.getAnnotationsByType(ExceptionTest.class);
        for (ExceptionTest excTest : excTests) {
            if (excTest.value().isInstance(exc)) {
                passed++;
                break;
            }
        }
        if (passed == oldPassed)
            System.out.printf("테스트 %s 실패: %s %n", m, exc);
    }
}
```

위와 같다. 이 때, @Repeatable 애노테이션 처리 시 주의해야할 점이 몇 가지 있다.

@Repeatable을 여러 개 명시하면 하나만 명시했을 때와 구분하기 위해 해당 '컨테이너' 애너테이션 타입이 적용된다.

위 예제 코드에서 getAnnotationsByType() 메서드는 반복 가능 애너테이션과 컨테이너 애너테이션을 모두 가져오지만, isAnnotationPresent() 메서드로 반복 가능 애너테이션이 달렸는지 검사하여 명확하게 구분한다. 

@ExceptionTest를 여러 번 명시한 메서드는 m.isAnnotationPresent(ExceptionTest.class)에 포함되지 않아 테스트를 모두 통과하는 반면,

@ExceptionTest를 한 번만 명시한 메서드는 m.isAnnotationPresent(ExceptionTestContainer.class)에 포함되지 않아 무시하고 지나친다.

애노테이션이 명시된 수와 상관없이 모두 검사하려면 둘을 따로따로 확인해야 한다.

테스트는 애노테이션으로 할 수 있는 일 중 극히 일부로 소스 코드에 추가 정보를 제공할 수 있는 도구를 만든다면 적당한 애노테이션 타입도 함께 정의하여 제공하자. 애노테이션으로 할 수 있는 일을 명명패턴으로 처리할 이유는 없다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판

