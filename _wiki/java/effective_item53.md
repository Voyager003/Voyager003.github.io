---
layout  : wiki
title   : 가변인수는 신중히 사용하라 
summary : 
date    : 2024-01-22 09:28:08 +0900
updated : 2024-01-22 10:02:03 +0900
tag     : java effectivejava
resource: 4B/8BF423-EABE-4F7A-B9C8-964711DA82CC
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 가변인수 유의점 

```java
public class VarArgsExample {

    public static void printNumbers(String message, int... numbers) {
        System.out.println(message);
        for (int num : numbers) {
            System.out.print(num + " ");
        }
        System.out.println();
    }
}
```

가변인수(varargs) 메서드는 명시한 타입의 타입의 인수를 0개 이상 받을 수 있다.

위 코드에서 가변인수는 'int...'로 선언되어 있다. 이 가변인수를 포함한 메서드를 호출하면 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건넨다.

좀 더 예시를 살펴보자.

```java
static int sum(int... args){
    int sum = 0;
    for (int arg : args){
        sum += arg;
    }
    return sum;
}
```

위 코드는 입력받은 int 인수들의 합을 계산하는 가변인수 메서드다. 

sum(1, 2, 3)은 6을 반환하고, sum()은 0을 반환하게 된다.

이 때 인수가 1개 이상이어야 할 때도 있는데, 예컨대 최솟값을 찾는 메서드에서 인수를 0개만 받을 수 있도록 설계하는 것은 좋지 않다. 

인수의 갯수는 런타임에 자동으로 생성된 배열의 길이로 알 수 있다.

인수가 1개 이상이어야 하도록 0개일 때 예외를 던지는 방법은 어떨까?

```java
static int min(int... args){
    if (args.length == 0) {
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    }
    int min = args[0];
    for (int i = 1; i < args.length; i++) {
        if (args[i] < min) {
            min = args[i];
        }
    }
    return min;
}
```

이 코드의 심각한 문제는 인수를 0개만 넣어 호출하면 컴파일시점이 아닌 런타임에 실패한다는 점이다.

또한 args의 유효성 검사를 명시적으로 해야하고, min의 초깃값을 Integer.MAX_VALUE로 설정하지 않고는 명료한 for-each 문을 사용할 수 없다.

이를 개선하려면 어떻게 할까?

```java
static int min(int firstArg, int... remainArgs) {
    int min = firstArg;
    for(int arg : remainArgs) {
        if (arg < min){
            min = arg;
        }
        return min;
    }
}
```

첫 번째로 매개변수로 평범한 매개변수를, 두 번째로 가변인수를 받는다면 가변인수만을 매개변수로 받을 때의 문제점을 개선할 수 있다.

이처럼 가변인수는 인수의 갯수가 정해지지 않았을 때 유용하다. 

## 가변인수의 성능 이슈

하지만 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다.

이유는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문이다.

이 작업의 비용을 감당할 수는 없지만, 가변인수의 유연성이 필요할 때 선택할 수 있는 좋은 패턴이 있다. 

- 메서드 다중정의를 활용한 경우

예를 들어 해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 가정해보자.

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```

그렇다면 위와 같이 인수가 0개인 것부터 4개인 것까지, 총 5개의 메서드를 다중정의 하자.

이는 마지막으로 정으된 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하는 것이다. 

[item52](https://voyager003.github.io/wiki/java/effective_item52/)의 다중정의 시에 주의할 점을 염두에 둔다면 꼭 필요한 특수 상황에서 도움이 될 것이다.

## 참고자료

- 이펙티브자바 3판




