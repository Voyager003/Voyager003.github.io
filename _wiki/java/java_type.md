---
layout  : wiki
title   : Java 데이터 타입과 변수, 배열
summary : 
date    : 2023-03-27 21:51:56 +0900
updated : 2023-03-28 23:48:17 +0900
tag     : java
resource: 00/B85F19-BB5C-4844-9CF6-008B1631F558
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/2)를 정리하고자 작성한 글

## Primitive type과 Reference type

![alt](https://1.bp.blogspot.com/-bpbdwrvRSaw/VflqVT6GaxI/AAAAAAAADxU/BSi7bF7ef34/w602-h249/Primitive%2Bvs%2BReference%2BType%2BJava.png)

### Primitive Type(원시(기본) 자료형)

> 선언 시, 사용하는 메모리가 정해져있고, Stack 영역에 값을 바로 저장하는 자료형

- byte : 8비트 부호 있는 2의 보수 정수이다. 최소값은 -128이고 최대값은 127(포함)이다.

- short : 16비트 부호 있는 2의 보수 정수이다. 최소값은 -32,768이고 최대값은 32,767(포함)이다.

- int : 32비트 부호 있는 2의 보수 정수이며 최소값은 -2^31 이고 최대값은 2^31-1이다. 
  - Java SE 8 이상에서는 int데이터 유형을 사용하여 최소값이 0이고 최대값이 2 32-1인 부호 없는 32비트 정수를 나타낼 수 있다.

- long : 64비트 2의 보수 정수입니다. 부호 있는 long의 최소값은 -2^63 이고 최대값은 2^63-1이다. 
  - Java SE 8 이상에서는 long데이터 유형을 사용하여 최소값이 0이고 최대값이 2 64 -1인 부호 없는 64비트 길이를 나타낼 수 있다.

- float : 단정밀도 32비트 IEEE 754 부동 소수점이다. 
  - 값의 범위는 이 논의의 범위를 벗어나지만 Java 언어 사양의 부동 소수점 유형, 형식 및 값 섹션에 지정되어 있다.

- double : double데이터 유형은 배정밀도 64비트 IEEE 754 부동 소수점이다.
  - 값의 범위는 이 논의의 범위를 벗어나지만 Java 언어 사양의 부동 소수점 유형, 형식 및 값 섹션에 지정되어 있습니다. 

- boolean : boolean데이터 유형에는 두 가지 값만 사용할 수 있다.
  - true및 false. 참/거짓 조건을 추적하는 단순 플래그에 이 데이터 유형을 사용하라.

- char : 단일 16비트 유니코드 문자이다.
  - 최소값 '\u0000'(또는 0)과 최대값 '\uffff'(또는 65,535 포함)이 있다.

### Reference Type(참조 자료형)

> 변수 값을 메모리에 직접 저장하지 않고, 값을 가지고 있는 Heap 영역의 주소값을 가지는 자료형

- Primitive type을 제외한 자료형으로 배열(array), 열거(enum), 클래스, 인터페이스가 이에 속한다.

## Literal(리터럴)

> 소스 코드의 고정된 값을 표현한 값

```java
boolean result = true;
char capitalC = 'C';
byte b = 100;
short s = 10000;
int i = 100000;
```

### 정수 리터럴

```java
int a = 15; // 10진수 리터럴 15
int b = 015; // 0으로 시작시 8진수, 10진수값으로 13출력
int c = 0x15; // 0x으로 시작시 16진수, 10진수값으로 21출력
int d = 0b0101; // 0b로 시작시 2진수, 10진수값으로 5출력
```

### 실수 리터럴

```java
double d1 = 0.123;
double d2 = 123E-4;
float f = 0.1234f;
double d3 = .123D;
```


- 소수점 형태 혹은 지수 형태로 표현된 값을 의미한다.
- float 혹은 double 타입으로 컴파일된다.
- 숫자 뒤 f 혹은 d를 명시적으로 붙인다.(float은 필수이나, double 타입의 경우 생략 가능)

### 문자 및 문자열 리터럴

```java
String st = "str";
char a = 'H';
char b = \uae00; // UNICODE
```
- 인용부호로 문자를 표현된 값을 의미한다.
- char 타입 리터럴은 작은 따옴표('), String 타입 리터럴은 큰 따옴표(")를 사용한다.
- Escape 문자를 가진다. -> \b(백스페이스), \t(탭), \n(라인 피드), \f(폼 피드), \r(캐리지 리턴), \\(백슬래시)

### 논리 타입(boolean) 리터럴

- true 혹은 false로 나타낸다.

### 그 외

- null은 특수 리터럴로 분류된다.
- class 리터럴은 <type name>.class의 형태로 나타내며, class 타입 자체를 나타내는 객체의 참조값을 반환한다.

## 변수 선언 및 초기화

```java
String str1;
int a;
int b,c,d;
```

- 변수 선언은 타입(String, int)과 변수 명(str1, a)로 나타낸다.

```java
public class Variables {
    int instanceVar; // 0으로 초기화되는 인스턴스 변수
    static int classVar; // 0으로 초기화되는 클래스 변수
    int initInstanceVar = 10; // 명시적 초기화
    static int initClassVar = 10; // 명시적 초기화

    void method(int num) { // 매개변수는 초기화 할 수 없고, 전달받는 값을 사용만 가능
        int a; // 선언 가능
        // int b = a; 자동으로 초기화 되지 않는다.
        a = 10; // 선언을 미리 했다면 초기화 O
        int b = a; // 선언과 동시에 초기화
  }
}
```

- 인스턴스 변수(Instance Variables, Non-Static Fields) : static 키워드 없이 선언된 필드로, 각 인스턴스 별로 다른 값을 가질수 있다.
- 클래스 변수(Class Variables, Static Fields) : static 키워드와 함께 선언된 필드로, 모든 인스턴스들이 공유하는 값이다. class 하나에 한 값이기 때문에 클래스 변수라 한다.
- 지역 변수(Local Variables) : 메서드 영역에서 사용되는 변수로, 다른 클래스에서 접근할 수 없는 변수이다.
- 매개 변수(Parameters) : 메서드의 인자로 전달되는 변수이다.

## 변수의 Scope와 Lifetime

```java
public class Variables {   
    static int cv; // 클래스변수
    int iv;        // 인스턴스변수

    void method(int param) { 
        int lv = 0; // 지역변수
    } 
}
```

- 인스턴스 변수 : 인스턴스가 생성되었을 때 생성되며, 인스턴스가 참조하는 변수가 없을 때 소멸한다(GC).
- 클래스 변수 : class가 메모리에 올라갈 때 생성되며, Java 프로그램 종료 시 소멸한다.
- 지역 변수 : 변수 선언문이 수행되었을 때 생성되고, 메서드 영역 블럭을 벗어날 때 소멸한다.
- 매개 변수 : 메서드 호출 시 생성되며, 메서드 종료시 소멸한다.

## Type 변환과 Casting, Type promotion

Type 변환은 크게 2가지 종류로 Promotion과 Casting 방식이 있다.

### Promotion(Widening Conversion)

```java
int num = 100;
double doubleNum = num;        // 100.0
double sum = num + doubleNum;  // 200.0
```

- int형의 변수를 double형의 변수에 대입하거나, 산술 시 Error가 없고, 출력 시에 형 변환이 일어난다.
- 대입 혹은 산술 시, Compiler가 자동으로 타입을 변환을 한다.(boolean 제외)
- Data 손실이 없는 한에서 자동으로 형 변환이 일어난다.
- 묵시적 형변환이라고도 한다.

### Casting(Narrow Conversion)

```java
long l = 1234;
int i = (int) l;
```

- 사용자가 타입 캐스트 연산자를 통해 직접 형 변환 하는 것
- 변수 앞에 casting 하려는 type의 변수를 명시한다. 
- 이명시적 형변환이라고도 한다.

## 1차원 및 2차원 배열 선언

### 1차원 배열 선언 및 초기화

```java
int [] arr1; // 선언
int arr2[]; // 선언
int [] arr3 = new int[10]; // 초기화 1
int [] arr4 = {1, 2, 3, 4, 5}; // 초기화 2
```

- 선언은 위의 두가지 방식 모두 가능하다.
- 초기화 1의 경우, 해당 크기만큼 배열의 element가 초기화되어 생성된다. 
- 배열 각 element는 index로 접근 가능하다.
- 초기화 2의 경우, 중괄호 사이 선언된 element의 수로 크기가 결정된다.

### 2차원 배열 선언 및 초기화

```java
String[][] names1 = {{ "asd", "dfg" },{ "zxc", "cvb"}};
String[][] names2 = new String[10][];
String[][] names3 = new String[10][10];
```

- 배열 element의 길이는 초기화시 지정해주지않아도 되지만, 명시하지 않는다면 null로 초기화된다.
- element의 index로 접근 가능하다.

## Type 추론과 var

> 변수의 타입을 명시적으로 적어주지 않고, 컴파일러가 자동으로 변수의 타입을 대입된 리터럴로 추론하는 것

### < > Operator

```java
Map<String, List<String>> m1 = new HashMap<String,List<String>>();

Map<String, List<String>> m1 = new HashMap<>();
```

- Java 7의 < > 오퍼레이터는 표현식에서 Generic의 타입 매개변수를 생략할 수 있다. 

### Lambda

```java
Predicate<String> lamb = (String x) -> x.length() > 0;

Predicate<String> lamb = x -> x.length() > 0;
```

- Java 8에 등장한 Lambda 표현식은 타입을 생략하고, 메서드를 하나의 식으로 표현할 수 있다.

### 지역 변수(Local) 타입 추론

```java
var str = "it's java";

if(str instanceof String) {
    System.out.println("...")
}
```

- Java 10에 도입된 var는 변수 선언 시, 타입 생략이 가능하며, 컴파일러가 타입을 추론한다.
- var는 초기화값이 있는 지역 변수로만 선언이 가능하다.
- 이미 타입이 정의되어 있기 때문에, 다른 타입의 재할당을 허용하지 않는다.


## 참고자료

- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html - Java docs
- https://google.github.io/styleguide/javaguide.html - java guide
- https://docs.oracle.com/javase/specs/jls/se10/html/jls-5.html
- https://www.geeksforgeeks.org/type-conversion-java-examples/
- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/variables.html
- https://javarevisited.blogspot.com/2015/09/difference-between-primitive-and-reference-variable-java.html#axzz7xAJ439kK - Type 이미지


