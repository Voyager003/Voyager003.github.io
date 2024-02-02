---
layout  : wiki
title   : 라이브러리를 익히고 사용해라 
summary : 
date    : 2024-01-30 14:06:20 +0900
updated : 2024-02-02 11:19:46 +0900
tag     : java effectivejava
resource: E3/F5A69B-6661-4799-84F3-97E862AED333
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 라이브러리의 필요성

```java
// 59-1
static Random rnd = new Random();

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```

무작위 정수를 생성하는 코드를 작성한다고 해보자.

이 코드에는 몇 가지 문제가 있다.

- n이 그리 크지않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다는 점

- n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다.(n값이 크다면 더 빈번함)

```java
// 59-2
public static void main(String[] args) {
    int n = 2 * (Integer.MAX_VALUE / 3);
    int low = 0;
    for(int i = 0; i < 1000000; i++)
        if(random(n) < n/2)
            low++;
    System.out.println(low);
}    
```

코드는 선택한 범위에서 무작위 수 백만개를 생성한 뒤에, 그 중 중간 값보다 작은 것이 몇개인지를 출력한다.

random() 메서드가 이상적으로 동작한다면 50만 개가 출력돼야 하지만, 무작위로 생성된 수 중 2/3 가량이 중간값보다 낮은 쪽으로 쏠려 666,666에 가까운 정숫값을 얻게된다.

여기서 살펴볼 수 있는 결함은 코드 59.1의 rnd.nextInt()가 반환값을 Math.abs에서 음수가 아닌 정수로 매핑하기 때문에 지정한 범위 바깥의 수가 종종 튀어나온다는 것이다.

이런 결함을 해결하기 위해서는 정수론, 2의 보수 계산 등을 고려해야 하지만, 이런 문제점을 Random.nextInt(int)가 해결해놨기 때문에 이를 사용하면 된다.

이처럼 **표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 앞서 사용한 다른 프로그래머들의 경험을 활용** 할 수 있다.

하지만 Java 7부터는 Random을 ThreadLocalRandom으로 대체하여 사용하는 것이 좋다. 고품질의 무작위 수를 생성할 뿐만 아니라 속도도 더 빠르다. 포크-조인 풀이나 병렬 스트림에서는 SplittableRandom을 사용하라.

### 추가적인 라이브러리의 이점

- 핵심적인 일과 크게 관련없는 문제를 해결하느라 시간을 허비할 필요 없음.

- 따로 시간을 들이지 않아도 성능이 지속해서 개선됨. -> 라이브러리 제작자들의 노력

- 기능이 점점 추가됨

- 작성한 코드가 낯익은 코드가 됨 -> 다른 개발자들이 더 읽기 좋고 유지보수하기 좋은 코드가 됨.

## Java 메이저 릴리즈(major release)

Javasms 메이저 릴리즈마다 많은 기능이 추가된다.

https://www.java.com/releases/

현재 기준으로는 Java 21.0.2까지 공개됨.

## 참고자료

- 이펙티브자바 3판




