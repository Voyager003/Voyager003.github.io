---
layout  : wiki
title   : 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라 
summary : 
date    : 2023-12-04 09:49:19 +0900
updated : 2023-12-04 11:09:48 +0900
tag     : java effectivejava
resource: 71/D527DF-8C40-4258-B16F-AB66C21CC1C9
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 마커 인터페이스(marker interface)

마커 인터페이스는 내부에 메서드나 상수가 없는 인터페이스로, 객체에 대한 런타임 유형 정보를 제공하므로 컴파일러와 JVM은 객체에 대한 추가 정보를 갖게 된다. [^1]

Serializable나 Cloneable과 같은 인터페이스가 좋은 예시로 Serialziable 인터페이스의 경우, 이를 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 직렬화할 수 있다고 알려주게 된다. API를 살펴보면

```java
public interface Serialziable {
} // 마커 인터페이스로 메서드나 상수가 없다.

// ObjectOutputStream.class

public class ObjectOutputStream
    extends OutputStream implements ObjectOutput, ObjectStreamConstants
{   
    ...

    private void writeObject0(Object obj, boolean unshared)
        throws IOException
        { 
        ...
        if (obj instanceof String) {
                writeString((String) obj, unshared);
            } else if (cl.isArray()) {
                writeArray(obj, desc, unshared);
            } else if (obj instanceof Enum) {
                writeEnum((Enum<?>) obj, desc, unshared);
            } else if (obj instanceof Serializable) {
                writeOrdinaryObject(obj, desc, unshared);
            } else {
                if (extendedDebugInfo) {
                    throw new NotSerializableException(
                        cl.getName() + "\n" + debugInfoStack.toString());
                } else {
                    throw new NotSerializableException(cl.getName());
                }
            }
        } finally {
            depth--;
            bout.setBlockDataMode(oldMode);
        }
        ...
    }
}
```

wrtieObject0() 메서드를 보면, 인자로 전달받은 객체가 Serializable 인터페이스를 구현했는지(Serializable의 인스턴스인지 확인)확인하고, 마킹되어있지 않으면 예외(NotSerializableException)를 던진다.

[마커 애노테이션](https://voyager003.github.io/wiki/java/effective_item39/)의 등장으로 마커 인터페이스가 사용되지 않을 것 같지만 두 가지 면에서 마커 애노테이션보다 낫다. item41이 말하는 장점은 무엇인지 살펴보자.

- 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애노테이션은 그렇지 않다.

마커 인터페이스를 사용했다면 런타임에 발견될 오류를 컴파일 타임 시점에 잡아낼 수 있다. 

위에서 살펴본 ObjectOutputStream의 writeObject 메서드는 당연히 인수로 받은 인스턴스가 Serialziable을 구현했을 거라고 가정한다. 이 때 Serialziable이 아닌 Object 인스턴스를 받도록 설계되었다. 

즉, 직렬화할 수 없는 객체를 넘기더라도 런타임 시점에야 문제를 확인할 수 있따는 것이다. 마커 인터페이스를 사용하는 주 이유는 컴파일타임의 오류 검출인데 이점을 살리지 못한 것이다.

- 적용 대상을 더 정밀하게 지정할 수 있다.

@Target 메타 애노테이션을 Element.TYPE으로 선언한 애노테이션은 모든 타입에 명시할 수 있다. 명시 타입을 더 세밀하게 제한하지는 못한다는 것이다.

이 때 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있다면?

마커를 인터페이스로 정의했다면 마킹하고 싶은 클래스에서 그 인터페이스를 구현하여 인터페이스의 하위타입을 보장할 수 있다.

반대로 마커 애노테이션이 인터페이스보다 나은 점은 무엇일까?

- 애노테이션 시스템의 지원을 받을 수 있다.

Spring 프레임워크의 경우 애노테이션을 적극 활용하는 프레임워크로 마커 애노테이션을 사용하는 방향이 일관성을 지키는 데 유리할 것이다.

## 경우에 따른 마커 선택

확실한 것은 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수 등)에 마킹해야 할 때는 클래스와 인터페이스만이 구현 및 확장이 가능하기 때문에 애노테이션을 쓸 수밖에 없다. 

마커를 클래스나 인터페이스에 적용해야 한다면 마킹된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면 마커 인터페이스를 사용해야 한다. 

이런 메서드를 작성할 일이 없다고 확신한다면 마커 애노테이션이 나은 선택이 될 것이다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판
- https://www.baeldung.com/java-marker-interfaces - makrker interfaces in java

## 주석

[^1]: A marker interface is an interface that doesn’t have any methods or constants inside it. It provides run-time type information about objects, so the compiler and JVM have additional information about the object.
