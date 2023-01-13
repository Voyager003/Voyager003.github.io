---
layout  : wiki
title   : Queue(큐) 
summary : 
date    : 2023-01-13 10:33:01 +0900
updated : 2023-01-13 15:12:46 +0900
tag     : bigO queue java
resource: 0A/303AD7-D49E-498F-BF73-BB5B4D0F2D98
toc     : true
public  : true
parent  : [[/Data-structure]]
latex   : false
---
* TOC
{:toc}

## Queue
![image](https://upload.wikimedia.org/wikipedia/commons/thumb/5/52/Data_Queue.svg/440px-Data_Queue.svg.png)

- **선형 자료구조**이다. 
- 먼저 삽입한 data가 먼저 나오는(FIFO, First In First Out)구조로 저장한다.
- 매표소에서 표를 사기위해 줄선 사람들 중, 먼저 줄을 선 사람이 먼저 나가는 상황가 유사하다.
- Queue의 맨 뒤에 Data를 삽입하는 Enqueue, 맨 앞의 Data를 삭제하는 Dequeue라는 작업이 있다.

---

## Java로 구현한 Queue (ArrayList 기반)

- Enqueue와 Dequeue
```java
public class Queue<T> {
    private ArrayList<T> queue = new ArrayList<T>();
    
    public void enqueue(T item) {
        queue.add(item);
    }
    
    public T dequeue() {
        if (queue.isEmpty()) {
            return null;
        }
        return queue.remove(0);
    }
    
    public boolean isEmpty() {
        return queue.isEmpty();
    }
```

- main 메서드에서 출력
```java
Queue<Integer> q = new Queue<Integer>();
q.enqueue(1); // 1
q.enqueue(2); // 1, 2
q.enqueue(3); // 1, 2, 3
System.out.println(q.dequeue); // 2, 3
System.out.println(q.dequeue); // 3
```

---

## java.util.Queue
- Java는 기본적으로 java.util 패키지에서 Queue를 제공한다.
- Enqueue에 해당하는 기능으로 add(), offer() 메서드를 제공한다.
- Dequeue에 해당하는 기능으로 poll(), remove() 메서드를 제공한다.
- Queue에서 Data 생성을 위해서는 LinkedList 클래스를 사용해야한다.

```java
Queue<T> queue = new LinkedList<T>();
```
Queue는 항상 첫 번째 저장된 Data를 삭제하므로, ArrayList와 같은 Array 기반의 자료구조 사용 시 빈 공간을 채우기 위해 Data의 복사가 발생하기 때문에 비효율적이기 때문이다.

---

## Queue의 시간 복잡도

- 탐색

100개의 data가 존재할 때 10번 째 data를 삭제한다고 하면, 100개 중 30번 째까지 찾아갸아 햐기 때문에 시간복잡도는 **O(n)**이다.

- 삽입 및 삭제

data가 삽입될 때 맨 뒤의 data가 삽입되는 연산만 수행되기 때문에 시간 복잡도는 **O(1)**이다.
삭제도 마찬가지로 동일한 연산이 수행되므로 시간 복잡도는  **O(1)**이다.


---

## 참조자료
- https://en.wikipedia.org/wiki/Queue_(abstract_data_type)
- https://www.coursera.org/lecture/data-structures/queues-EShpq
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Queue.html
