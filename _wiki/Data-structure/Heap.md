---
layout  : wiki
title   : Heap(힙) 
summary : 
date    : 2023-01-30 09:45:21 +0900
updated : 2023-01-30 09:45:50 +0900
tag     : Data-structure java tree
resource: B8/7C8426-EEBA-459E-B763-2D83CBAE8017
toc     : true
public  : true
parent  : [[/Data-structure]]
latex   : false
---
* TOC
{:toc}

## Heap

![alt](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c4/Max-Heap-new.svg/440px-Max-Heap-new.svg.png)

- Heap은 마지막 Level을 제외하고 모든 Level이 완전히 채워져있는 *complete binary tree(완전 이진 트리)* 를 기본으로 한 자료구조이다. 
- data의 최댓값과 최솟값을 빠르게 찾기위해 고안됐다.
- Heap 구조
  - Heap은 최댓값을 구하기 위한 Max Heap과 최솟값을 구하기 위한 Min Heap으로 분류된다.
  - Max Heap에서 각 node의 값은 해당 node의 자식 노드가 가진 값보다 크거나 같다
  ```
            20                     
           /  \
          12   8
         /  \    
        5    6      // max 
  ```
  - Min Heap의 경우 각 node의 값은 해당 node의 자식 node가 가진 값보다 크거나 작다.
  
---

## Java로 구현한 Max Heap (ArrayList)

### Heap class

```java
import java.util.ArrayList;

public class Heap {
    public ArrayList<Integer> heapArr = null;
    
    public Heap(Integer data) {
        heapArr = new ArrayList<Integer>();
        heapArr.add(null);
        heapArr.add(data);
    }
}
```

Array의 index는 0부터 시작하지만, root node의 index를 1로 지정하여 편하게 볼 수 있도록 생성자로 null을 삽입하고 data를 삽입했다.

### data 삽입

```java
import java.util.ArrayList;

public class Heap {
  public ArrayList<Integer> heapArr = null;

  public Heap(Integer data) {
    heapArr = new ArrayList<Integer>();

    heapArr.add(null);
    heapArr.add(data);
  }

  public boolean insert(Integer data) {
    if (heapArr == null) {
      heapArr = new ArrayList<Integer>();

      heapArr.add(null);
      heapArr.add(data);
    } else {
      heapArr.add(data);
    }
    return true;
  }
}

// main 메서드에서 실행
Heap heapEx1 = new Heap();
heapEx1.insert(1);
heapEx1.insert(2); 
heapEx1.insert(3);
heapEx1.insert(4);
heapEx1.insert(5);

System.out.println(heapTest.heapArr); // {null, 1, 2, 3, 4, 5};
```

출력 시, ArrayList 형식으로 data를 삽입한 순서대로 출력되겠지만, complete binary tree의 구조를 생각해본다면

```
            1                     
           / \
          2   3
         / \    
        4   5  
```

다음과 같이 heap이 구성되었을 것이다.

### 특정 node 위치 구분

```
             1                     
           /   \
          2      3
         / \    / \
        4   5  6   7
```

data 삽입만으로는 Heap이라고 하기에는 부족하기 때문에, index를 통해 node의 위치를 알아내도록 했다. 

위의 그림으로 보자면 parent node index를 기준으로 왼쪽 child node라면 parent node index * 2를 하고
오른쪽 chile node라면 parent node index*2 + 1가 되도록 설계한다.

```java
import java.util.ArrayList;

public class Heap {
  public ArrayList<Integer> heapArr = null;

  public Heap(Integer data) {
    heapArr = new ArrayList<Integer>();

    heapArr.add(null);
    heapArr.add(data);
  }

  public boolean move_up(Integer inserted_idx) {
    if (inserted_idx <= 1) {
      return false;
    }
    Integer parent_idx = inserted_idx / 2;
    if (this.heapArr.get(inserted_idx) > this.heapArr.get(parent_idx)) {
      return true;
    } else {
      return false;
    }
  }

  public boolean insert(Integer data) {
    Integer inserted_idx, parent_idx;

    if (heapArr == null) {
      heapArr = new ArrayList<Integer>();

      heapArr.add(null);
      heapArr.add(data);
      return true;
    }

    this.heapArr.add(data);
    inserted_idx = this.heapArr.size() - 1; 

    while (this.move_up(inserted_idx)) {
      parent_idx = inserted_idx / 2;
      Collections.swap(this.heapArr, inserted_idx, parent_idx);
      inserted_idx = parent_idx;
    }
    return true;
  }
}
```

move_up이라는 메서드를 통해 node를 순회하도록 하고 node의 위치를 교체할 필요가 없다면 while문을 탈출하여 max heap이 적용된 complete binary tree를 이루도록 한다.

### data 삭제 

```java
public boolean move_down(Integer popped_idx) {
        Integer left_child_popped_idx, right_child_popped_idx;

        left_child_popped_idx = popped_idx * 2;
        right_child_popped_idx = popped_idx * 2 + 1;

        // 왼쪽 chile node가 없을 때
        if (left_child_popped_idx >= this.heapArr.size()) {
            return false;
            // 오른쪽 chile node가 없을 때
        } else if (right_child_popped_idx >= this.heapArr.size()) {
            if (this.heapArray.get(popped_idx) < this.heapArr.get(left_child_popped_idx)) {
                return true;
            } else {
                return false;
            }
            // 왼쪽, 오른쪽 chile node가 모두 있을 때
        } else {
            if (this.heapArr.get(left_child_popped_idx) > this.heapArr.get(right_child_popped_idx)) {
                if (this.heapArr.get(popped_idx) < this.heapArr.get(left_child_popped_idx)) {
                    return true;
                } else {
                    return false;
                }
            } else {
                if (this.heapArr.get(popped_idx) < this.heapArr.get(right_child_popped_idx)) {
                    return true;
                } else {
                    return false;
                }
            }
        }
    }

    public int pop() {
        Integer returned_data, popped_idx, left_child_popped_idx, right_child_popped_idx;

        if (this.heapArr.size() <= 1) {
            return null;
        }

        returned_data = this.heapArr.get(1);
        this.heapArr.set(1, this.heapArr.get(this.heapArray.size() - 1));
        this.heapArr.remove(this.heapArr.size() - 1);
        popped_idx = 1;

        while (this.move_down(popped_idx)) {
            left_child_popped_idx = popped_idx * 2;
            right_child_popped_idx = popped_idx * 2 + 1;

            if (right_child_popped_idx >= this.heapArr.size()) {
                if (this.heapArr.get(popped_idx) < this.heapArr.get(left_child_popped_idx)) {
                    Collections.swap(this.heapArr, popped_idx, left_child_popped_idx);
                    popped_idx = left_child_popped_idx;
                }
            } else {
                if (this.heapArr.get(left_child_popped_idx) > this.heapArr.get(right_child_popped_idx)) {
                    if (this.heapArr.get(popped_idx) < this.heapArr.get(left_child_popped_idx)) {
                        Collections.swap(this.heapArray, popped_idx, left_child_popped_idx);
                        popped_idx = left_child_popped_idx;
                    }
                } else {
                    if (this.heapArr.get(popped_idx) < this.heapArr.get(right_child_popped_idx)) {
                        Collections.swap(this.heapArr, popped_idx, right_child_popped_idx);
                        popped_idx = right_child_popped_idx;
                    }
                }
            }
        }
        return returned_data;
    }
```

---

## Heap의 시간 복잡도

binary tree의 depth를 h로 표기한다고 가정하고 n개의 node를 가지는 heap에 data 삽입 혹은 삭제시, 
최악의 경우 첫 root node부터 가장 끝의 leaf node까지 비교하게 된다. 이는 h = logn에 가까워지는가 tree 구조와 동일하게 시간 복잡도는 **O(logn)** 이 된다.

---

## 참고자료
- https://en.wikipedia.org/wiki/Heap_(data_structure) - Heap 이미지
- https://youtu.be/jfwjyJvbbBI - Heap 설명
- https://www.geeksforgeeks.org/max-heap-in-java/ - java 코드 구현