---
layout  : wiki
title   : Java의 제어문 
summary : 
date    : 2023-04-08 20:01:04 +0900
updated : 2023-04-08 20:30:58 +0900
tag     : java
resource: 0B/DD2CFB-C1B3-445D-A4C7-0D5D94FD5D05
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/4) 4주차 과제

## 조건문
 
> 조건식 결과에 따라 명령문 실행 여부가 결정되는 데, 조건식에는 True 혹은 False를 산출할 수 있는 연산식이나 boolean이 올 수 있다.

### if, else statement

```java
int x = 3;
if (x==3) {
    System.out.println("x는 3이다");
} else {
    System.out.println("x는 3이 아니다");
}
```

x 의 값이 3이므로 if문을 정상적으로 통과하여 "x는 3이다"를 출력한다.

### else if statement

```java
int x = 3;
if (x == 5){
    System.out.println("x = 5");
} else if (x == 3) {
    System.out.println("x = 3");
}else{
    System.out.println("wrong");
```

다중 조건으로 else if문을 통과하여 "x=3"을 출력한다.

### switch statement(선택문)

> 변수, 식의 값이 다양한 분기(branch)를 통하여 프로그램 실행 흐름을 제어하게 한다.

```java


switch (x) {
    case 1:
        System.out.println(x); // x가 1일 때 실행
        break;
    case 2:
        System.out.println(x); // x가 2일 때 실행
        break;
    case 3:
        System.out.println(x); // x가 3일 때 실행
        break;
    default:
        System.out.println(x); // x가 모든 조건에 만족 하지 않을때 
        break;
    }
```

## 반복문

### for 

```java
for(int i = 0; i < 5; i++){
    System.out.println(i);
}
```

- 시작인 0부터 5보다 작을 때 까지 출력한다면, 1에서 4까지 반복하여 출력한다.

### while

```java
while(연산식) {
    // code 
}

// 예시
while (i<5) {
    System.out.println(i);
    i++;
}
```

- i가 5보다 작을 떄 까지, i를 증가시키면서 출력하기 때문에 1에서 4까지 반복하여 출력한다.

### do while

```java
int i =0;
do {
    System.out.println(i);
    i++;
} while (i>10);
```

- 조건에 관계 없이 적어도 한번 이상 반복문을 돌기 때문에, 0을 출력하고 반복문을 마친다.


## 참고자료

- https://docs.oracle.com/javase/tutorial/java/nutsandbolts/if.html
