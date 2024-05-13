---
layout  : wiki
title   : Java 직렬화의 대안을 찾아라 
summary : 
date    : 2024-05-13 09:46:42 +0900
updated : 2024-05-13 11:34:17 +0900
tag     : java effectivevjava
resource: B0/22DD59-BFF4-48DC-B653-B13096695FAA
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 직렬화(Serialization)

`직렬화`는 컴퓨터 과학의 데이터 스토리지 문맥에서 데이터 구조나 오브젝트 상태를 동일하거나 다른 컴퓨터 환경에 저장(이를테면 파일이나 메모리 버퍼에서, 또는 네트워크 연결 링크 간 전송)하고 나중에 재구성할 수 있는 포맷으로 변환하는 과정이다. [^1]

각 PC의 OS마다 서로 다른 가상 메모리 주소 공간을 갖기 때문에, Reference Type의 데이터들은 인스턴스를 전달 할 수 없다. 때문에 주소값이 아닌 Byte 형태로 직렬화된 객체 데이터를 전달해야 한다.

이 때 직렬화된 데이터들은 primitive type이 되고, 이는 파일 저장이나 네트워크 전송 시 파싱이 가능한 유의미한 데이터로 변하게 된다.

## 보안적 문제

직렬화의 근본적 문제점은 공격 범위가 너무 넓고, 지속적으로 더 넓어져 방어하기 어렵다는 점이다. 예를 들어 ObjectInputStream의 readObject 메서드 호출 시 객체 그래프가 역직렬화(deserialzation)되기 때문이다.  

Serialization 인터페이스를 구현한 readObject() 메서드는 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있는 생성자로, 바이트 스트림을 역직렬화하는 과정에서 이 메서드가 그 타입들 안의 모든 코드를 수행할 수 있다. 이는 그 타입들의 코드 전체가 공격 범위에 들어간다는 의미이다. 라이브러리 코드를 살펴보자.

```java
private final Object readObject(Class<?> type)
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        if (! (type == Object.class || type == String.class))
            throw new AssertionError("internal error");

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            Object obj = readObject0(type, false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
                freeze();
            }
            return obj;
        } finally {
            passHandle = outerHandle;
            if (closed && depth == 0) {
                clear();
            }
        }
    }
    
    ...
```

readObject() 메서드 내 readObject0() 메서드를 호출하여 실제로 직렬화된 데이터를 해석하고 객체를 생성하는 작업이 이뤄지는데, 이 과정에서 직렬화된 데이터가 역직렬화되고, 필요한 객체가 생성된고, 반환되는 타입은 Object이다.

이처럼 역직렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드들을 가젯(gadget)이라 하며, 가젯 체인같은 방법으로 보안 공격이 가능하다.

때문에 Java 표준 라이브러리, Apache commons 컬렉션과 같은 서드파티 라이브러리를 비롯하여 애플리케이션 자신의 클래스들도 공격 범위에 포함된다. 모든 직렬화 가능 클래스들을 공격에 대비하도록 작성해도 여전히 취약할 수 있다.

item85에서 예시로 든 HashSet과 문자열만 사용해 만든 역직렬화 폭탄(deserialzation bomb)의 예시를 살펴보자.

```java
static byte[] bomb() {
    Set<Object> root = new HashSet<>();
    Set<Object> s1 = root;
    Set<Object> s2 = new HashSet<>();
    for (int i = 0; i < 100; i++) {
        Set<Object> t1 = new HashSet<>();
        Set<Object> t2 = new HashSet<>();
        t1.add("foo");
        s1.add(t1);
        s1.add(t2);
        s2.add(t1);
        s2.add(t2);
        s1 = t1;
        s2 = t2;
    }
    return serialize(root); // 직렬화
}
```

위의 객체 그래프는 201개의 HashSet 인스턴스로 구성되며, 각각 3개 이하의 객체 참조를 갖는다. 스트림 전체 크기는 5744바이트이지만, 역직렬화 시에 문제가 된다.

HashSet 인스턴스를 역직렬화하려면 그 원소들의 해시코드를 계산해야 하는데, 루트 HashSet에 담긴 두 원소는 각각 다른 HashSet을 2개씩을 원소로 갖는 HashSet으로 반복문에 의해 이 구조가 깊이 100단계까지 만들어진다. 때문에 hashCode() 메서드를 2^100번 넘게 호출해야된다.

역직렬화가 계속 지속되는 것도 문제지만, 잘못되었다는 신호조차 주지않고 몇 개의 객체만 생성한다고 해도 stack 깊이 제한에 걸려버린다. 

## 해결법

이처럼 직렬/역직렬화로 인한 문제들은 어떻게 대처해야할까?  

- 아무것도 역직렬화 하지 않기

신뢰할 수 없는 바이트스트림을 역직렬화하는 행위 자체가 공격에 노출되는 행위이다. 

레거시 시스템때문에 직렬화를 완전히 배제할 수 없을때의 차선책은 신뢰할 수 없는 데이터는 절대 역직렬화 하지 않는 것이다. 

직렬화를 피할수 없고 역직렬화 한 데이터가 안전한지 완전히 확신할 수 없다면 Java 9에 추가된 역직렬화 필터링(java.io.ObjectInputFilter)를 고려해보자. 

역직렬화 필터링은 데이터 스트림이 역직렬화 되기 전에 필터를 설치하는 기능으로 클래스 단위로, 특정 클래스를 받아들이거나 거부할 수 있다. '기본 수용' 모드에서는 블랙리스트에 기록된 잠재적으로 위험한 클래스를 거부하며, '기본 거부' 모드에서는 화이트리스트에 기록된 안전하다고 알려진 클래스만 수용한다. item85는 화이트 리스트 방식을 권장한다. 

- 크로스 플랫폼 구조화된 데이터 표현(cross-platform structured-data representation)

책에서 지칭하는 방법으로 직렬화로 인한 위험을 회피하는 여러 방법을 말한다.

먼저 흔히 볼 수 있는 `json`은 key-value 쌍의 집합으로 구성된 구조화된 데이터 객체로 텍스트 기반이어서 사람이 읽을 수 있는 간단한 추상화 도구이다.

`프로토콜 버퍼`는 이진 표현으로 구조화된 데이터를 직렬화하는 방식으로 텍스트 표현(pbtxt)도 지원한다.

## 참고자료

- 이펙티브자바 3판
- [^1]: 위키백과 설명

