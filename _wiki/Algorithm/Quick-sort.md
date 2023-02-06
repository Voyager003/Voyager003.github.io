---
layout  : wiki
title   : Quick Sort(퀵 정렬)
summary : 
date    : 2023-02-06 10:27:15 +0900
updated : 2023-02-06 10:28:31 +0900
tag     : java sorting-algorithm
resource: CA/2072F8-9FFA-40B9-A950-1C744B3CF243
toc     : true
public  : true
parent  : [[/Algorithm]]
latex   : false
---
* TOC
{:toc}

## Quick Sort

- divide and conquer(분할 정복)을 통해 정렬하는 알고리즘으로 과정은 다음과 같다. 
- pivot(기준점)을 정해, pivot보다 작은 data는 left, 큰 data는 right로 나눈다.
- 각 left와 right는 Recursive call을 통해 다시 동일 함수를 호출하여 위의 작업을 반복한다.
- left + pivot + right를 반환한다.

## Java로 구현한 Quick Sort

### pivot을 기준으로 나눈 뒤, 합치기

```java
public class QuickSort {
    public void split(ArrayList<Integer> data) {
        if (data.size() <= 1) {
            return;
        }
        int pivot = data.get(0);

        ArrayList<Integer> leftArr = new ArrayList<Integer>();
        ArrayList<Integer> rightArr = new ArrayList<Integer>();

        for (int idx = 1; idx < data.size(); idx++) {
            if (data.get(idx) > pivot) {
                rightArr.add(data.get(idx));
            } else {
                leftArr.add(data.get(idx));
            }
        }
        ArrayList<Integer> mergedArr = new ArrayList<Integer>();
        mergedArr.addAll(leftArr);
        mergedArr.addAll(Arrays.asList(pivot));
        mergedArr.addAll(rightArr);
    }
}
```

pivot을 주어진 ArrayList의 첫 element를 기준으로 선정하고, pivot을 기준으로 작다면 leftArr, 크다면 rightArr에 data를 삽입하는 로직이다.

정렬을 마쳤다면 leftArr, pivot, rightArr을 순서대로 ArrayList에 추가한다면 정렬된 data를 확인할 수 있을 것이다.

이제 추가적으로 Recursive call을 통해 동일 함수를 호출하도록 해야한다.

### Recursive call 적용

```java
public class QuickSort {
    public ArrayList<Integer> sort(ArrayList<Integer> data) {
        if (data.size() <= 1) {
            return data;
        }
        int pivot = data.get(0);
        
        ArrayList<Integer> leftArr = new ArrayList<Integer>();
        ArrayList<Integer> rightArr = new ArrayList<Integer>();        
        
        for (int idx = 1; idx < data.size(); idx++) {
            if (data.get(idx) > pivot) {
                rightArr.add(data.get(idx));
            } else {
                leftArr.add(data.get(idx));
            }
        }
        
        ArrayList<Integer> mergedArr = new ArrayList<Integer>();
        mergedArr.addAll(this.sort(leftArr));
        mergedArr.addAll(Arrays.asList(pivot));
        mergedArr.addAll(this.sort(rightArr));
        
        return mergedArr;        
    }
}
```

정렬이 될때까지 반복해야하기 때문에 동일 메서드(this.sort(Arr))를 통해 메서드를 호출했다. 메서드를 마치고 return하게되면 정렬된 Arr를 반환하게 되고 이를
mergedArr에 순서대로 합친다면 정렬된 결과를 얻을 수 있다.

### main 메서드에서 실행

```java
Integer[] array = {4, 1, 2, 5, 8, 7, 9, 14, 12};
ArrayList<Integer> list = new ArrayList<Integer>(Arrays.asList(array));

QuickSort quickSort = new QuickSort();
ArrayList<Integer> sortedList = quickSort.sort(list);
System.out.println("sortedList = " + sortedList); // sortedList = [1, 2, 4, 5, 7, 8, 9, 12, 14]
```

## Quick Sort의 시간 복잡도

최선의 경우, 배열이 균등하게 이등분 되어 recursive call의 depth는 k가 된다. 각 단계에서 pivot을 올바르게 위치시키기 위한 비교 연산 횟수는 평균적으로 N번 
이루어지므로 총 연산횟수는 O(kN)이다. 이때, k = logN 이므로 **O(kN) = O(NlogN)** 이다.

하지만 배열이 이미 정렬되어있는 최악의 경우, 배열에서 원소가 한개씩만 분리되어 recursive call의 depth가 N이 된다. 
각각의 단계에서 비교 연산이 평균적으로 N번 이루어지므로 총 연산횟수는 **O(N^2)** 이다.

또한 Stable을 보장하는 merge-sort와 달리 Sorting 후에 동일한 element에 대한 우선순위가 유지되지 않는 Unstable sort이다.

## 참고자료
- https://www.youtube.com/watch?v=7BDzle2n47c&t=3s&ab_channel=%EC%97%94%EC%A7%80%EB%8B%88%EC%96%B4%EB%8C%80%ED%95%9C%EB%AF%BC%EA%B5%AD - Quick Sort 설명 및 구현