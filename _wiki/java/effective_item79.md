---
layout  : wiki
title   : 과도한 동기화는 피하라 
summary : 
date    : 2024-03-21 13:48:14 +0900
updated : 2024-04-01 11:40:06 +0900
tag     : java effectivejava
resource: 04/F658FC-3771-4219-BB31-7E2052164D38
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 동기화 시 주의점

- 동기화 메서드 혹은 블록 안에서 제어를 클라이언트에 양도하는 경우

```java
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); // notifyElementAdded 호출
        }
        return result;
    }
} 
```

코드는 Set을 감싼 래퍼 클래스로, Set에 원소가 추가되면 클라이언트는 알림을 받는다.

observer(관찰자)는 addObserver, removeObserver 메서드를 호출하여 구독 신청 및 해지를 한다. 두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

```java
@FunctionalInterface public interface SetObserver<E> {
    // ObservableSet에 원소가 더해지면 호출됨
    void added(ObservableSet<E> set, E element);
}
```

ObservableSet은 잘 동작할 것으로 예상된다. 이제 0부터 99까지 출력하는 다음 프로그램을 살펴보자.

```java
public static void main(String[] args) {
	ObservableSet<Integer> set = new ObservableSet<>(New HashSet<>());
	
	set.addObserver(new SetObserver<Integer>() {
		public void added(ObservableSet<Integer> s, Integer e) {
			System.out.println(e);
			if (e == 23) {
                s.removeObserver(this);
            }
		}
	});

	for (int i = 0; i < 100; i++) {
		set.add(i);
    }
}
```

0부터 99까지 출력하다가 값이 23이면 자기 자신을 제거하는 observer를 추가한다면?

0부터 23까지 출력한 뒤에 observer 자신을 제거한 뒤 종료할 것을 예상했지만 실제로는 ConcurrentModificationException을 던진다. 

이유는 observer의 added 메서드 호출이 일어난 시점은 notifyElementAdded가 observer의 리스트를 순회하는 도중이기 때문이다. 

added 메서드가 ObservableSet의 removeObserver 메서드를 호출하고, 다시 observers.remove 메서드를 호출할 때 리스트에서 원소를 제거하는 시점에 리스트를 순회하는 도중이기 때문에 예외를 던진 것이다.

ElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 하는 것을 보장하지만 콜백을 거쳐 되돌아온 후에 수정하는 것까지는 못하게 된다.

- 백그라운드 스레드를 사용하는 경우

```java
set.addObserver(new SetObserver<Integer>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService exec = Executors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get();
            } catch (ExecutionException | InterruptedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});
```

코드는 구독해지(removeObserver)를 직접 호출하지 않고 ExecutorService를 사용해 다른 스레드에게 넘긴 경우이다.

프로그램에서 예외는 발생하지 않지만, 백그라운드 스레드가 s.removeObserver를 호출하면 observer를 lock하려고 시도하지만 메인 스레드가 lock을 쥐고 있어 lock을 얻을 수 없다.

동시에 메인 스레드는 백그라운드 스레드가 observer를 제거하기만을 기다리기 때문이 교착상태에 빠지게 되는 것이다.

이처럼 동기화된 영역 안에서 override(재정의)된 메서드를 호출하거나 클라이언트가 넘겨준 함수 객체를 호출하는 것을 `alien method(외계인 메서드)` 라 한다.

다행이 이런 문제는 alien method를 동기화 블록 바깥으로 옮기면 된다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
```

notifyElementAdded에서 observer 리스트를 복사해서 사용하면 lock 없이도 안전하게 순회 가능하다.

이렇게 동기화 영역 바깥에서 호출되는 alien method를 open call(열린 호출)이라 한다.

더 나은 방법은 Java의 [util.concurrent](https://docs.oracle.com/javase/7/docs/api/index.html?java/util/concurrent/package-summary.html) 라이브러리를 사용하는 것이다.

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
		observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
		return observers.remove(observer);
}

private void notifyElementAdded(E element) {
		for (SetObserver<E> observer : observers)
				observer.added(this, element);
}
```

CopyOnWriteArrayList는 ArrayList를 구현한 클래스로 내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현되어 있다. 이는 내부 배열이 절대 수정되지 않아 순회 시 lock이 필요없어 빠르며 수정할 일이 드물고 순회만 빈번히 일어난다면 observer 리스트 용도로는 최적이다.

## 성능 및 선택지

멀티코어가 일반화 된 현재는 과도한 동기화가 초래하는 진짜 비용은 lock을 획득하는 데 드는 CPU 시간이 아니라 `지연 시간`이다. 

race condition에서 낭비하는 시간, 즉 병렬로 실행할 기회를 읽고 모든 코어가 메모리를 일관되게 보기 위한 지연 시간이 진짜 비용인 것이다.

item79는 가변 클래스를 작성 시, 두 가지 선택지를 제공한다.

- 동기화를 전혀 하지않고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화 하기

- 동기화를 내부에서 수행하여 thread-safe한 클래스로 만들기(객체 전체에 lock을 거는 것보다 동시성을 월등히 개선할 수 있는 경우에 해당)

클래스를 내부에서 동기화하기로 했다면 lock splitting(분할), lock striping(스트라이핑), nonblocking concurrency control(비차단 동시성 제어)를 동원해 동시성을 높일 수 있다. 
여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화해야 한다. 그런데 클라이언트가 여러 스레드로 복제돼 구동되는 상황이라면 다른 클라이언트에서 이 메서드를 호출하는 걸 막을 수 없으니 외부에서 동기화할 방법이 없다.


## 정리

동기화의 기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것!

lock 획득 -> 공유 데이터 검사 -> 필요하다면 수정, lock을 해제하는 과정이 오래 걸린다면 동기화 영역 바깥으로 옮기는 방법(open call)을 찾아보자.

합당한 이유가 있을 때 내부에서 동기화하고, 동기화 여부를 문서에 명확히 밝히자. 

## 참고자료

- 이펙티브자바 3판
- https://inelpandzic.com/articles/effective-java-tldr-concurrency/

