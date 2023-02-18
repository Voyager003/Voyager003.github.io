---
layout  : wiki
title   : Java의 Map interface
summary : 
date    : 2023-01-18 00:55:58 +0900
updated : 2023-01-18 00:57:01 +0900
tag     : java hash
resource: FF/067D36-9059-4065-8B4C-C4117E5CD4B8
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## java.util.HashMap

![alt](https://techvidvan.com/tutorials/wp-content/uploads/sites/2/2020/03/collection-framework-hierarchy-in-java.jpg)

HashMap은 Java Collections Framework에 속한 구현체 클래스로 Java 2에서 정식으로 출시되었습니다. 


---

## HashMap & HashTable

---

## 동시성 이슈

![alt](https://i0.wp.com/javaconceptoftheday.com/wp-content/uploads/2020/07/HashMapVsConcurrentHashMap.png?w=744&ssl=1)

동시성은 동시에 실행되는 것처럼 보이는 것으로 SingleCore에서 Multi thread를 동작시키는 방식을 말합니다.

### Thread Safe

ConturrentHashMap의 경우 내부적 동기화로 Thread가 safe합니다. Thread Safe한다는 것은

### Null 허용 유무

HashMap은 최대 하나의 null키를 허용하고 value

###


---

## 참고자료
- https://techvidvan.com/tutorials/java-collection-framework - Java Collections 이미지
- https://d2.naver.com/helloworld/831311 - Java HashMap은 어떻게 동작하는가?
- https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html - Java 11 Hashmap
- https://javaconceptoftheday.com/hashmap-vs-concurrenthashmap-in-java/ - Hashmap vs ConcurrentHashmap