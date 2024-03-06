---
layout  : wiki
title   : 추상화 수준에 맞는 예외를 던지라 
summary : 
date    : 2024-03-06 09:36:13 +0900
updated : 2024-03-06 10:11:20 +0900
tag     : java effectivejava
resource: C3/BC6353-82C4-4D8D-8699-D2A385FF8E74
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 예외 번역(exception translation)

수행하려는 일과 관련없는 예외가 발생한다면, 메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴 때 발생하는 일이다. 이는 내부 구현 방식을 드러내 상위 레벨의 API를 오염시키게 된다.

예시를 들어본다면

```java
public class FileReader {
    public String readFirstLine(String path) {
        try {
            BufferedReader reader = new BufferedReader(new FileReader(path));
            return reader.readLine();
        } catch (FileNotFoundException e) {
            throw e; // 저수준 예외를 그대로 전파
        } catch (IOException e) {
            throw e; // 저수준 예외를 그대로 전파
        }
    }
}
```

FilerReader의 readFirstLine 메서드는 FileNotFoundException과 IOException 이라는 저수준의 예외를 그대로 던지고 있다. 이는 이 메서드를 사용하는 클라이언트는 이 두 가지 예외에 대해 알아야 하며 메서드의 추상화 수준을 깨트리게 된다.

```java
public class FileReader {
    public String readFirstLine(String path) {
        try {
            BufferedReader reader = new BufferedReader(new FileReader(path));
            return reader.readLine();
        } catch (FileNotFoundException | IOException e) {
            throw new ReadException(e); // 추상화 수준에 맞는 예외로 전환
        }
    }
}
```

변경된 코드는 FileNotFoundException과 IOException을 잡아서 ReadException으로 바꿔서 던지는 방식을 사용했다.

ReadException은 'readFirstLine' 메서드와 동일한 추상화 수준에 있는 예외로, 이를 통해 내부 구현 방식의 세부 사항을 숨기고 메서드가 수행하는 작업과 관련된 예외만 클라이언트에게 보여줄 수 있다. 

이처럼 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 하는데 이를 `예외 번역` 이라 한다.

### 예외 연쇄(exception chaining)

예외 연쇄는 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식이다.

이는 별도의 접근자 메서드(Throwable의 getCause)를 통해 언제든 저수준 예외를 꺼내볼 수 있어 디버깅에 도움을 준다.

```java
public class FileReader {
    public String readFirstLine(String path) {
        try {
            BufferedReader reader = new BufferedReader(new FileReader(path));
            return reader.readLine();
        } catch (FileNotFoundException | IOException e) {
            ReadException readException = new ReadException("Error reading the file at " + path);
            readException.initCause(e); // 원인 예외 설정
            throw readException; // 예외 연쇄
        }
    }
}
```

ReadException은 FileReader에서 발생한  FileNotFoundException 또는 IOException을 원인 예외로 설정하고 있다. 여기서 getCause() 메서드를 통해 원인 예외를 얻어 처리할 수 있다.

여기서 대부분의 표준 예외 클래스는 Throwable의 원인 예외를 받는 생성자를 가지고 있어 코드를 더 간결하게 작성할 수 있다. 

```java
public class FileReader {
    public String readFirstLine(String path) {
        try {
            BufferedReader reader = new BufferedReader(new FileReader(path));
            return reader.readLine();
        } catch (FileNotFoundException | IOException e) {
            throw new ReadException("Error reading the file at " + path, e); // 예외 연쇄
        }
    }
}
```

ReadException 생성자가 원인 예외를 인자로 받아, 예외 연쇄를 간결하게 구현하고 있다. 

이는 ReadException을 처리하는 코드에서 getCause() 메서드를 통해 원인 예외를 얻어 처리할 수 있다.

## 요점

무턱대로 예외를 전파하는 것보다 예외 번역이 우수한 방법이지만 남용해선 안된다. 

가능하다면 저수준 메서드가 반드시 성공하도록 보장하여 하위 계층에서 예외가 발생하지 않도록 하는 것이 최선의 방법이다. 

item73은 차선책으로 하위 계층에서 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에게 전파하지 않는 방법을 제시한다. 여기서 발생한 예외는 util.logging과 같은 로그를 남겨 클라이언트 코드와 사용자에게 문제를 전파하지 않으면서 로그를 분석해 조치를 취할 수 있도록 한다.

## 참고자료

- 이펙티브자바 3판

