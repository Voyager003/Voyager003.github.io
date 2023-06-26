---
layout  : wiki
title   : try-finally보다는 try-with-resources를 사용하라 
summary : 
date    : 2023-06-26 09:59:22 +0900
updated : 2023-06-26 11:59:00 +0900
tag     : java effectivejava
resource: 7D/A8AD28-1EDF-4F7F-A81C-91C9D8A12C66
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## try-finally

InputStream, OutputStream, java.sql.Connection 등 close()를 호출해 직접 닫아줘야하는 자원들이 많다.

close()는 클라이언트가 놓치기 쉬워 예측할 수 없는 성능 문제를 야기할 수도 있다.

이러한 경우 안전망으로 item8의 finalizer를 활용하지만 이미 item8에서 문제점을 살펴봤었다.

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.

```java
static String firstLineOfFile(String path) throws IOException{

    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

이 코드는 firstLineOfFile()의 br.readLine()에 문제가 발생하면 예외를 던지고, br.close()하여 stream을 닫는 메서드이다.

이 때, 예외는 try 블록과 finally 블록 두 가지 모두 발생할 수 있는데, 기기에 물리적 문제가 발생한다면 firstLineOfFile()의 readLine()이 예외를 던지고 close()는 실패할 것이다.

이는 두 번째 예외가 첫 번째 예외를 완전히 삼켜버려 StackTrace에 첫 번째 예외에 대한 정보가 남지 않아 디버깅이 어렵다.

상기된 문제로 item9는 try-with-resources를 해결책으로 제시한다.

## try-with-resources 

Java 7부터 자원 해제를 자동으로 제공하는 try-with-resources가 추가됐다.

```java
try (SomeResource resource = getResource()) {
    use(resource);
} catch(...) {
    ...
}
```

try에 자원 인스턴스를 전달하면 finally 블록으로 종료 처리하지 않아도 try 코드 블록이 끝나면 자동으로 자원을 종료해주는 기능이다.

이 때, try에 전달할 수 있는 자원은 AutoCloseable 인터페이스의 구현체로 한정된다.

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

BufferedReader의 API를 살펴보면,

```java
public class BufferedReader extends Reader {

    private Reader in;

    private char[] cb;
    private int nChars, nextChar;
    ...
}
```

Reader를 상속받고 있으며, Reader를 살펴보면

```java
public abstract class Reader implements Readable, Closeable {

    private static final int TRANSFER_BUFFER_SIZE = 8192;
    ...
}
```

Readable과 Closeable을 상속받고 있는 것을 확인할 수 있다.

이제 try-with-resources를 처음 작성했던 코드에 적용해보면

```java
static String firstLineOfFile(String path) throws IOException {
        // try - with - resources
    try (BufferedReader br = new BufferedReader(new FileReader(path)) {
        return br.readLine();
    }
}
```

다음과 같아진다. 

이제 firstLineofFile()은 readLine과 BuffredReader의 close 호출 양쪽에서 예외 발생 시, close에서 발생한 예외는 숨겨지고, readLine에서 발생한 예외가 기록된다.

이 때, 숨겨진 예외는 그냥 버려지지 않고 StrackTrace에 suppressed라는 표시를 남기고 출력되며, Throwable에 추가된 getSuppressed()를 이용하면 프로그램 코드에서 가져올 수도 있다.

try-with-resources을 사용하면 간결하면서 정확하고 쉽게 자원을 회수 할 수 있고, 만들어지는 예외 정보도 훨씬 유용하다.


## 참고자료

- 이펙티브 자바 3판
 
