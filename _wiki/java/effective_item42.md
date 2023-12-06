---
layout  : wiki
title   : 익명 클래스보다는 람다를 사용하라 
summary : 
date    : 2023-12-06 20:16:43 +0900
updated : 2023-12-06 22:15:18 +0900
tag     : java effectivejava
resource: 60/1F765F-5A79-46E1-ADF8-B4B2E501A3C4
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 람다 표현식 도입 이전의 함수 객체

```java
Collections.sort(words, new Comparator<String>() {
      public int compare(String s1, String s2){
          return Integer.compare(s1.length(), s2.length());
    }
});
```

JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단은 익명 클래스([item24](https://voyager003.github.io/wiki/java/effective_item24/#%EC%9D%B5%EB%AA%85-%ED%81%B4%EB%9E%98%EC%8A%A4anonymous-class)) 가 되었다.

예시 코드는 문자열을 길이 순으로 정렬하는 데 있어 정렬을 위한 비교함수로 익명 클래스를 사용했다.

전략 패턴처럼 함수 객체를 사용하는 과거 객체 지향 디자인 패턴에는 익명 클래스면 충분했지만, 익명 클래스 방식은 너무 길기 때문에 Java는 함수형 프로그래밍에 적합하지 않았다.

## 람다 표현식(Lambda Expression)

Java 8에서 추상 메서드 하나짜리 인터페이스는 특별한 의미를 인정받았다. 

함수형 인터페이스라 부르는 인터페이스들의 인스턴스를 람다 표현식을 사용해 만들 수 있게 되었다.

```java
Collections.sort(words, 
          (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

컴파일러가 문맥을 살펴 타입을 추론하므로 매개변수와 반환값의 타입을 생략할 수 있다. 상황에 따라 컴파일러가 타입을 결정하지 못할 수 도 있는데, 그럴 때는 프로그래머가 직접 명시해야한다.

타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다 표현식의 모든 매개변수 타입은 생략하도록 하자. 그런 뒤 컴파일러가 타입을 알 수 없다는 오류를 낼 때만 해당 타입을 명시하면 된다.

추가적으로 람다 표현식 자리에 비교자 생성 메서드와 List 인터페이스의 sort() 메서드를 추가한다면

```java
// 비교자 생성 메서드
Collections.sort(words, comparingInt(String::length));

// List.sort()
words.sort(comparingInt(String::length));
```

더욱 간결하게 만들 수 있다.

람다 표현식을 지원하면서 기존에 적합하지 않았던 부분에서도 함수형 객체를 실용적으로 사용할 수 있게 되었다. 


```java
// 상수별 클래스 몸체와 데이터를 사용한 열거 타입
public enum Operation {
    PLUS("+")    {public double apply(double x, double y){return x + y;}},
    MINUS("-")   {public double apply(double x, double y){return x - y;}},
    TIMES("*")   {public double apply(double x, double y){return x * y;}},
    DIVIDE("/")  {public double apply(double x, double y){return x / y;}};

    private final String symbol;
    ...
    
// 람다 표현식 적용
public enum Operation {
    PLUS("+", (x, y) -> x + y),
    MINUS("-", (x, y) -> x - y),
    TIMES("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;
    
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public double apply(double x, double y){
        return op.applyAsDouble(x, y);
    };
    ...
```

기존 apply 추상 메서드를 람다 표현식으로 구현하여 생성자에 넘기고, 생성자의 인스턴스 필드를 저장했다. 이 후 apply 메서드에서 필드에 저장된 람다 표현식을 호출하여 간결하게 만들 수 있다.

## 주의점 

람다 표현식으로 기존 방식을 모두 대체할 수 있다고 생각할 수 있지만, 그렇지만은 않다. 

메서드와 클래스와는 달리 람다 표현식은 이름이 없고 문서화가 불가능하여, 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다 표현식을 사용하지 않는 것이 좋다. (1~3 줄내로 작성)

람다 표현식이 길거나 읽기 어렵다면 줄여보거나 쓰지 않는 방향으로 리팩토링하라.

또한 열거 타입의 경우, 인스턴스가 런타임에 만들어지기 때문에 생성자에 넘겨진 인수들의 타입은 컴파일 타임에 추론되어 열거 타입 생성자 안의 람다 표현식은 열거 타입 인스턴스 멤버에 접근할 수 없다.

따라서 상수별 동작을 단 몇 줄로 구현하기 어렵거나, 인스턴스 필드나 메서드를 사용해야만 하는 상황이라면 상수별 클래스 몸체를 사용해야 한다.

추가로 람다 표현식도 익명 클래스처럼 직렬화 형태(ex) 가상 머신별로) 다를 수 있어 람다 표현식을 직렬화하는 것은 삼가해야 한다. 이는 익명 클래스의 인스턴스도 마찬가지이다. 직렬화가 필요하다면 private 정적 중첩 클래스의 인스턴스를 사용하도록 하자.

## 반대로 익명클래스를 사용하는 경우

먼저 람다 표현식은 함수형 인터페이스만 사용된다. 따라서 추상 클래스의 인스턴스를 만들 때 람다 표현식을 사용할 수 없기 때문에 익명 클래스를 사용해야 한다. 

또한 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때도 익명 클래스를 사용 가능하다. 

무엇보다 람다 표현식은 자기 자신(this)를 참조할 수 없다. 람다 표현식에서 this 키워드는 바깥 인스턴스를 가르킨다는 것이다. 

그에 반면 익명 클래스에서 this 키워드는 익명 클래스의 인스턴스 자신을 가리키기 때문에 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 사용해야 한다. 

## 참고자료

- 이펙티브자바 3판
