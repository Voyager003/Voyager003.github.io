---
layout  : wiki
title   : 커스텀 직렬화 형태를 고려해보라 
summary : 
date    : 2024-05-27 09:27:40 +0900
updated : 2024-05-27 11:37:47 +0900
tag     : java effectivejava
resource: 18/7A940A-01EE-47AB-B6F6-AB32F3FAF791
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 기본 직렬화

기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 충분히 고민한 뒤에 합당한 경우에만 사용해야 한다. 일반적으로 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 직렬화를 사용해야 한다.

어떤 객체의 기본 직렬화 형태는 '그 객체를 루트로 하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩'한다. 즉, 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 객체들이 연결된 위상(topology)까지 기술한다. 하지만 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.

다음은 item87에서 설명하는 기본 직렬화 형태에 적합한 예시이다.

```java
public class Name implements Serializable {

    private final String lastName; // nullable
    private final String firstName; // nullable
    private final String middleName; // 중간 이름이 없다면 null
    
    ...
}
```

먼저 이름은 이름, 성, 중간이름 3개의 문자열로 구성되며, 앞 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다. 주의할 점은 기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject() 메서드를 제공해야 할 때가 많다. 

예시의 Name 클래스의 경우는 readObject메서드가 lastName과 firstName 필드가 null이 아님을 보장해야 한다.

### 기본 직렬화가 적합하지 않은 경우

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 생략
}
```

코드는 논리적으로 일련의 문자열을 표현하는데, 물리적으로는 문자열들을 이중 LinkedList로 연결했다. 

이 클래스에 기본 직렬화 형태를 사용한다면 각 노드의 양방향 연결 정보를 포함, 모든 엔트리를 기록하게 된다.

이처럼 '객체의 물리적 표현과 논리적 표현의 차이가 클 경우에 기본 직렬화를 사용하는 경우' 다음과 같은 문제가 발생한다.

- 공개 API가 현재의 내부 표현 방식에 영구적으로 묶임

예시의 경우, private 클래스인 StringList.Entry가 공개 API가 되어버리는데, 다음 release에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 여전히 LinkedList로 표현된 입력도 처리할 수 있어야 한다. 

- 많은 공간을 차지

예시의 직렬화 형태는 LinkedList의 모든 엔트리와 연결 정보까지 기록했지만, 엔트리와 연결 정보는 내부 구현에 해당하여 직렬화 형태에 포함할 가치가 없다. 이는 직렬화 형태가 너무 커져 디스크에 저장하거나 네트워크 전송 속도가 느려지게 된다.

- 시간이 많이 소요

직렬화 로직은 객체 그래프의 위상에 관한 정보가 없어 그래프를 직접 순회할 수 밖에 없다. 

- StackOverFlow 발생 가능성

기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 작업은 중간 정도 크기의 객체 크래프에서도 StackOverFlow를 일으킬 수 있다. (이 때, StackOverFlow를 일으키는 데 필요한 리스트의 최소 크기는 플랫폼 구현과 명령줄 플래그에 따라 달라진다.)

## 커스텀 직렬화 형태

그렇다면 위 StringList의 합리적인 직렬화 형태는 무엇일까? 

다음은 item87에서 제공하는 예시이다.

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;
    
    // 이제 직렬화 X
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // 지정한 문자열을 리스트에 추가
    public final void add(String s) { ... }
    
    /**
    * {@code StringList} 인스턴스를 직렬화
    * @serialData 이 리시트의 크기를 기록한 뒤
    * ({@coide int}), 이어서 모든 원소를
    * 순서대로 기록
    */
    private void writeObject(ObjectOutputStream s) thorws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        
        // 모든 원소를 올바른 순서로 기록
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
    }
    
    private void readObject(ObjectInputStream) thorws IOException, ClassNontFoundException {
    s.defaultReadObject();
    int numElements = s.readInt();
    
    for (int i = 0; i<numElements; i++) {
        add((String) s.readObject());
    }
}
    
...   
```

StringList의 필드 모드가 transient 되더라도  writeObject, readObject는 가장 먼저 defaultWriteObject와 defaultReadObject를 호출한다. 이 때, 인스턴스 필드가 transient라도 직렬화 명세는 defaultObject 메서드를 호출하라고 요구한다.

이렇게 하면 향후 release에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호 호환된다.

구버전 readObject 메서드에서 defaultReadObject를 호출하지 않는다면 역직렬화 시 StreamCorruptedExcpetion이 발생할 것이다. 

## 주의점

- transient 한정자
  
transient 한정자는 직렬화 대상에서 해당 필드를 제외한다.  

기본 직렬화를 수용하건 안하건 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화된다. 따라서 transient로 선언해도 되는 인스턴스 필드에는 모두 transient 한정자를 명시해야 한다. 

이는 캐시된 해시 값처럼 다른 필드에서 유도되는 필드와 JVM을 실행할 때마다 값이 달라지는 필드(native 자료구조를 가리키는 long 필드)가 이에 해당된다.

`해당 객체의 논리적 상태와 무관한 필드라고 확신하는 경우에만 transient를 생략해야한다.`

- 기본 직렬화의 경우

기본 직렬화 사용 시, transient 필드들은 역직렬화될 때 기본값으로 초기화된다. 

객체의 참조필드는 null, 숫자 기본 타입 필드는 0, boolean 필드는 false이다. 때문에 기본값을 그대로 사용해선 안된다면 readObject 메서드에서 defaultReadObject를 호출한 뒤, 해당 필드를 원하는 값으로 복원하거나, 그 값을 처음 사용할 때 초기화 하는 방법도 있다.

- 동기화 메커니즘

기본 직렬화 사용 여부와 상관없이 객체의 전체 상태를 읽는 메서드에 적용해야하는 동기화 메커니즘을 직렬화에도 적용해야 한다. 

모든 메서드를 synchronized로 선언하여 thread-safe하게 만든 객체에 기본 직렬화를 사용하려면 writeObject도 다음과 같이 선언해야 한다.

```java
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
}
```

writeObject 메서드 내에서 동기화하고 싶은 경우 클래스의 다른 부분에서 사용하는 lock 순서를 똑같이 따라야 한다. 그렇지 않다면 자원 순서 교착상태(resource-ordering deadlock)에 빠질 수 있다. 

- UID 명시

어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬버전 UID를 명시적으로 부여하자.

이는 직렬버전 UID가 일으키는 잠재적인 호환성 문제를 없애준다. 또한 UID를 명시하지 않으면 런타임에 값을 생성하느라 복잡한 연산을 수행하기 때문에 성능이 느려질 수 있다.

```java
private static final long serialVersionUID = <무작위 long>
```

직렬버전 UID 선언은 각 클래스에 위와 같이 선언해주면 된다. 이 때, 직렬버전 UID가 꼭 고유할 필요는 없다.

한편 직렬 버전 UID가 없는 기존 클래스를 구버전으로 직렬화된 인스턴스와 호환성을 유지한 채 수정하고 싶다면, 구버전에서 사용한 자동 생성된 값을 그대로 사용해야 한다.

기본 버전의 클래스와 호환성을 끊고 싶다면 단순히 직렬버전 UID의 값을 바꿔주면 된다. 이는 기존 버전의 직렬화된 인스턴스를 역직렬화할 때 InvalidClassException을 던진다. 

`구버전으로 직렬화된 인스턴스들과의 호환성을 끊는 경우를 제외하고는 직렬 버전 UID를 절대 수정해선 안된다.`

## 참고자료

- 이펙티브자바 3판

