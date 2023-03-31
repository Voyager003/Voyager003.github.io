---
layout  : wiki
title   : Java 연산자 
summary : 
date    : 2023-03-31 21:24:12 +0900
updated : 2023-03-31 23:31:00 +0900
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



## 참고자료

- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/op1.html 
- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/operators.html
