---
layout  : wiki
title   : 복잡도 
summary : 
date    : 2023-01-09 15:34:38 +0900
updated : 2023-01-09 22:52:46 +0900
tag     : algorithm bigO
resource: 5F/CC3F64-BC9E-440B-890E-D5B99B3F0728
toc     : true
public  : true
parent  : [[/Algorithm]]
latex   : false
comment : true
---
* TOC
{:toc}

## 복잡도
> 알고리즘의 복잡도는 Computational Complexity(계산 복잡도)로 알고리즘을 (계산 가능한 기계를 통해) 실행하기 위한 Resource(자원)을 의미한다.
 

- 시간 복잡도

 시간 복잡도(Time-complexity)는 문제를 해결하는데 걸리는 시간과 입력의 함수 관계를 가르킨다. 쉽게 풀면 Data의 갯수가 증가함에 따라 어떤 알고리즘을 동작시키는데 실행시키는 Operation(작업)의 시행 횟수라고 생각할 수 있다. 
 
자료의 갯수 n, 시행 횟수를 c로, 0 이상의 자연 수n과 시행 횟수 c의 관계를 T로 정의하고
자료 갯수의 집합을 N, 그에 따른 실행 횟수의 집합을 C라고 정의하면 그에 따른 함수 T는

> T : N -> C (N={n|n≥0, n∈N}, C={c|c≥0, c∈N})

와 같다.


- 공간 복잡도

 공간 복잡도(Space-complexity)는 문제를 해결하는데 프로그램이 얼마나 많은 Memory를 차지하는지를 분석하는 방법으로, 무어의 법칙으로 인한 컴퓨터 및 Memory 성능의 발전으로 Memory의 제약이 사라지자 알고리즘에서는 시간 복잡도를 더 우선시한다.
 

- 시간 복잡도의 주요 요소

 결론부터 말하자면 반복문(loop)이 가장 중요한 요소이다. 입력의 크기가 커지면 커질수록 반복문이 알고리즘의 수행 시간을 지배한다.
 

---
 
## 알고리즘 성능 표기법

- **Big-O (빅-오) 표기법 : O(N)**
  - 알고리즘 최악의 실행 시간을 표기
  - '가장 많고 일반적'으로 많이 사용
  - 아무리 최악의 상황이라도, 이 정도의 성능은 보장한다는 의미


- Ω(오메가) 표기법 : Ω(N)
  알고리즘 최상의 실행 시간을 표기


- Θ(세타) 표기법 : Θ(N)
  알고리즘 평균 실행 시간을 표기
  
---

## Big-O 표기법 (점근적 상한 표기법)

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F99EF1E395C7EB4B601)

- 사용 이유
  - Big-O 표기법은 알고리즘 효율성을 상한선 기준으로 표기하기 때문.
  - 알고리즘 효율성은 값이 클수록(그래프가 위로 향할수록) 비효율적임을 의미한다.
  
- O(입력)
    - 입력 n에 따라 결정되는 시간 복잡도 함수
    - O(1), O(log n), O(n), O(n log n), O(n^2), O($2^n$), O(n!) 등으로 표기한다.
    - 입력 n 의 크기에 따라 기하급수적으로 시간 복잡도가 늘어날 수 있다
    - O(1) < O(log n) < O(n) < O(nlog n) < O(n^2) < O(2^n) < O(n!)순서로 성능이 저하된다.


---

## 참고자료
- https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84_%EB%B3%B5%EC%9E%A1%EB%8F%84
- https://youtu.be/6Iq5iMCVsXA
