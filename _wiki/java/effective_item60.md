---
layout  : wiki
title   : 정확한 답이 필요하다면 float과 double은 피하라 
summary : 
date    : 2024-02-03 20:23:37 +0900
updated : 2024-02-03 20:56:07 +0900
tag     : java effectivejava
resource: A1/83B10E-4E16-4C55-8D10-D3F751E5D536
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## float, double의 용도

float과 double은 과학, 공학 계산요이으로 설계되었다. -> 부동소수점 연산, 정밀한 근사치 계산

```java
System.out.println(1.03 - 0.42); // 0.6100000000000001
```

1.03달러 중 42센트를 사용하는 경우를 출력하는 예시인데, 위와 같은 값이 나온다.

이는 원하는 결과가 아니다. 이러한 금융 계산에서는 BigDecimal, int, long을 사용해야 한다.

### BigDecimal

```java
BigDecimal funds = new BigDecimal("1.00");
```

원하는 값을 얻을 수는 있지만 BigDecimal은 기본 타입보다 쓰기가 불편하고, 더 느리다는 단점이 있다.

### int, long

BigDecimal의 대안으로 int 혹은 long 원시 타입을 쓸 수도 있다. 

이는 다룰 수 있는 값의 크기가 제한되고 소수점을 직접 관리해야 하지만, 정확한 값을 얻을 수 있다.

## 참고자료

- 이펙티브자바 3판

