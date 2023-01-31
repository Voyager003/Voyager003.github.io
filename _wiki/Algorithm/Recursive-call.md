---
layout  : wiki
title   : Recursive call(재귀 호출)
summary : 
date    : 2023-01-31 10:10:25 +0900
updated : 2023-01-31 10:12:04 +0900
tag     : java stack 
resource: CA/01F4E8-DC14-4E04-8D57-F12F935C3E7D
toc     : true
public  : true
parent  : [[/Algorithm]]
latex   : false
---
* TOC
{:toc}

## Recursive call

- 재귀 호출은 함수 내에서 함수가 자기 자신을 다시 호출하는 것을 말한다.
- 재귀 호출을 고려할 때에는 최악의 경우에 수행할 연산 횟수를 계산하여 재귀 호출을 사용해도 될지 확인하는 **경우의 수 계산**
- 재귀가 호출됐을 때 **수행할 동작**
- 재귀가 반복하지 않도록 한 시점에 재귀함수를 끊어야하는지 **탈출 조건**이 필요하다.

---

## Java로 구현한 재귀 호출

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = sc.nextInt();
        sc.close();
        System.out.println(fibonacci(n));
    }

    static int fibonacci(int n) {
        // 재귀 탈출 조건
        if (n == 0) {
            return 0;
        }
        if (n == 1) {
            return 1;
        }
        // 수행 동작
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
}
```

Beakjoon에 재귀 호출 예시에 적절한 피보나치 문제이다.

재귀 용법의 경우 Stack 자료구조에 기반을 두는데, 만약 탈출 조건없이 수행 동작이 계속 된다면 fibonacci 메서드가 계속 호출되면서 call Stack에 쌓이게 된다.

![alt](https://miro.medium.com/max/1400/1*7VhC2dGxtwMCtbMDuyTMmQ.webp)

이렇게 새로운 함수를 계속 호출하면서 stack에 메모리가 계속적으로 저장되는데, stack 메모리에 더 이상 가용 메모리가 없을 경우 Stack overflow가 발생한다.

이를 해결하기 위해 n==0이면 0을 n==1이라면 1을 반환하는 탈출 조건을 넣어 재귀 함수를 끝낼 수 있도록했다.

---

## 함께 보기
- https://www.acmicpc.net/problem/10870 - BeakJoon 문제

---

## 참고자료
- https://javascript.plainenglish.io/understanding-javascript-execution-context-call-stack-and-stack-overflow-157b7b358e88 - 스택 이미지