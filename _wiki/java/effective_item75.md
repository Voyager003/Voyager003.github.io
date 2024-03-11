---
layout  : wiki
title   : 예외의 상세 메시지에 실패 관련 정보를 담으라 
summary : 
date    : 2024-03-11 10:55:03 +0900
updated : 2024-03-11 11:30:31 +0900
tag     : java effectivejava
resource: FC/199358-F33B-41AE-8F5D-43D9B554F318
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 스택 추적(Stack trace)

Java 시스템은 예외를 잡지 못해 프로그램이 실패하면 예외 객체의 toString 메서드를 호출해 얻는 문자열인 스택 추적 정보를 자동으로 출력한다.

프로그램 실패를 분석해야 한다면 이 toString 메서드에 실패 원인에 관한 정보를 가능한 많이 담아 반환하는 일이 중요하다.

실패 순간을 포착하려면 발생환 예외에 관여된 모든 매개변수, 필드의 값을 실패 메시지에 담아야 한다. (이 때 비밀번호나 암호 키같은 보안과 관련된 정보는 담아선 안됨)

또한 예외의 상세 메시지와 최종 사용자에게 보여줄 오류 메시지를 혼동해서는 안된다.

item 75는 필요 정보를 예외 생성자에서 모두 받아 상세 메시지까지 미리 생성해놓는 방법을 제시한다.

```java
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
    
    // 실패를 포착하는 상세 메시지 생성
    super(String.format(
            "최솟값: %d, 최댓값: %d, 인덱스: %d",
            lowerBound, upperBound, index));
            
    // 실패 정보를 저장 
    this.lowerBound = lowerBound;
    this.upperBound = upperBound;
    this.index = index;
}
```

위와 같이 명시한다면 실패를 더 잘 포착할 수 있다. 

포착한 실패 정보는 예외 상황을 복구하는 데 유용할 수 있어 검사 예외에서 도움을 줄 것이다.

이 때 호출자가 실패와 관련된 정보를 얻을수 있는 접근자 메서드를 제공한다면 더 좋을 것(item70)


## 참고자료

- 이펙티브자바 3판


