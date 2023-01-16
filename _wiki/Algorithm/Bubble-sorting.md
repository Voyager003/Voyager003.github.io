---
layout  : wiki
title   : Bubble sort(버블 정렬) 
summary : 
date    : 2023-01-16 13:31:48 +0900
updated : 2023-01-16 13:32:36 +0900
tag     : algorithm java sorting-algorithm
resource: CE/3D8482-8FC4-404E-BBCE-8E90DE30A68B
toc     : true
public  : true
parent  : [[/Algorithm]] 
latex   : false
---
* TOC
{:toc}

## Sorting Alrogithm (정렬 알고리즘)

> **정렬 알고리즘**은 목록의 element를 순서대로 배치하는 알고리즘이다. 

- 특징

![alt](https://github.com/qkraudghgh/coding-interview/raw/master/Interview/question/sorting.png)

크게 Comparisons(비교) 방식과 Non-Comparisons(비-비교) 방식으로 나눌 수 있다.

Comparisons : Bubble Sort, Selection Sort, Insertion Sort, Merge Sort, Heap Sort, Quick Sort 등이 있다.

Non-Comparisons : Counting Sort, Radix Sort 등이 있다.

굉장히 많은 Sorting Algorithm이 있는데, 이는 알고리즘마다 예상되는 속도가 다르며 '**Stable**'한지 안한지에 따라 사용되어야 할
Sorting이 다를 수 있기 때문이다.

여기서 '**Stable**'이란 동일한 element가 있을 때 Sorting 전의 순서와 Sorting 후의 순서가 동일함을 보장하는 것이 Stable이다.

> Example1
> 
> [{k: 4, v: 1}, {k: 3, v: 2}, {k: 3, v: 1}, {k: 2, v: 1}, {k: 5, v: 1}] 을 stable한 sorting algorithm을 이용한다면
> 
> [{k: 2, v: 1}, {k: 3, v: 2}, {k: 3, v: 1}, {k: 4, v: 1}, {k: 5, v: 1}]가 된다. 
> 
> k가 3인 동일한 정렬 기준을 가진 Element가 2개가 있지만 input된 순서 그대로 정렬되었다.

또한 속도가 아닌 Space Complexity(공간 복잡도)가 고려 대상이 될 수 있다. 

---

## Bubble sort(버블 정렬)
> n개의 원소를 가진 Array를 정렬할 때, in-place sort로 인접한 두 data를 비교해가면서 정렬을 진행하는 방식이다.

![alt](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdF7CXf%2FbtrunN5i9wZ%2FwDQ3px86zQFffgoq8X6jk0%2Fimg.png)
 
- element의 이동이 거품이 수면으로 올라오는 듯한 모습을 보이기 때문에 지어진 이름이다.
- n개의 element에 대해 n개의 memory를 사용한다.
- data를 하나 씩 비교할 수 있어 정밀 비교가 가능하지만, n이 커질수록 비교 횟수가 많아지므로 성능면에서 안좋다.

---

## Java에서 구현

- data 삽입 및 정렬

```java
import java.util.ArrayList;
import java.util.Collections;

ArrayList<Integer> sortlist = new ArrayList<Integer>();
sortlist.add(1); // 1
sortlist.add(3); // 1 3
sortlist.add(2); // 1 3 2

for(int idx = 0; idx < sortlist.size()-1; idx++) {
    if(sortlist.get(idx) > sortlist.get(idx+1)) {
        Collections.swap(sortlist. idx, idx+1);
    }
}
System.out.println(sortlist); // 1, 2, 3
```

- 개선

위의 코드에서는 반복문이 한 번 실행될 때마다 가장 큰 숫자가 뒤에서부터 1개씩 결정된다. 이 때, 첫 번째 반복문이 끝나면 이미 {1, 2, 3}으로 정렬이 끝났기 때문에 
더 이상 정렬을 할 필요가 없어진다. 이미 정렬이 모두 마쳤다면 반복문을 더 이상 반복할 필요가 없다. data 변경(자리 바꿈) 유무를 판단하는 
boolean 변수를 추가하여 개선한다.

```java
import java.util.ArrayList;
import java.util.Collections;

public class BubbleSort {
    
    public ArrayList<Integer> sort(ArrayList<Integer> sortlist) {
        for (int idx = 0; idx < sortlist.size() - 1; idx++) {
            boolean swap = false;
            
            for (int idx2 = 0; idx2 < sortlist.size() - 1 - idx; idx2++) {
                if (sortlist.get(idx2) > sortlist.get(idx2 + 1)) {
                    Collections.swap(sortlist, idx2, idx2 + 1);
                    swap = true;
                }
            }
            
            if (swap == false) {
                break;
            }
        }
        return sortlist;
    }
}
```

반복문에서 data의 교환(자리 바꿈)이 일어난다면 swap을 true로 변경하여 data 변경이 일어났다는 것을 알려주고, data 변경이 없었다면 더 이상 반복문을 실행할
필요가 없기 때문에 break로 반복문을 탈출하고 sortlist를 return 했다. 


---

## Bubble sort의 시간 복잡도

Bubble sort의 시간 복잡도는 각 양 옆의 index를 비교하기 때문에 반복문이 2개 사용된다. 따라서 시간 복잡도는 **O(n)=n^2**이다.
모든 경우의 수를 탐색해야하는 최악의 경우는 n*(n-1)/2며, 이미 완전 정렬되어있는 최선의 경우는 O(n)이된다.

---

## 참고자료
- https://en.wikipedia.org/wiki/Sorting_algorithm
- https://github.com/qkraudghgh/coding-interview/blob/master/Interview/question/previous_interview.md
- https://dev-coco.tistory.com/160 - bubble sort 그림
- https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Algorithm