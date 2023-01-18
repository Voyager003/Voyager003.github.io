---
layout  : wiki
title   : Array(배열)
summary : 
date    : 2023-01-10 09:47:30 +0900
updated : 2023-01-19 00:45:16 +0900
tag     : Array bigO java
resource: F1/091EF2-3CD4-4051-B86E-8E6D2FCB1590
toc     : true
public  : true
parent  : [[/Data-structure]]
latex   : false
---
* TOC
{:toc}

## 자료구조로써 Array(배열)

> 같은 Type의 변수들을 인접한 Memory 위치에 있는 Data를 모아놓은 집합

- 필요성과 특징
    - element가 일렬로 나열되어 있는 '**선형 자료구조**' 이다.
    - **중복을 허용**하고, **순서**가 있다.
    - 같은 종류의 Data를 효율적으로 관리하기 위해서 사용한다.
    - 이를 통해, 첫 Data의 위치에서 상대적인 위치(index)로 Data를 접근하여 빠른 접근이 가능하다.
    - 다만 이러한 특징으로 Data의 추가 및 삭제가 어려워, 미리 Array의 최대 길이를 지정해야 한다는 단점이 있다.
    

---

## Java에서 Array

### 기본 문법

Java는 기본 문법으로 Array를 지원한다.

```java
int [] x = {length};
int [] ex1 = {1,2,3,4,5};
System.out.println(ex1[0]); // 1
```

기본 Array는 객체를 생성할 때 Array의 크기를 선언하도록 되어있다.

### java.util.Arrays

Java는 Array를 보다 쉽게 다루기 위한 Class를 제공한다.

```java
import java.util.Arrays;
System.out.println(Arrays.toString(ex1)); 
```

- 기본 문법에서 선언한 int형 Array를 String으로 변환한다. 

### List와 ArrayList

- java.util의 Interface List<E>

```java
List <Integer> list1 = new ArrayList<Integer>();
```
List는 Interface로 모든 method가 추상 메서드인 경우를 의미한다.

List라는 Interface 안에 ArrayList 클래스가 있다. (도형 - 정사각형의 관계라고 비유할 수 있다.)

List로 선언된 변수는 필요에 따라 다음과 같이 다른 List의 클래스를 쓸 수 있는 **구현상의 유연성을 제공**한다.


```java
List <Integer> list1 = new ArrayList<Integer>();
list1 = new LinkedList<Integer>();
```

- java.util.ArrayList<E>

```java
ArrayList <Integer> = new ArrayList<Integer>();
```
ArrayList는 Interface인 List와 달리 Class이다. ArrayList class로 구현한 Array는 data를 추가하는데로 size가 동적으로 
늘어난다. 이 때, 배열 방이 다 차게되면(확장한 공간을 모두 사용) Array의 크기를 두 배로 늘리게 된다. 두 배로 늘린 Array에 기존 배열에
있었던 data를 모두 복사하게 되는데 이를 **Doubling(더블링)** 이라 한다. 

코드를 통해 살펴보자.

```java
public class Main {

  public static class ArrayList {
      
    private Object[] data;
    private int size; // array의 크기
    private int index; // 다음 data를 추가할 위치를 기억하고 있는 index
    
    public ArrayList() {
      this.data = new Object[this.size];
      this.size = 1;
      this.index = 0; // 
    }

    // data 추가
    public void add(Object object) { 
      if (this.index == this.size - 1) { // 배열방이 모두 찼다면
        doubling(); // doubling을 실행 
      }
      data[this.index] = object; // 공간이 있다면, 배열방 맨 끝에 data를 추가
      this.index++; // data 추가로 index 1 증가
    }
  }
  
  // doubling 과정
  private void doubling() {
    this.size = this.size * 2; // 현 배열방을 2배로 늘림
    Object[] newData = new Object[this.size];
    for (int i = 0; i < data.length; i++) {
      newData[i] = data[i]; // doubling을 통해 배열방을 현 data로 할당
    } 
    this.data = newData;
  }

  public Object get(int i) throws Exception {
    if (i > this.index - 1) {
      throw new Exception("ArrayIndexOutOfBound");
    } else if (i < 0) {
      throw new Exception("Negative Value");
    }
    return this.data[i];
  }

  public void remove(int i) throws Exception {
    if (i > this.index - 1) {
      throw new Exception("ArrayIndexOutOfBound");
    } else if (i < 0) {
      throw new Exception("Negative Value");
    }
    
    for (int x = i; x < this.data.length - 1; x++) {
      data[x] = data[x + 1]; // 삭제할 data 기준으로 한 칸씩 앞으로 
    }
    this.index--; // index를 1 감소
  }
}

// main 메서드에서 실행

ArrayList al = new ArrayList();
al.add("0");
al.add("1");
al.add("2");
al.add("3");
al.add("4");
al.add("5");
al.add("6");
System.out.println(al.get(5)); // 5
al.remove(5);
System.out.println(al.get(5)); // 6
```

---

## Array의 시간 복잡도

- 접근

접근은 Array 내에서 n번 째 index에 해당하는 값을 찾아내는 연산이다.
Array의 접근은 **O(1)** 의 시간 복잡도를 갖는다. 찾고자 하는 값이 몇 번째 index에 있는지 알고 있따면 빠른 검색 속도를 갖는다. Array의 첫 번째 변수에는 시작
주소값이 저장되고, A[n]의 값을 찾아가기 위해 시작주소값에서 단순 사칙연산이 수행되기 때문이다.


- 탐색

Array의 탐색은 **순차 탐색**이다. index를 알지 못할 때 원하는 값을 찾기 위해 Array를 배열을 모두 확인해야 한다.

![Image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FebR7Gv%2FbtqUWtaBdb1%2FPsZJhqmDd2BUy8b9OpJfw1%2Fimg.jpg)

위에서 A[3]의 값을 찾기위해서는 A[0]부터 순서대로 A[3]까지 순서대로 탐색한다. 즉 **O(n)** 의 시간 복잡도를 가진다.



- 삽입 및 삭제

![imgae](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FctbBif%2FbtqU0xXor35%2Ftv2mfnmBKKakoWTN1FokYk%2Fimg.jpg)

Array에 빈 공간이 있다는 것을 전제로, A[6]에 5라는 값을 삽입하거나 삭제하고 싶을 때, 해당 index를 알고 있다면 O(1)의 시간 복잡도를 가지지만, 해당 index를 
탐색해야 한다면 탐색의 시간복잡도인 **O(n)** 에 해당한다.

---


## 참고자료
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Array.html
- https://docs.oracle.com/javase/specs/jls/se7/html/jls-10.html
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ArrayList.html
- https://www.baeldung.com/java-collections-complexity
- https://www.youtube.com/watch?v=I4_uFyjWZn4&t=60s&ab_channel=%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%8C%80%ED%95%9C%EB%AF%BC%EA%B5%AD

