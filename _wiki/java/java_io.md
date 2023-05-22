---
layout  : wiki
title   : Java I/O
summary : 
date    : 2023-05-22 19:27:14 +0900
updated : 2023-05-22 22:33:49 +0900
tag     : java
resource: 51/FB087C-A10E-4683-B641-C08707F5090F
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/13) 13주차 과제 

## Java Stream

### Stream(스트림)

![](https://docs.oracle.com/javase/tutorial/figures/essential/io-ins.gif)
![](https://docs.oracle.com/javase/tutorial/figures/essential/io-outs.gif)

> 데이터를 입출력하기 위한 추상화된 개념으로, 데이터가 이동하는 통로, 흐름을 나타낸다.

- 스트림은 데이터를 운반하는데 사용되는 연결 통로이다.
- 스트림은 FIFO 구조로 단방향 통신, 즉 하나의 스트림으로 입/출력을 동시에 처리할 수 없다.
- 실제 데이터가 모두 전송되기 전까지 지연 상태를 유지한다.
- 입/출력을 동시에 처리하려면 InputStream과 OutputStream 2개의 스트림이 필요하다.
- 이 때, 먼저 보낸 데이터를 먼저 받게 되어있으며 중간에 건너뜀 없이 연속적으로 데이터를 주고 받는다.

#### InputStream, OutputStream

- InputStream, OutputStream은 **바이트 단위**로 데이터를 전송하며 입/출력 대상에 따라 다르다.
    - FileInputStream, FileOutputStream : 파일
    - ByteArrayInputStream, ByteArrayOutputStream : 메모리(byte 배열)
    - PipedInputStream, PipedOutputStream : 프로세스(프로세스 간 통신)
    - AudioInputStream, AudioOutputStream : 오디오 장치
- 제시된 입/출력 스트림은 모두 InputStream, OutputStream의 자손으로 읽고 쓰는데 필요한 추상 메서드를 구현해놓은 구현체이다.

- InputStream
  - abstract int read() : 입력 스트림으로부터 1바이트를 읽고 읽은 바이트를 반환한다. 더 이상 읽을 바이트가 없으면 -1을 반환
  - int read(byte[] b) : 입력 스트림으로부터 b.length만큼의 바이트를 읽고 읽은 바이트를 b에 저장하고 읽은 바이트 수를 반환한다. 더 이상 읽을 바이트가 없으면 -1을 반환
  - int read(byte[] b, int off, int len) : 입력 스트림으로부터 len만큼의 바이트를 읽고 읽은 바이트를 b[off]부터 저장하고 읽은 바이트 수를 반환한다. 더 이상 읽을 바이트가 없으면 -1을 반환

- OutputStream
  - abstract void write(int b) : 출력 스트림으로 1바이트를 쓴다.
  - void write(byte[] b) : 출력 스트림으로 b.length만큼의 바이트를 쓴다.
  - void write(byte[] b, int off, int len) : 출력 스트림으로 b[off]부터 len만큼의 바이트를 쓴다.

### 보조 스트림(Decorator Stream)

- 일반적으로 입/출력 작업은 작은 단위의 데이터를 여러 번에 걸쳐 전송하는 경우가 많다.
- 이 때, 매번 작업 수행 시 실제 입/출력 장치와 통신하는 비용이 발생하므로 작업이 느려질 가능성이 있다.
- 이러한 비효율성을 개선하기 위해 스트림의 기능을 보완하기 위한 보조 스트림인 버퍼를 사용한다.
- 버퍼는 입/출력 횟수(시스템 콜)를 줄여 성능 상 이점이 생기는 것이다.

```java
// 스트림 생성
FileInputStream fileInputStream = new FileInputStream("test.txt");

// 보조 스트림을 생성
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);

// 사이즈 정의 생성
BufferedInputStream bis = new BufferedInputStream(fileInputStream, 8192);

// 보조스트림을 이용해 데이터를 읽기
bufferedInputStream.read();
```

#### 보조 스트림 종류

- FilterInputStream, FilterOutputStream : 필터를 이용한 입출력 처리
- BufferedInputStream, BufferedOutputStream : 버퍼를 이용한 입출력 처리
- DataInputStream, DataOutputStream : 자바의 primitive type을 그대로 입출력 처리
- SequenceInputStream : 두 개의 스트림을 하나로 연결하여 입출력 처리
- LineNumberInputStream : 읽어온 데이터의 라인번호를 카운트
- ObjectInputStream, ObjectOutputStream : 객체를 직렬화하여 입출력 처리
- PrintStream : 출력 스트림에 출력하는 메서드를 추가
- PushbackInputStream : 버퍼로 읽어온 데이터를 다시 되돌림

### 문자기반 스트림 Reader, Write

- 바이트기반의 스트림의 단점을 보완하기 위한 문자기반의 스트림이다.
- 문자 데이터 입출력시 사용한다.

#### 종류

- FileReader, FileWriter : 파일 입출력
- ByteArrayInputStream, ByteArrayOutputStream : 메모리 입출력
- PipedReader, PipedWriter : 프로세스 간 통신
- StringBufferInputStream, StringBufferOutputStream : 문자열 입출력
- CharArrayReader, CharArrayWriter : 문자 배열 입출력

### NIO(New Input/Output)

- JDK 1.4에 등장한 NIO는 버퍼, 채널 기반으로 동작하는 입/출력이다.
- 스트림과 달리 양방향으로 입/출력이 가능하여 입/출력을 위한 별도의 채널을 만들 필요가 없다.
- 일반 I/O는 출력 스트림이 1byte를 쓰면 입력 스트림이 1byte를 읽는다.
  - NIO는 기본적으로 버퍼를 사용해 입출력을 사용하여 더 높은 성능을 가진다.
- blocking과 non-blocking 특징을 모두 갖는다.
  - 입출력 작업 준비가 완료된 채널만 선택하여 작업 스레드가 처리하기 때문에 작업 스레드가 blocking되지 않는다.


#### NIO Package

- java.nio : 다양한 버퍼 클래스
- java.nio.channels : 파일 채널, TCP, UDP 채널 클래스
- java.nio.channels.spi : java.nio.channels 패키지를 위한 서비스 제공자 클래스
- java.nio.charset : 문자셋 인코딩 및 디코딩 클래스
- java.nio.charset.spi : java.nio.charset 패키지를 위한 서비스 제공자 클래스
- java.nio.file : 파일 및 파일 시스템 접근 클래스
- java.nio.file.attribute : 파일 및 파일 시스템 속성에 접근하기 위한 클래스

## 표준 스트림(System.in, System.out, System.err)

- 콘솔을 통한 데이터 입력과 데이터 출력을 의미한다.
- Java 애플리케이션 실행과 동시에 사용할 수 있게 자동적으로 생성되어 별도로 생성 코드작성 필요가 없다.

### System 클래스

```java

public final static InputStream in = null;

...

public final static PrintStream out = null;

...

public final static PrintStream err = null;
```

- in, out, err는 System 클래스의 선언된 static 변수다.
  - System.in : 콘솔로부터 데이터를 입력받기
  - System.out : 콘솔로 데이터를 출력
  - System.err : 콘솔로 데이터를 출력
- 이 때, 버퍼를 이용하는 BufferedInputStream, BufferedOutputStream의 인스턴스를 사용한다.

### setOut, setErr, setIn

- 최초에는 System.in, out, err의 입출력대상이 콘솔이지만 입/출력을 콘솔 이외의 다른 입/출력 대상으로 변경할 수 있다.
  - static void setOut(PrintStream out) : System.out의 출력을 지정된 PrintStream으로 변경
  - static void setErr(PrintStream err) : System.err의 출력을 지정된 PrintStream으로 변경
  - static void setIn(InputStream in) : System.in의 출력을 지정된 InputStream으로 변경

```java
public class setEx {

    public static void main(String[] args) {

        try (FileOutputStream fos = new FileOutputStream("test.txt");
             PrintStream ps = new PrintStream(fos)) {

            // System.out 출력 대상을 test.txt 파일로 변경
            System.setOut(ps);

            System.out.println("out - ex1");
            System.err.println("err - ex1");
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// test.txt에 out - ex1 반영
```

## 파일 읽고 쓰기 

```java
public class InputEx {
  public static void main(String[] args) {
      
    try {
      FileInputStream file = new FileInputStream("C:\\test.txt");
      FileOutputStream fileOut = new FileOutputStream("C:\\output.txt", true);


      BufferedInputStream buffIn = new BufferedInputStream(file);
      BufferedOutputStream buffOut = new BufferedOutputStream(fileOut);

      int ch;

      while((ch = buffIn.read()) != -1) {
        buffOut.write(ch);
      }


      buffIn.close();
      buffOut.close();
      file.close();
      fileOut.close();
    } catch (Exception e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
}
```

- 입력파일 C:\text.txt에서 데이터를 버퍼를 사용해 c:\output.txt에 쓰는 예시이다.
- FileInputStream과 FileOutputStream을 사용하여 입력 파일과 출력 파일을 open
- BufferedInputStream과 BufferedOutputStream을 생성하여 각각 file과 fileOut 스트림을 감싼다.
- while 루프문으로 입력 파일에서 데이터를 읽어와 읽은 데이터를 출력 파일에 write
- 입력 파일의 끝에 도달하면 read()는 -1을 반환하므로, while 루프를 종료
- buffIn, buffOut, file, fileOut 스트림을 각각 닫아주고, 입출력 작업을 완료

## 참고자료

- https://docs.oracle.com/javase/tutorial/essential/io/streams.html 
- https://five-cosmos-fb9.notion.site/I-O-af9b3036338c43a8bf9fa6a521cda242
