---
layout  : wiki
title   : Stack(스택)
summary : 
date    : 2023-01-16 09:53:15 +0900
updated : 2023-01-16 11:09:54 +0900
tag     : bigO stack java
resource: AF/60E7B1-E1A7-46E7-8C85-A0D951FA5834
toc     : true
public  : true
parent  : [[/Data-structure]]
latex   : false
---
* TOC
{:toc}

## Stack
![alt](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e4/Lifo_stack.svg/700px-Lifo_stack.svg.png)

- 한 쪽 끝(상단 그림과 같은 경우 맨 위의 data)에서만 data를 넣고 뺄 수 있는 자료구조이다.
- 가장 최근에 stack에 추가한 data가 가장 먼저 제거되는 LIFO(Last In First Out)를 따른다.
- data를 넣는 **push()** 와 data를 꺼내는 **pop()** 이라는 작업으로 data를 취급한다. 

---

## Java로 구현한 Stack (ArrayList 기준)

- push와 pop 구현

```java
import java.util.ArrayList;

public class StackEx<T> {
    
    public ArrayList<t> stack = new ArrayList<T>();
    
    public void push(T data) {
        stack.add(data);
    }
    
    public T pop() {
        if (stack.isEmpty()) {
            return null;
        }
        return stack.remove(stack.size()-1);
    }
    
    public boolean isEmpty() {
        return stack.isEmpty();
    }
}
```
push를 통해 data를 삽입하고, stack이 비어있지 않다면 가장 나중에 삽입한 값을 제거하도록 stack.size()-1로 값을 return 했다.


- main 메서드에서 실행

```java
StackEx<Integer> st = new StackEx<Integer>();
st.push(1); // 1 
st.push(2); // 1 2
System.out.println(ms.pop); // 2 -> 가장 나중에 삽입한 2를 stack에서 빼면서 값을 return
st.push(3); // 1
System.out.println(ms.pop); // 3
System.out.println(ms.pop); // 1
```


---


## java.util.Stack

- java.util 패키지에서 Stack 클래스를 제공한다.
    - push() : data를 stack 가장 위에 삽입한다. 
    - pop() : stack 맨 위의 개체를 제거하고 그 값을 반환한다.
    - peek() : stack에서 data를 제거하지 않고 맨 위의 data를 탐색 및 반환한다.
    - empty() : stack이 비어있는지 boolean으로 반환한다.
    - search(Object o) : 해당 Object의 위치를 반환한다. 이 때 Stack top 위치는 1, 해당 Object가 없다면 -1을 반환한다.

---

## Stack의 시간 복잡도

- 탐색

stack에 4개의 data가 쌓여있다고 가정한다. 첫 data가 찾아야 할 data였다면 한 번의 탐색으로 찾을 수 있겠지만, 맨 마지막 data가 
찾아야할 data라면 4번의 탐색 과정을 거쳐야 한다. 따라서 시간 복잡도는 **O(n)** 이 된다.


- 삽입 및 삭제

data 삽입 및 삭제 시, 항상 stack 맨 위의 자료를 push하거나 pop하기 때문에 한 번의 operation으로 data의 삽입 및 수행이 수행되므로 
시간 복잡도는 **O(1)** 이 된다.

---

## 참고자료
- https://en.wikipedia.org/wiki/Stack_(abstract_data_type) - 이미지 출처
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Stack.html
