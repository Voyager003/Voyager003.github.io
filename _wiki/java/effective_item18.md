---
layout  : wiki
title   : 상속보다는 컴포지션을 사용하라 
summary : 
date    : 2023-08-19 23:25:17 +0900
updated : 2023-08-19 23:52:34 +0900
tag     : java effectivejava
resource: D7/02FCB9-5D6F-475B-BBA3-6014399DE22E
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 상속보다 컴포지션

상속은 코드를 재사용하는 강력한 수단이지만 항상 최선은 아니다. 

잘못 사용하면 상속이 객체의 캡슐화를 깨뜨려 객체의 유연성을 해치는 설계를 하게될 수 있으며, 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 발생할 수 있다. 그러므로 항상 설계자는 항상 확장을 충분히 고려하고 문서화 해두지 않으면 하위 클래스는 상위 클래스의 변화에 맞춰 수정해야한다.

item18에서 제시하는 예시를 살펴보자.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {

    // 추가된 원소
    private int addCount = 0;

    public InstrumentedHashSet(){
    }

    public InstrumentedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }

    @Override 
    public boolean add(E e){
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c){
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount(){
        return addCount;
    }
}
```

잘 구현된 것처럼 보이지만 문제점을 살펴보자.

addAll() 호출 시, add를 사용해 구현되어 addCount값이 중복해서 더해지는 결과를 가져오는 코드가 된다. 이는 내부 구현 방식에 해당하는 부분을 고려하여 수정하더라도 다음 릴리즈 버전에서 이 내부 구현이 유지될지 알 수 없다.

addAll()을 다른식으로 재정의하는 방법도 있지만, 이는 시간이 더 들고 오류를 내거나 성능저하를 불러올 수 있다. 또한 하위 클래스에서 접근이 불가한 private 필드를 써야하는 상황이라면 이 방식으로는 구현 자체가 불가능하다.

하위 클래스가 깨지기 쉬운 또 하나의 이유는 다음 릴리즈에서 상위 클래스에 새로운 메서드가 추가된다면, 하위 클래스에서 재정의하지 못한 메서드를 사용해 허용되지 않은 원소를 추가할 수 있게 된다. 다음 릴리즈에서 상위 클래스에 추가된 새 메서드가 내가 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입이 다르다면 컴파일조차 불가하다.

이러한 문제를 어떻게 해결할까?

## 컴포지션(Composition)을 사용한 방법

**Composition(컴포지션)**은 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하는 방법을 통해 기능을 확장시키는 것이다.

새로운 클래스의 인스턴스 메서드들은 private 필드로 참조하는 기존 클래스의 대응하는 메서드(forwarding method)를 호출해 그 결과를 반환하며, 이를 forwarding(전달)이라 한다. 이렇게 구현하면 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어날 수 있으며, 기존 클래스에 새로운 메서드가 추가되더라도 영향을 받지 않는다.

코드로 살펴보자.

```java
// 래퍼 클래스 
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}


// 재사용할 수 있는 전달 클래스
import java.util.Collection;
import java.util.Iterator;
import java.util.Set;

public class ForwardingSet<E> implements Set<E> {

    // private 필드로 기존 클래스의 인스턴스 참조
    private final Set<E> s;
    public ForwardingSet(Set<E> s){
        this.s = s;
    }

    @Override
    public int size() {
        return s.size();
    }

    @Override
    public boolean isEmpty() {
        return s.isEmpty();
    }

    @Override
    public boolean contains(Object o) {
        return s.contains(o);
    }

    @Override
    public Iterator<E> iterator() {
        return s.iterator();
    }

    @Override
    public Object[] toArray() {
        return s.toArray();
    }

    @Override
    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean add(E e) {
        return s.add(e);
    }

    @Override
    public boolean remove(Object o) {
        return s.remove(o);
    }

    @Override
    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    @Override
    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    @Override
    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    @Override
    public void clear() {
        s.clear();
    }
}

// 
```

ForwardingSet은 Set 인터페이스를 구현현했고, Set의 인스턴스를 인수로 받는 생성자를 생성했다. 임의의 Set에 기능을 덧 씌워 새로운 Set으로 만든 것이 이 클래스의 핵심이다.
상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자에 대응하는 생성자를 별도로 정의해줘야한다. 하지만, 컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용할 수 있다.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<> cmp);
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

다른 Set인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet과 같은 클래스를 래퍼 클래스(wrapper class)라고 하며, 다른 Set에 기능을 덧 씌운다는 뜻에서 Decorator pattern(데코레이터 패턴)이라고 한다. 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우에 위임에 해당한다.

래퍼 클래스는 단점이 거의 없으나 콜백 프레임워크와는 어울리지 않는다는 점은 주의해야한다. 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출 때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 몰라 this(자신)의 참조를 넘기고, 콜백 때 래퍼가 아닌 내부 객체를 호출하게된다. 전달 메서드가 성능에 주는 영향이나 래퍼 객체가 메모리 사용량에 주는 영향은 실전에서 별다른 영향이 없다고 밝혀졌다.

## 그렇다면 상속은 언제?

상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 사용해야 한다.(B is A)

클래스 B가 클래스 A를 상속하려고 할 때 클래스 B가 클래스 A라고 확신할 수 없다면 상속해서는 안된다. (Stack과 Properties 는 원칙을 위반한 클래스이다.)

컴포지션 대신 상속을 사용하기로 결정하기 전에 확장하려는 클래스의 API에 아무런 결함이 없는지 확인하고, 그 결함이 하위 클래스에도 전파되도 괜찮은지 확인해야 한다.

## 참고자료 

- 이펙티브 자바 3판
- https://www.youtube.com/watch?v=dJ5C4qRqAgA&ab_channel=%EC%9A%B0%EC%95%84%ED%95%9C%ED%85%8C%ED%81%AC - 조영호님의 우아한 객체지향




