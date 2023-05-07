---
layout  : wiki
title   : Java 예외 클래스 
summary : 
date    : 2023-03-07 20:33:23 +0900
updated : 2023-05-07 21:29:31 +0900
tag     : java
resource: C7/8F4668-FE8A-4305-BDB7-CD9E06CA8A4E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/9) 9주차 과제

* TOC
{:toc}

## Java 예외 처리 방법 (try, catch, throw, throws, finally)

### try-catch

```java
try {
    // 예외 발생할 가능성이 있는 로직
} catch (Exception1 e1) {
    // Exception1 발생 시, 이를 처리하기 위한 로직
} catch (Exception2 e2) {
    // Exception2 발생 시, 이를 처리하기 위한 로직
}

// 예시
public class ExceptionEx {

    public static void main(String[] args) {
        try {
            /// 1을 0으로 나눴으므로 예외가 발생
            System.out.println(1 / 0);

        } catch (IllegalArgumentException e) {
            System.out.println(e.getClass().getName());
            System.out.println(e.getMessage());
        } catch (ArithmeticException e) {
            System.out.println(e.getClass().getName());
            System.out.println(e.getMessage());
        } catch (NullPointerException e) {
            System.out.println(e.getClass().getName());
            System.out.println(e.getMessage());
        }
        System.out.println("Exception 발생 X");
    }
}

// 결과
Task :ExceptionEx.main()
java.lang.ArithmeticException
```

- try 블럭에는 여러 개의 catch 블록이 올 수 있다.
- 이 중 발생한 예외의 종류와 일치하는 단 한개의 catch 블록만이 수행된다.
- catch 블럭안의 ExceptionN은 예외 클래스, eN은 해당 클래스의 인스턴스를 가르키는 참조변수이다.
- 예시의 경우, 1을 0으로 나눴을 때 ArithmeticException이 발생하여 이를 처리하기 위한 로직이 실행됐다.
- 예외가 발생하지 않은 경우, catch문을 모두 스킵하고 'Exception 발생 X'를 출력한다.

### Multicatch block

```java
public class ExceptionDemo {

    public static void main(String[] args) {
        try {
            System.out.println(1/0);
        } catch (RuntimeException | ArithmeticException e) {
            /// Error
            System.out.println(e.getMessage());
        }
    }
}
```

- JDK 1.7부터 catch block을 하나로 합칠 수 있게 되었다.
- 이 때, 나열된 예외 클래스들이 부모-자식 관계에 있다면 오류가 발생한다.
- 이는 자식 클래스로 잡아낼 수 있는 예외는 부모 클래스도 잡아낼 수 있기 때문에 코드가 중복된 것이기 때문이다.
- 컴파일러는 중복된 코드를 제거하라는 의미로 Error를 발생시킨다.
- 또한 하나의 블록으로 여러 예외를 처리하는 것이기 때문에, Multicatch block 블록 내 발생한 예외가 어디에 속한 것인지 알 수 없다.
- 그래서 참조변수 e에는 '|'로 연결된 예외들의 공통 조상 클래스에 대한 정보가 담긴다.

### throw

```java
public class ExceptionEx {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        String userName = scanner.nextLine();

        try {
            if (userName.equals(“Rome”)) {
                throw new IllegalArgumentException(“Improper username”);
            }
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

- **throw**는 고의로 예외를 발생시키는 키워드이다.
- 예시는 'Rome'이라는 username을 사용하면 예외를 발생시켜 프로그램을 끝내는 로직이다.

## throws

```java
void method() throws Exception1, Exception2, … ExceptionN {
    // method
}
```

- thorws는 메서드에 예외를 선언하는 키워드이다.
- 여러 개의 메서드를 ','로 구분하여 선언할 수 있다.
- throws로 예외가 선언된 메서드 사용 시, 사용자가 이를 확인하고 예외를 처리해야한다.
- 이는 해당 메서드에는 예외를 처리하지 않고, 해당 메서드를 사용하는 쪽이 예외를 처리하도록 책임을 전가하는 역할을 한다.

## finally

```java
try {
    // 예외가 발생할 가능성이 있는 로직
} catch {
    // 예외 처리를 위한 로직
} finally {
    // 예외 발생 여부와 상관없이 항상 실행되어야 할 로직
}

// 예시
public class ExceptionEx {

    public static void main(String[] args) throws Exception {
        methodA();
        System.out.println(“methodA가 복귀한 후 실행될 문장”);
    }

    static void methodA() {
        try {
            System.out.println(“try”);
            return;
        } catch (Exception e) {
            System.out.println(“catch”);
        } finally {
            System.out.println(“finally”);
        }
    }
}

// 결과
Task :ExceptionEx.main()
try
finally
methodA가 복귀한 후 실행될 문장
```

- 예외 발생 여부와 상관없이 항상 실행되어야 할 코드를 포함시킬 목적으로 사용한다.
- 예외 발생 시, try -> catch -> finally 순이며, 발생하지 않는다면 try -> finally 순으로 실행되는 것이다.
- 또한 finally 블록 내의 문장은 try, catch 블록에 return 문이 있더라도 실행된다.

### try-with-resource

```java
// try-with-resource 사용 전
public static void main(String args[]) throws IOException {
        FileInputStream is = null;
        BufferedInputStream bis = null;
        
        try {
            is = new FileInputStream("file.txt");
            bis = new BufferedInputStream(is);
        //...
        } catch (IOException e) {
        // 에러처리
        } finally {
        // 어떤 경우에도 반드시 자원 해제
        if (is != null) is.close();
        if (bis != null) bis.close();
    }
}

// try-with-resource 사용 후
public static void main(String args[]) {
    
    try (
        FileInputStream is = new FileInputStream("file.txt");
        BufferedInputStream bis = new BufferedInputStream(is)
        ) {
        //... 
        } catch (IOException e) {
        // 에러처리
    }
}
```

- Java 7부터 자원 해제를 자동으로 제공하는 try-with-resources가 추가됐다.
- 예시로 DB connection과 같은 시스템 자원을 사용하는 코드의 경우 finally 구문을 통해 자원을 닫아주는 형태로 작성했다.
- try에 '()'를 열고 자원을 할당 시킨 뒤, 해당 자원은 블록 수행 후 자동으로 반환된다.
- 이 때, AutoCloseable 인터페이스를 구현한 클래스만이 자동으로 반환된다.

## Exception(예외) 계층 구조

> 프로그램이 Java 프로그래밍 언어의 의미적 제약 조건을 위반할 때, JVM은 이 오류를 예외로 프로그램에 알린다. 

예외는 다음 이유로 인해 예외가 발생한다.

## 예외의 종류

<img width="637" alt="스크린샷 2023-03-08 오전 12 31 47" src="https://user-images.githubusercontent.com/85725033/223469451-6a4d8302-8cd4-41a9-962b-39dc616b54b0.png">

### Throwable(Checked)

> Exception 과 Error class는 Throwable의 직접적인 sub class이다.

**Throwable**는 예외처리 최상위 클래스이다.

### Error(Unchecked)

> 프로그램이 일반적으로 복구되지 않을 것으로 예쌍되는 모든 예외의 super class이다.

**Error**는 메모리 부족 및 심각한 시스템 오류와 같은 애플리케이션의 복구 불가능한 시스템 예외로, 개발자는 오류가 발생하도록 하고 예외를 잡으려고 해선 안된다.

### Exception(Checked)

> 프로그램이 복구하고자 하는 모든 예외의 super class이다.

애플리케이션 로직에서 사용할 수 있는 실질적 최상위 예외 클래스이다.

Exception과 하위 예외는 모두 Compiler가 Check하지만, 하기에 설명된 Runtime Exception은 예외로 한다.

#### Runtime Exception(Unchecked)

Compiler가 체크하지 않는 unchecked 예외이다. nullpointerExcpetion, IllegalArgument.. 등 예외를 포함한다.

## RuntimeException과 아닌 것의 차이

![](http://www.tcpschool.com/lectures/img_java_exception_class_hierarchy.png)

- 위 그림의 파란 점선 영역은 **CheckedException**, 주황색 점선 영역은 **UnCheckedException** 이다.
- Runtime(JVM 구동)시 예외가 발생하는 것은 UnCheckedException, 실행하지 않고 예외가 검출되는 것은 CheckedException(RuntimeException)이다.
- RuntimeException 클래스를 상속받는 자식 클래스들은 주로 치명적 예외 상황을 발생시키지 않는 예외들로 구성되어 있다.
  - try-catch 블록보다 예외가 발생하지 않도록 코드를 작성하는 방향이 좋다.
- CheckedException 클래스인 Exception 클래스에 속하는 자식 클래스들은 치명적인 예외 상황을 발생시키기 때문에 반드시 try-catch 문을 사용하여 예외처리를 해야한다.
  - Java Compiler는 RuntimeException 클래스 외 Exception 클래스의 자식 클래스에 속하는 예외가 발생할 가능성이 있는 구문을 예외 처리 하지 않았을 때 반드시 예외를 처리하도록 강제한다.
  - 이를 Compile 단계에서 확인할 수 있다.

## 커스텀 예외(Custom Exception) 만드는 법

- 커스텀 예외 클래스는 일반 예외, 실행 예외로 선언할 수 있다.
  - 일반 예외로 선언할 경우 Exception을 상속한다.
  - 실행 예외로 선언할 경우, RuntimeException을 상속한다.

### 예시

```java
public class CustomException extends RuntimeException {

    // 1. 매개 변수가 없는 기본 생성자
    CustomException() {

    }

    // 2. 예외 발생 원인(예외 메시지)을 전달하기 위해 String 타입의 매개변수를 갖는 생성자
    CustomException(String message) {
        super(message); // RuntimeException 클래스의 생성자를 호출
    }
}

// 테스트
public static void main(String[] args) {
    try{
        test();
    } catch (CustomException e) {
        System.out.println("custom exception test");
    }
}

    public static void test() throws CustomException {
        throw new CustomException("Exception test");
    }
```

- throw로 강제로 예외를 발생시켜 메서드를 호출한 곳에서 커스텀 예외를 처리했다.


## 참고자료

- https://docs.oracle.com/javase/specs/jls/se11/html/jls-11.html#jls-11.1.1 - Java docs
- https://sites.google.com/site/myyclassnotes/java/exceptionhandlinginjava - 예외 계층 이미지
- https://docs.oracle.com/javase/tutorial/essential/exceptions/throwing.html
- https://wisdom-and-record.tistory.com/46
