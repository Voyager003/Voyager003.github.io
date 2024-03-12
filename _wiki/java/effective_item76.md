---
layout  : wiki
title   : 가능한 한 실패 원자적으로 만들라 
summary : 
date    : 2024-03-12 09:30:50 +0900
updated : 2024-03-12 10:46:12 +0900
tag     : java effectivejava
resource: 92/A415A4-6C49-4C50-A27C-C7E1F294D1A5
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 실패 원자적(Failure atomic)

호출된 메서드가 실패하더라도 해당 객체가 메서드 호출 전의 상태를 유지하는 것을 `실패 원자적` 이라 한다.

item76은 메서드를 실패 원자적으로 만드는 방법을 몇 가지 제시한다.

### 불변 객체로 설계

불변 객체는 객체 생성 이후에 내부 상태가 변하지 않는 객체를 말한다.

Java에서 변수들은 기본적으로 가변적이지만, final 키워드를 사용하면 참조값을 변경 못하도록 만들어 불변성을 확보할 수 있다. 

final 키워드를 사용해 불변 클래스를 만드려면 다음 원칙을 따라야 한다.

- 확장할 수 없도록 final 키워드를 클래스에 선언
- 필드에 직접 접근하지 못하도록 모든 필드를 private으로 선언
- setter 메서드를 제공하지 않기
- 모든 가변 필드를 final로 선언하여 필드의 값이 한 번만 할당되도록 하기
- 깊은 복사(deep copy: '실제 값'을 새로운 메모리 공간에 복사)를 수행하는 생성자 메서드로 모든 필드를 초기화
- 실제 객체 참조를 반환하는 대신 복사본을 반환하려면 getter 메서드를 제공하여 객체 복제를 수행

이를 만족하는 코드를 작성해보면 다음과 같다.

```java
public final class FinalClassEx {

	// final 필드 키워드 선언
	private final int id;
	private final String name;
	private final HashMap<String,String> testMap;

	// getter 제공
	public int getId() {
		return id;
	}

	public String getName() {
		return name;
	}

	public HashMap<String, String> getTestMap() {
		return (HashMap<String, String>) testMap.clone();
	}

	// 깊은 복사를 수행하는 생성자 메서드
	public FinalClassExample(int i, String n, HashMap<String,String> hm){
		System.out.println("Performing Deep Copy for Object initialization");

		// "this" keyword refers to the current object
		this.id=i;
		this.name=n;

		HashMap<String,String> tempMap=new HashMap<String,String>();
		String key;
		Iterator<String> it = hm.keySet().iterator();
		while(it.hasNext()) {
			key=it.next();
			tempMap.put(key, hm.get(key));
		}
		this.testMap=tempMap;
	}
}
```

해당 메서드가 실패하더라도 새로운 객체가 만들어지지 않을 수는 있지만, 기존 객체가 불안정한 상태에 빠지는 일을 방지할 수 있다.

### 매개변수의 유효성 검사

작업 수행에 앞서 매개변수의 유효성을 검사하는 방법이 있다.

```java
public final class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
    
        // 매개변수의 유효성을 검사
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name must not be null or empty");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age must be a non-negative number");
        }
        
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```

유효성을 검사하여 객체 생성과 상태 설정이 원자적으로 이루어지므로 객체가 불안정한 상태에 빠지는 일을 방지할 수 있다. 

여기서 추가적으로 실패 가능성이 있는 모든 코드를 객체의 상태를 바꾸는 코드보다 앞에 배치하는 방법도 있다. 계산을 수행하기 전에 인수의 유효성을 검사할 수 없을 경우 덧붙여 사용할 수 있는 기법으로 TreeMap의 API에서 이를 살펴볼 수 있다.

```java
/**
     * Returns this map's entry for the given key, or {@code null} if the map
     * does not contain an entry for the key.
     *
     * @return this map's entry for the given key, or {@code null} if the map
     *         does not contain an entry for the key
     * @throws ClassCastException if the specified key cannot be compared
     *         with the keys currently in the map
     * @throws NullPointerException if the specified key is null
     *         and this map uses natural ordering, or its comparator
     *         does not permit null keys
     */
    final Entry<K,V> getEntry(Object key) {
        // Offload comparator-based version for sake of performance
        if (comparator != null)
            return getEntryUsingComparator(key);
        Objects.requireNonNull(key);
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
        return null;
    }
    
    ...
```

API에서 `지정한 키를 현재 맵에 있는 키와 비교할 수 없는 경우 classCastException을 반환한다.` 라고 명시되어 있다. (Java 17)

TreeMap에서 원소를 추가할 때, 다른 타입의 원소를 추가하려고 할 때 Tree를 변경하기 전에 해당 원소가 들어갈 위치를 찾는 과정에서 ClassCastException을 던지게 되는 것이다.

### 객체의 임시 복사본에서 작업을 수행한 뒤, 성공한다면 원래 객체와 교체

데이터를 임시 자료구조에 저장해 작업하는 것이 더 빠를 경우 적용하기 좋은 방식.

### 실패를 가로채는 복구 코드를 작성해 작업 전 상태로 되돌리는 법

주로 디스크 기반의 내구성을 보장해야하는 자료구조에 사용

## 주의점

- 동시성 문제 
  
두 스레드가 동기화없이 같은 객체를 동시에 수정한다면 일관성이 깨질 수 있다.

- 비용이 큰 연산

실패 원자적으로 만들 수 있더라도 비용이나 복잡도도를 고려해야 한다.

- API 설명에 명시

메서드 명세에 기술한 예외라면 예외가 발생해도 객체의 상태는 메서드 호출 전과 똑같이 유지되어야 하는 것이 기본 규칙. 이를 지키지 못하면 API 설명에 명시해야 함.

## 참고자료

- 이펙티브자바 3판
- https://www.baeldung.com/java-immutable-object - java immutable
- https://www.digitalocean.com/community/tutorials/how-to-create-immutable-class-in-java
  
