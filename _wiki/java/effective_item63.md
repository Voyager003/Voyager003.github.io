---
layout  : wiki
title   : 문자열 연결은 느리니 주의하라 
summary : 
date    : 2024-02-07 09:37:57 +0900
updated : 2024-02-07 09:48:25 +0900
tag     : java effectivejava
resource: 7B/6286D4-3BF1-460B-90A4-E4CF3A732C0B
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 문자열 연결 연산자(+)

문자열 연결 연산자는 여러 문자열을 하나로 합쳐주는 편리한 수단이다. 

이는 한 줄짜리 출력값이나 작고 크기가 고정된 객체의 문자열 표현을 만들 때는 괜찮지만 성능 저하를 고려해야 한다.

이유는 문자열 n개를 잇는 시간은 n^2에 비례하기 때문이다. 

문자열은 불변이기 때문에 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하여 성능 저하는 피할 수 없다. 예로 살펴보자.

```java
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(i);
    }
    return result;
}
```

위 코드는 문자열 연산이 반복문 안에서 수행되며 문자열을 변경할 때마다 새로운 문자열 객체를 생성하므로, 반복문이 실행될 때마다 문자열을 연결하는 것은 많은 메모리 할당과 복사 작업을 유발하게 된다.

item63이 제시하는 해결책은 StringBuilder를 사용하는 것이다.

```java
public String statement() {
    StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
        b.append(lineForItem(i));
    }
    return b.toString();
}
```

품목을 100개로 하고 lineForItem이 길이 80인 문자열을 반환하게 하여 성능을 비교한 결과 StringBuilder를 사용한 statement가 6.5배나 빨랐다.

성능을 고려하여 문자열을 연결하는 경우 문자열 연결 연산자를 피하고 StringBuilder의 append() 메서드를 사용하도록 하자.

## 참고자료

- 이펙티브자바 3판

