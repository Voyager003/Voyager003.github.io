---
layout  : wiki
title   : Java 연산자 
summary : 
date    : 2023-03-31 21:24:12 +0900
updated : 2023-04-05 22:15:08 +0900
tag     : java
resource: 01/421E7C-C6D0-4F93-B939-D037808BE93A
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디 3주차 과제](https://github.com/whiteship/live-study/issues/3)

## 산술 연산자(Arithmetic Operator)

- 산술 연산자는 부동 소수점, Character, 정수형과 함께 사용할 수 있다.
    - Additive operator(+) : 덧셈 연산자
    - Subtraction operator(-) : 뺄셈 연산자
    - Multiplication operator(*) : 곱셈 연산자
    - Division operator(/) : 나눗셈 연산자 -> 연산자가 모두 Integer라면 int를 반환하고, 부동 소수점의 값은 잃는다.
    - Remainder operator(%) : 나머지 연산자

## 비트 연산자(Bitwise Operator)

- bit단위 논리연산 시, 사용하는 연산자
    - AND(&) : 대응되는 bit가 모두 1이라면 1을 반환
    - OR(|) : 대응되는 bit 중 하나라도 1이면 1을 반환
    - XOR(^) : 대응되는 bit가 서로 다르다면 1을 반환
    - NOT(~) : bit연산을 반전시켜 반환(1->0, 0->1)
    - Left, Right Shift(<<, >>) : bit를 좌, 우로 옮긴 값을 반환
      - left shift 연산은 n만큼 left shift한다고 하면 2^n이 int형에 곱해진다.
      - right shift 연산은 n만큼 right shift한다고 하면 2^n을 나눈다.
      - 피연산자가 음수인 경우, 빈 공간을 1로 채운다.
    ```java
    int x = 15;
    int z = -50;
    System.out.println(Integer.toBinaryString(x)); // 1111 
    System.out.println(Integer.toBinaryString(x<<2)); // 111100 (1111 -> 111100) 
    System.out.println(Integer.toBinaryString(x>>2)); // 0011 (1111 -> 0011)
    System.out.println(Integer.toBinraySTring(z)); // 111111111111111111111001110
    System.out.println(Integer.toBinraySTring(z>>1)); // 11111111111111111111100111
    ```
  
    - Unsigned Right(>>>) : 피연산자의 비트 열을 오른쪽으로 이동하고, 이동에 따른 빈 공간은 0으로 채운다.
    ```java
    int x = -1; // 1111 1111
    System.out.println(x>>>1); // 0111 1111
    ```

## 관계 연산자(Relational Operator)

- 관계 연산자는 연관된 서로의 값들이 같은지 비교하거나, 크고 작음을 서로 비교한다. 
  - equal to(==) : primitive type에 한해 두 피연산자의 값이 같으면 true 아니라면 false를 반환한다.
    - reference type의 경우, 객체의 참조 주소값을 비교하는데 이는 두 개의 값이 같은지 판단하는게 아닌 두개의 주소가 같은지 판단하는 것이다.
  - not equal to(!=) : equal to의 반대 개념으로 같지 않다면 true 아니라면 false를 반환한다.
    - equal to와 마찬가지로 primitive type값 비교는, 이 두개의 값(Value)이 서로 다르다면 true를 반환하게 된다. 
    피연산자가 reference type이라면, 둘의 동일한 heap 주소값을 가지고 있다면 false, 동일하지 않은 주소값을 가지고 있으면 true를 반환
  - greater than(>) : 첫 번째 연산자가 두번째 연산자 보다 크다면, true를 반환
  - greater than or equal to(>=) : 첫 번째 연산자가 두번째 연산자보다 같거나 크다면, true를 반환
  - less than(<) : 첫 번째 연산자가 두번째 연산자보다 작다면 true를 반환
  - less than or equal to(<=) : 첫 번째 연산자가 두번째 연산자보다 작거나, 같으면 true를 반환

## 논리 연산자(Logical Operator)

- 논리 연산자는 논리식을 판단하여 true, false를 반환한다.
    - && : 논리식이 모두 참이면 true (논리 AND 연산)
    - || : 논리식이 하나라도 참이면 true (논리 OR 연산)
    - ! : 논리식의 결과를 반환 (논리 NOT 연산)
    ```java
    int a = 5;
    int b = 10;
    System.out.println(a > b && a == b + 5); // true && true => true
    System.out.println(a < b && a == b + 5); // false && true => false
    System.out.println(a > b || a == b - 5); // true && false => true
    System.out.println(a < b || a == b - 5); // false && false => false
    System.out.println(!(a < b)); // false => true
    System.out.println(!(a > b)); // true => false
    ```

## instanceof

> Object가 특정 class, interface인지 여부를 확인한다.

```java
// 표기
(Object reference variable) instanceof (class/interface type)

// 예시
class A {}

class B extends A {
}


  class Parent { }
  class Child extends Parent {
  }

  void run() {
    Parent parent = new Parent();
    Child child = new Child();

    System.out.println(parent instanceof Parent);   // true
    System.out.println(child instanceof Child);     // true

    System.out.println(child instanceof Parent);    // true
    System.out.println(parent instanceof Child);    // false
}
```
- Object type(객체 타입)을 확인하는데 사용하는 연산자, 여부를
- 객체 instanceof 객체타입 == true를 반환한다.
- 자녀객체 instanceof 부모타입 == true를 반환한다.
- 부모객체 instanceof 자녀타입 == false를 반환한다.

## assignment(=) operator (대입 연산자)

> 값을 메모리의 일부에 대입하거나 저장할 때 사용되는 연산자이다.

<img width="811" alt="스크린샷 2023-04-05 오후 9 03 32" src="https://user-images.githubusercontent.com/85725033/230074876-837523db-c67d-42b2-a6a1-ba3379421921.png">

## 화살표(->) 연산자

> 매개변수를 받아 값을 반환하는 짧은 코드 블록으로, 이름이 필요하지 않으며 메서드 본문에서 바로 구현할 수 있다.

```java
// 구현
(Parameter)->{구현코드(Lambda식)};

// 예시

// method
public int sum(int a, int b){
    return a+b;
}

// lambda
(a, b)->a + b;
```

- Java 8에 추가된 기능으로 Lambda 표현식에 사용되는 연산자이다.
- 이름이 필요하지 않아 익명함수라고도 한다.

## 3항 연산자

> 피연산자 3개를 가지는 조건 연산자이다.

```java
// 구현
조건식 ? 반환값1 : 반환값2

// 예시
int num1 = 1, num2 = 2;
int result;
result = (num1 - num2 > 0) ? num1 : num2;
System.out.println("둘 중 더 큰 수는" + result); // 둘 중 더 큰수는 2
```

- 물음표(?) 앞의 조건식에 따라 결과가 true라면 반환값1을 반환하고, false라면 반환값2를 반환한다.

## 연산자 우선 순위

<img width="811" alt="스크린샷 2023-04-05 오후 9 03 32" src="https://user-images.githubusercontent.com/85725033/230078378-31ac53d9-2be8-485b-bf08-0040a65e9187.png">


## switch 연산자

```java
// 기본 형식
switch(입력변수) {
    case 입력값1: ...
         break;
    case 입력값2: ...
         break;
    ...
    default: ...
         break;
}
```

- 입력변수의 값과 일치하는 case 입력값(입력값1, 입력값2, ...)이 있다면 해당 case문에 속한 문장들이 실행된다. 
- break는 해당 case문을 실행 한 뒤 switch문을 탈출하기 위함이다.
- 만약 break 문이 빠져 있다면 그 다음의 case 문이 실행된다.

### 추가된 기능

```java
public enum Day { SUNDAY, MONDAY, TUESDAY,
  WEDNESDAY, THURSDAY, FRIDAY, SATURDAY; 
}

// ...

  int numLetters = 0;
  Day day = Day.WEDNESDAY;
  switch(day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDATE               -> System.out.println(7);
    case THURSDATY, SATURDAY    -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
    default -> throw new IllegalStateException("Invalid day: " + day);
}
```

- 기존 switch는 break 미사용 시, 다음 case로 이어지고, 각 case마다 break를 작성해야 하는 번거로움이 있었다.
- Java 12SE부터 :대신 -> 연산자를 사용하여 break없이 한 개의 표현식은 한 개의 case로 인식할 수 있다.
- 또한 콤마(,)를 사용하여 한 case에 여러 값 입력 가능하다.

```java
int numLetters = switch(day) {
  case MONDAY, FRIDAY, SUNDAY -> 6;
  case TUESDATE               -> 7;
  case THURSDATY, SATURDAY    -> 8;
  case WEDNESDAY              -> 9;
};

T result = switch(arg) {
    case L1 -> e1;
    case L2 -> e2;
    default -> e3;
};
```

- 기존에는 switch문 내에서 새로운 로컬 변수를 사용하려면 switch 블록 내부에서만 사용하거나, 블록 외부에 변수 선언 후 사용 가능했다.
- Java 12부터 switch문을 사용하여 값 반환 가능

```java
int j = switch(day) {
  case MONDAY   -> 0;
  case THUSDAY  -> 1;
  case default  -> {
    int k = day.toString().length();
    int result = f(k);
    yield result;
  }
};
```

- Java 13에서 추가된 키워드 yield는 switch문의 case의 값을 반환한다.


## 참고자료

- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op1.html 
- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html
- http://tcpschool.com/java/java_operator_assignment - 대입 연산자
- https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html - 람다식
- https://docs.oracle.com/en/java/javase/13/language/switch-expressions.html - switch expressions
