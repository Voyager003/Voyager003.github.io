---
layout  : wiki
title   : readObject 메서드는 방어적으로 작성하라 
summary : 
date    : 2024-06-03 09:42:14 +0900
updated : 2024-06-03 11:20:14 +0900
tag     : java effectivejava
resource: 84/DC3925-4E9B-49E2-96F5-692553C9C4FA
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## readObject 주의점 

```java
// 방어적 복사를 사용하는 불변 클래스
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end   종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException     start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    this.start + "가 " + this.end + "보다 늦다.");
    }

    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    @Override public String toString() { return start + " - " + end; }

    ...
}
```

위 코드는 불변인 날짜 범위 클래스를 만들기 위해 Date 필드를 사용했는데, 불변식을 유지하기 위해 생성자와 접근자에서 Data 객체를 방어적으로 복사했다.

이 클래스를 직렬화하기로 결정했다고 하자. Period 객체의 물리적 표현이 논리적 표현과 부합하여 기본 직렬화 형태를 사용해도 나쁘지 않아보인다. 그렇다면 implements Serializable을 명시한다면 끝낼 수 있지 않을까?

하지만 이렇게 해서는 이 클래스의 주요 불변식을 더는 보장하지 못하는데, 이유는 readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문이다.

readObject는 매개변수로 바이트 스트림을 받는 또 다른 public 생성자이다. 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화하여 만들어지는데, 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건넨다면 정상적인 생성자로 만들어낼 수 없는 객체를 생성해낼 수 있다. 

때문에 다른 생성자와 같은 수준으로 주의를 기울여야 한다. 인수가 유효한지 검사해야하고 필요하다면 매개변수를 방어적으로 복사해야한다. 그렇지 않다면 공격자는 해당 클래스의 불변식을 깨뜨릴 수 있다.

단순하게 Period 클래스에 implements Serializable만 추가했다고 해보자.

```java
public class BogusPeriod {
    // 진짜 Period 인스턴스에서는 만들어질 수 없는 바이트 스트림
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
        0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
        ... 생략
    }

    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }

    static Object deserialize(byte[] sf) {
        try (ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(sf)) {
            try (ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
                return objectInputStream.readObject();
            }
        } catch (IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

serializedForm을 초기화하는데 사용한 바이트 배열 리터럴은 정상적인 Period 인스턴스를 직렬화한 뒤에 수정하여 만들었다. 프로그램을 실행하게되면

```java
Fri Jan 01 12:00:00: PST 1999 - Sun Jan 01 12:00:00 PST 1984
```

위와 같이 출력된다. 이는 Period를 직렬화할 수 있도록 선언한 것만으로 클래스의 불변식을 깨뜨리는 객체를 만들 수 있게 된 것이다.

### 해결법

위 문제를 해결하려면 Period의 readObject 메서드가 defaultReadObject를 호출한 뒤, 역직렬화된 객체가 유효한지 검사해야한다. 검사에 실패하면 InvalidObjectException을 던지게 하여 잘못된 역직렬화가 발생을 막을 수 있다.

```java
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    s.defaultReadObject();    
        
    // 불변식을 만족하는지 검사
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException(start + " after " + end);
    }
}
```

이상의 작업으로 허용되지 않은 Period 인스턴스를 생성하는 것은 막을 수 있지만 추가적인 문제점이 있다.

정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다. 

```java
public class MutablePeriod {
    // Period 인스턴스
    public final Period period;

    // 시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;

    // 종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;

    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);

            // 유효한 Period 인스턴스를 직렬화
            out.writeObject(new Period(new Date(), new Date()));

            /*
             * 악의적인 '이전 객체 참조', 내부 Date 필드로의 참조를 추가
             */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 참조 #5
            bos.write(ref); // 시작(start) 필드
            ref[4] = 4; // 참조 #4
            bos.write(ref); // 종료(end) 필드

            // Period 역직렬화 후 Date 참조를 '훔친다.'
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```

공격자가 ObjectInputStream에서 Period 인스턴스를 읽은 뒤 스트림 끝에 추가된 악의적인 객체 참조를 읽어 Period의 내부 정보를 얻는 코드이다. 이 참조값으로 얻은 Date 인스턴스들을 수정할 수 있기때문에 Period 인스턴스는 더이상 불변이 아니게된다.


```java
    public static void main(String[] args) {
        MutablePeriod mp = new MutablePeriod();
        Period p = mp.period;
        Date pEnd = mp.end;

        // 시간을 되돌리자!
        pEnd.setYear(78);
        System.out.println(p);

        // 60년대로 돌아가자!
        pEnd.setYear(69);
        System.out.println(p);
    }
}

// 실행 결과
// Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
// Wed Nov 22 00:21:29 PST 2017 - Sat Nov 22 00:21:29 PST 1969
```

이처럼 Period 인스턴스를 불변식을 유지한 채 생성됐지만, 의도적으로 내부의 값을 수정할 수 있다. 

문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히하지 못했기 때문이다. 

**객체 역직렬화 시, 클라이언트가 소유해선 안되는 객체 참조를 갖는 필드를 모두 방어적으로 복사해야 한다.**

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // 가변 요소들을 방어적으로 복사
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식을 만족하는지 검사
    if (start.compareto(end) > 0) {
        throw new InvalidObjectException(start + " 가 " + end + " 보다 늦다.");
    }
}
```

유효성 검사에 앞서 방어적 복사를 먼저 수행하며, Date의 clone 메서드는 사용하지 않았다. 모두 Period 객체를 공격으로부터 보호하는데 필요하다. 여기서 final 필드는 방어적 복사가 불가능하기 때문에 readObject 메서드를 사용하려면 start와 end 필드에서 final 한정자를 제거해야 한다. 

프로그램의 결과는 다음 내용을 출력한다. 

```java
Wed Nov 22 00:23:41 PST 2017 - Wed Nov 22 00:@3:41 PST 2017
Wed Nov 22 00:23:41 PST 2017 - Wed Nov 22 00:@3:41 PST 2017
```

## readObject vs 커스텀 메서드

item87은 기본 readObject 메서드를 써도 좋을지를 판단하는 방법을 소개한다.

transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가? 

이 대답이 'No'라면 커스텀 readObject 메서드를 생성하여 모든 유효성 검사와 방어적 검사 수행을 권장한다.
 
final이 아닌 직렬화 기능 클래스라면 readObjet와 생성자의 공통점이 하나 더 있는데, 생성자처럼 readObject 메서드도 재정의 가능 메서드를 호출해서는 안된다는 것이다.

이를 어겼는데 해당 메서드가 재정의되면, 하위 클래스의 상태가 완전히 역직렬화되기 전에 하위 클래스에서 재정의된 메서드가 실행되며, 이는 오작동으로 이어질 것이다.

## 참고자료

- 이펙티브자바 3판



