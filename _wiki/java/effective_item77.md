---
layout  : wiki
title   : 예외를 무시하지 말라 
summary : 
date    : 2024-03-13 09:27:19 +0900
updated : 2024-03-13 09:33:25 +0900
tag     : java effectivejava
resource: A0/A8A2D5-F533-4F34-B0CF-9F4D6FC2B8C1
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 예외의 필요성

예외는 문제 상황에 잘 대처하기 위해 존재한다. try catch 블록에서 catch를 비운다면 예외가 존재할 이유는 없어진다. 

FileInputStream을 닫는 경우와 같이 예외를 무시해야 하는 경우도 있다.

예외를 무시하기로 했다면 catch 블록 안에 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 변경하여 이를 명시하자.

```java
...

try {
    numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimneoutException | ExecutionException ignored) {
    ...
}
```

이는 검사, 비검사 예외에 똑같이 적용된다.

예외를 무시하지 않고 바깥으로 전파하도록 놔둬도 최소한 디버깅 정보를 남긴 채 프로그램을 중단하게 할 수 있다.

## 참고자료

- 이펙티브자바 3판

