---
layout  : wiki
title   : 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
summary : 
date    : 2023-08-28 11:18:05 +0900
updated : 2023-08-28 15:14:06 +0900
tag     : java effectivejava
resource: 54/C376A9-DF85-4EDD-BA0F-3A0D1C2C98D8
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 상속의 위험성

item18에서는 상속을 염두에 두지 않고 설계하고 주의점을 문서화하지 않은 외부 클래스의 상속할 때의 위험성을 경고했다.

리마인드 해보자면 상속은 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 발생할 수 있어, 이를 고려하지 않으면 하위 클래스는 상위 클래스의 변화에 맞춰 수정해야 한다는 것이다. 때문에 상속용 클래스는 재정의할 수 있는 메서드를 내부적으로 어떻게 이용하는지 문서로 남겨야 한다. 

문서에 남겨야 할 내용은 다음과 같다.

- 재정의 가능한 메서드(public, protected 중 final이 아닌 모든 메서드)를 호출할 때 호출 순서, 각 호출 결과 처리 영향을 기술한다.

- 백그라운드 스레드, 정적 초기화 과정 등 호출할 수 있는 모든 상황을 남긴다.

- Java 8에 도입된 기능으로, 메서드 주석에 @ImplSpec를 명시하면 Javadoc 도구가 생성된다. 코드는 다음과 같다.

```java
/**
     * {@inheritDoc}
     *
     * @implSpec
     * This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * {@code UnsupportedOperationException} if the iterator returned by this
     * collection's iterator method does not implement the {@code remove}
     * method and this collection contains the specified object.
     *
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        ...
```

하지만 내부 메커니즘을 문서화 하는 것만이 상속을 위한 설계의 전부는 아니다. 

## 문서화 외 상속 설계

- 내부 동작과정 훅(hook) 선별

클래스 내부 동작과정 중간에 끼어들 수 있는 훅을 선별하여 protected 메서드 형태로 공개해야 할 수도 있다. item19는 java.util.AbstractList의 removeRange()를 예시로 든다.

```java
/** Removes from this list all of the elements whose index is between fromIndex, inclusive, and toIndex, exclusive. Shifts any succeeding elements to the left (reduces their index). This call shortens the list by (toIndex - fromIndex) elements. (If toIndex==fromIndex, this operation has no effect.)
This method is called by the clear operation on this list and its subLists. Overriding this method to take advantage of the internals of the list implementation can substantially improve the performance of the clear operation on this list and its subLists.
Params:
fromIndex – index of first element to be removed toIndex – index after last element to be removed
Implementation Requirements:
This implementation gets a list iterator positioned before fromIndex, and repeatedly calls ListIterator.next followed by ListIterator.remove until the entire range has been removed. Note: if ListIterator.remove requires linear time, this implementation requires quadratic time.
**/

protected void removeRange(int fromIndex, int toIndex) {
        ListIterator<E> it = listIterator(fromIndex);
        for (int i=0, n=toIndex-fromIndex; i<n; i++) {
            it.next();
            it.remove();
        }
    }
}
```

주석을 간단하게 해석해보면, '이 메서드는 이 목록과 그 하위 목록에 대한 clear 연산에 의해 호출됩니다. 이 메서드를 재정의하여 목록 구현의 내부를 활용하면 이 목록과 그 하위 목록에 대한 지우기 작업의 성능을 크게 향상시킬 수 있습니다.' 라고 적혀있는 것을 볼 수 있다. 

removeRange()를 제공하는 이유는 하위 클래스에서 부분 리스트의 clear 메서드를 고성능으로 만들귀 쉽게 하기 위해서이다. 메서드가 없었다면 하위 클래스에서 clear 메서드를 호출하면 제거할 원소 수의 제곱에 비례해 성능이 느려지거나 새로 구현해야 했을 것이다.

이 때, protected 메서드 하나하나가 내부 구현에 해당하므로 그 수는 가능한 적어야하며 한편으로는 너무 적게 노출해 상속으로 얻는 이점마저 없애지 않도록 주의해야 한다.

상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다. 

하위 클래스를 여러개 만들 때까지 사용되지 않는 protected 멤버는 private으로 돌리고, 배포 전에 반드시 클래스를 만들어 검증해야 한다. 

- 생성자에서 재정의 가능 메서드 호출을 금지

상속용 클래스의 생성자는 직/간접적으로 재정의 가능 메서드를 호출해서는 안된다. 

이는 상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되기 때문에, 하위 클래스에서 재정의한 메서드가 생성자보다 먼저 호출된다. 

재정의한 메서드가 하위 클래스의 생성자에서 초기화한 값에 의존하게 된다면 코드는 의도한대로 동작하지 않는다. 

```java
public class Super{

    // 생성자가 재정의 가능한 메서드 호출
    public Super(){
       overrideMe();
    }

    public void overrideMe(){

    }
}

public final class Sub extends Super {
    
    // 초기화 되지 않은 final 필드를 생성자에서 초기화
    private final Instant instant;
    
    Sub() {
       instant = Instant.now();
    }

    // 상위 클래스의 생성자 호출
    @Override 
    public void overrideMe(){
        System.out.println(instant);
    }
}
```

instant를 두 번 출력하는 것을 기대했지만, 처음 출력되는 값은 null이다. 

이는 상위 클래스의 생성자는 하위 클래스의 생성자가 인스턴스 필드를 초기화하기도 전에 overrideMe를 호출했기 때문이다. 

상속용 클래스의 생성자는 재정의 가능 메서드를 호출해선 안된다.

- Cloneable, Serializable 인터페이스는 상속에서 닫기

해당 인터페이스를 구현할 때 따르는 제약은 생성자와 비슷하다.

clone, readObject 새로운 인스턴스를 생성하기 때문에, 모두 직/간접적으로 재정의 가능 메서드를 호출해서는 안된다. 

clone의 경우, 하위 클래스의 clone 메서드가 복제본의 상태를 수정하기 전에 재정의한 메서드를 호출하게되고, readObject는 하위 클래스가 역직렬화되지 전에 재정의한 메서드부터 호출하기 때문이다. 이는 복제본 내부 어딘가에서 여전히 원본 객체의 데이터를 참조하고 있다면 원본 객체도 피해를 입는 것이다.

- Serializable을 구현한 상속용 클래스에서 주의할 점

readResolve, writeReplace 메서드를 가진다면 이는 protected로 선언해야 한다.

private으로 선언 시, 하위 클래스에서 무시되기 때문이다. 이는 상속을 허용하기 위해 내부 구현을 클래스 API로 공개하는 예 중 하나이다.

## 그 외 일반적인 구체 클래스의 경우

- 상속용으로 설계하지 않은 클래스는 상속을 금지한다.

금지하는 방법 2가지는 클래스를 final로 선언하는 방법과 모든 생성자를 private, package-private으로 선언하고 public 정적 팩토리를 만들어주는 방법이 있다. 정적 팩토리 방법은 내부에서 다양한 하위 클래스를 만들어 쓸 수 있는 유연성을 줄 수 있다. (item17)

- 상속을 꼭 허용해야 하는 경우

재정의 가능 메서드를 사용하지 않게 만들고 문서로 남긴다.

- 클래스 동작을 유지하면서 재정의 가능 메서드를 사용하는 경우

재정의 가능 메서드는 자신의 본문 코드를 private 도우미 메서드로 옮기고, 이 도우미 메서드를 호출하도록 수정한다. 무슨 의미일까?

```java
public class NonePrivate {

    public nonePrivate() {
    }
    
    public void overrideMe() {
        System.out.println("override");
    }
    
    public void do() {
        overrideMe();
    }
}   
```

위 코드는 재정의 가능 메서드가 그대로 노출되는 코드이다.

```java
public class BetterPrivate {

    public BetterPrivate() {
    }

    public void overrideMe() {
        helper();
    }

    private void helper() {
        System.out.println("override");
    }

    public void do() {
        helper();
    }
}
```

이처럼 재정의 기능 메서드의 내용을 helper()라는 도우미 메서드로 옮겼다. 기계적이긴 하지만 클래스의 동작을 유지하면서 재정의 가능한 메서드를 사용할 수 있는 방법이다.

## 참고자료

- 이펙티브 자바 3판
