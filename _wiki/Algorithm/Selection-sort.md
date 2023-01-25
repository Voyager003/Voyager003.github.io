---
layout  : wiki
title   : Selection sort(선택 정렬)
summary : 
date    : 2023-01-25 13:27:09 +0900
updated : 2023-01-25 14:31:16 +0900
tag     : sorting-algorithm java algorithm
resource: 7E/6D5432-76C4-4A51-97F7-91DCF50ED194
toc     : true
public  : true
parent  : [[/Algorithm]]
latex   : false
---
* TOC
{:toc}

## Selection sort

<p align="center"><img src="https://upload.wikimedia.org/wikipedia/commons/9/94/Selection-Sort-Animation.gif?20080103121010"></p>

- 선택 정렬은 다음과 같은 순서를 반복하면서 정렬하는 알고리즘이다.
    - 주어진 data에서 최솟값을 찾는다.
    - 해당 최솟값을 data 맨 앞에 위치한 값과 교체한다.
    - 맨 앞의 위치를 뺀 나머지 data를 동일한 방법을 반복한다.

---

## Java에서 구현한 Selection sort

```java
import java.util.ArrayList;
import java.util.Collections;

public class SelectionSortEx1 {
    public ArrayList<Integer> sort(ArrayList<Integer> scList) {
        int lowest;
        for (int curr = 0; curr < scList.size(); curr++) {
            lowest = curr;
            for (int idx = curr + 1; idx < scList.size(); idx++) {
                if (scList.get(lowest) > scList.get(idx)) {
                    lowest = idx;
                }
            }
            Collections.swap(scList, lowest, curr);
        }
        return scList;
    }
}
```

list에서 가장 작은 data를 기억하기 위한 lowest와 현재의 값을 가르키는 curr를 선언했다. 

순회 시, lowest를 현재 위치(curr)을 기준으로 그 다음 index부터 비교하여 lowest가 더 크다면 lowest를 현 index를 가르키도록 한다.

이 후, Colletions의 swap 메서드를 통해 lowest와 curr의 위치를 바꿔준다.



---

## Selection sort의 시간 복잡도

버블 정렬과 동일하게 현 index와 다음 index를 비교해야하기 때문에 이중 반복문을 사용하게 된다. 따라서 시간 복잡도는 **o(n^2)** 가된다.

모든 경우의 수를 탐색해야하는 최악의 경우는 n*(n-1)/2며, 이미 완전 정렬되어있는 최선의 경우는 O(n)이 된다.

---

## 참고자료
- https://commons.wikimedia.org/wiki/File:Selection-Sort-Animation.gif - 선택 정렬 그림
- https://www.youtube.com/watch?v=uCUu3fF5Dws&ab_channel=%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%8C%80%ED%95%9C%EB%AF%BC%EA%B5%AD - selection sort 설명 
