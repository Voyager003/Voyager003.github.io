---
layout  : wiki
title   : equals를 재정의하려거든 hashCode도 재정의하라
summary : 
date    : 2023-07-10 10:00:45 +0900
updated : 2023-07-10 11:44:39 +0900
tag     : java effectivejava 
resource: 6C/DCBC43-4F68-42C2-83FE-FB7288774EA7
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Overriding hashCode

equals를 재정의한 클래스에서 hashCode를 재정의하지 않는다면 hashCode의 일반 규약을 어겨 해당 클래스의 HashMap/Set과 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다. 

여기서 Object 명세의 일반 규약은 다음과 같다.

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 인스턴스의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.

- equals(Object)가 두 인스턴스를 같다고 판단했다면, 두 인스턴스의 hashCode는 같은 값을 반환해야 한다.

- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 인스턴스의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 인스턴스에 대해서는 다른 값을 반환해야 Hashtable의 성능이 좋아진다. 
    - Hashtable에서 인스턴스를 저장하거나 탐색할 때, 일반적으로 O(1)의 시간복잡도로 객체를 탐색할 수 있다.
    - 하지만 hashCode의 값이 충돌하는 경우에 같은 hashCode 값을 가진 인스턴스들을 Hashtable의 한 버킷에 저장하고, 이들을 비교하여 원하는 객체를 찾아야 한다.
    - 이 과정에서 선형 탐색이 필요할 수 있으며, 이는 시간 복잡도를 O(n)까지 증가시킬 수 있다.

item11은 hashCode 재정의를 잘못했을 때, 문제되는 코드로 다음을 제시한다.

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(short areaCode, short prefix, short lineNum){
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999,"프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자번호");
    }
    
    private static short rangeCheck(int val, int max, String arg){
        if(val < 0 || val > max){
            throw new IllegalArgumentException(arg+" : "+val);
        }
        return  (short) val;
    }

    @Override
    public boolean equals(Object o){
        if(o == this)
            return true;
        if(!(o instanceof PhoneNumber)){
            return false;
        }

        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }
}
```

PhoneNumber 클래스의 인스턴스를 HashMap의 원소로 사용한다고 가정하자.

다음에 다음 코드를 실행한다.

```java
Map<PhoneNumber, String> map = new HashMap<>();
        map.put(new PhoneNumber((short) 707, (short) 867, (short) 5309), "ex");
        
Assertions.assertThat(map.get(pn)).isEqualTo("ex"); // fail
```

map에 인스턴스를 put하고 난뒤 기대한 예상값은 "ex"였지만 실제로는 null을 반환했다.

이유는 2개의 PhoneNumber 인스턴스가 사용됐기 때문이다.

첫 번째로는 HashMap에 "ex"를 put할 때, 두 번째로는 get할 때 사용된 것이다.

hashCode를 재정의하지 않았기 때문에 논리적 동치인 두 인스턴스가 서로 다른 hashCode를 반환하여 상단의 두 번째 규약을 위반했다.

이를 해결하려면 서로 다른 인스턴스에 다른 hashCode를 반환하도록 해야한다. 

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
        
    return result;
}

// test
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(areaCode, prefix, lineNum), "ex");

Assertions.assertThat(map.get(pn)).isEqualTo("ex");
```

hashCode() 메서드를 재정의한 코드이다.

int 변수에서 result를 초기화하고 해당 필드의 hashCode를 계산하고, 계산한 hashCode로 result를 갱신했다. 

이제 기댓값인 "ex"를 반환하는 것을 확인할 수 있다.

## 캐싱 방식

클래스가 불변이며 hashCode를 계산하는 비용이 크다면 캐싱하는 방식을 고려하는 것이 좋다.

```java
private int hashCode; // 자동으로 0으로 초기화

@Override
public int hashCode() {
    int result = hashCode;

    if(result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    
    return result;
}
```

해시의 키로 사용되지 않는 경우, hashCode가 처음 호출될 때 계산하는 지연 초기화(lazy initialization) 전략이 있다. 

지연 초기화 시 주의할 점은 그 클래스를 thread-safe 하도록 신경써야 하며(item83), 성능을 높인답시고 hashCode 계산 시 핵심 필드를 생략해선 안된다. 

hashCode가 반환하는 값의 생성 규칙을 API 공표하지말자. 이는 클라이언트가 이 값에 의지하지 않고 추후 계산 방식을 바꿀 수도 있기 때문이다.

## 참고자료 

- 이펙티브 자바 3판

