---
layout  : wiki
title   : 지역변수의 범위를 최소화하라 
summary : 
date    : 2024-01-29 09:49:02 +0900
updated : 2024-01-29 10:24:11 +0900
tag     : java effectivejava
resource: 44/4004FF-C2FE-4457-8A8A-B07769E8830E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 지역변수의 유효범위 최소화

지역변수의 유효범위를 최소로 줄인다면 가독성, 유지보수성이 높아지고 오류 발생을 낮출 수 있다.

item57에서 제시하는 방법은 다음과 같다.

- 가장 처음 쓰일 때 선언

- 선언과 동시에 초기화

초기화에 필요한 정보가 충분치 않다면 선언을 미뤄야 한다. 이 때, try-catch문은 이 규칙에서 예외다.

변수 초기화 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야 한다. 이는 예외가 블록을 넘어 메서드까지 전파되기 때문이다.

변수 값을 try 블록 바깥에서도 사용해야 한다면 try 블록 앞에서 선언한다.

```java
public static void main(String[] args) {

    // 변수를 try 블록 앞에서 선언
    int result;

    try {
        // try 블록 내에서 변수 초기화
        int num1 = 10;
        int num2 = 0;

        // 예외 발생 가능성이 있는 코드
        result = num1 / num2;
    } catch (ArithmeticException e) {
        
        // 예외 처리
        System.out.println("Error: Division by zero");
            
        // 변수를 catch 블록 내에서 초기화할 수도 있음
        result = -1;
    }
}
```

- 반복 변수의 값을 반복문 종료 후에도 써야하는 상황이 아니라면 while문보다 for문을 사용

```java
// 반복자가 필요할 때의 관용구
for(Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ...
}

Iterator<Element> i = c.iterator();
while(i.hasNext()) {
    doSomething(i.next());
}

Iterator<Element> i2 = c2.iterator();
while(i.hasNext()) { // bug
    doSomething(i2.next());
}
```

두 번째 while문에서 반복변수 i2를 초기화했지만, 이는 이전의 while문에서 사용한 i를 다시 사용한 경우이다. 

i의 유효 범위가 끝나지 않았기 때문에 컴파일도 잘되고 실행 시 예외도 던지지 않지만, 두 번째 while문에서 c2를 순회하지 않고 c2가 비어있다고 판단하게 된다. 

이런 버그를 피하려면 for문을 사용하여 방지하는 것이 좋다.

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
    ...
}

// comile error!
for(Iterator<Element> i2 = c2.iterator(); i.hasNext();) { // i를 찾을 수 없다는 컴파일 오류 발생
    Element e = i2.next();
    ...
}
```

for문을 사용하면 컴파일 타임에 오류를 알 수 있으며, 변수 유효 범위가 for문 범위와 일치한다.

더불어 while문보다 짧아 가독성이 좋아진다.

- 메서드를 작게 유지하고 한 가지 기능에 집중

한 메서드에서 여러 기능을 처리하는 경우, 그 중 한 기능과 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다.

이를 방지하는 단순한 방법은 메서드를 기능별로 쪼개는 것.

## 참고자료

- 이펙티브자바 3판

