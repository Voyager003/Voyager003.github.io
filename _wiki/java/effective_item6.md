---
layout  : wiki
title   : 불필요한 객체 생성을 피하라
summary : 
date    : 2023-06-05 19:49:47 +0900
updated : 2023-06-05 21:19:56 +0900
tag     : effecitvejava java
resource: 13/0C2144-9AF6-42B2-84B2-C12E0425A698
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

item6의 주요 내용은 새로운 인스턴스 생성을 피하고 재사용을 하도록 만드는 것이다.

## 문자열 객체 생성

```java
String st = new String("str");

```

극단적 예시로 생성자로 String 인스턴스를 만드는 예시를 든다.

"str"이라는 문자를 자주 호출하는 메서드의 경우 수많은 String 인스턴스가 생성될 것이다.

```java
String s1 = "str";
String s2 = "str";

System.out.println(s1==s2); // true
```

위의 코드는 새 인스턴스 생성대신 하나의 String 인스턴스를 사용한다.

JVM 내부에서 동일한 컨텐츠를 가진 리터럴은 동일한 레퍼런스를 참조하도록 하여, 같은 문자열 리터럴("str")을 사용하는 모든 코드가 같은 인스턴스를 재사용한다는 것이 보장된다.

## 정적 팩토리 메서드

```java
Boolean b1 = Boolean.valueOf("true");
Boolean b2 = Boolean.valueOf("true");

System.out.println(b1==b2); // true
```

Boolean.valueOf()는 특정 인스턴스의 primitive 값을 반환하는 정적 팩토리 메서드이다.

valueOf()가 두 번 사용되었지만, 동일 객체를 반환했다.

이는 반환한 객체는 Boolean.TRUE이기 때문에 동일한 결과를 나타낸 것이다.

이처럼 새 인스턴스를 생성해야하는 생성자와 달리 정적 팩토리 메서드는 그렇지 않다.

## 생성 비용이 비싼 객체

생성 시에 메모리를 많이 차지하거나 시간이 오래 걸리는 객체를 비용이 비싼 객체라 한다.

item6는 비싼 객체를 반복적으로 사용해야하는 경우, 캐싱하여 재사용할 수 있는지 고려해보자고 말한다.

```java
stati boolean isRomanNumeral(String s) {
    return s.matches("...정규표현식...");
}
```

matches()는 정규표현식을 받아 Pattern 인스턴스로 컴파일한다.

이는 유한 상태 머신(finite state machine)을 만들어 내는데, 이 인스턴스의 생성 비용이 비싸다.

정규표현식이 변하지않는이상 Pattern 인스턴스를 여러번 만들 필요가 없고 재사용을 하면된다.

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("...정규표현식...");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

ROMAN 인스턴스를 미리 만들어두고 캐싱하여 사용한다면 생성 비용을 줄일 수 있게 된다.

하지만 이 또한 isRomanNumeral()가 초기화된 후에 메서드를 한 번도 호출하지 않는 경우 ROMAN 필드는 쓸데없이 초기화 된 상황이다. 

## 어댑터

어댑터는 실제 작업은 뒷단 인스턴스에 위임하고, 자신은 제2의 인터페이스 역할을 해주는 인스턴스이다.

```java
//뒷 단에 데이터를 넣는다
Map<String,String> map = new HashMap<>();
map.put("t1", "good");
map.put("t2", "bad");

//어댑터를 생성
Set<String> ch1 = map.keySet();
Set<String> ch2 = map.keySet();

ch1.remove("t1"); //ch1에 키 값을 지우기

//뷰 확인(keySet)
System.out.println(ch2.size()); // 결과 : 1

//뒷 단 데이터(map)를 확인
System.out.println(map.size()); // 결과 : 1
```

- 뒷 단(map)에 데이터를 넣고, 어댑터(ch1, ch2)를 생성했다.
- 이 때, 매번 새로운 set 인스턴스가 만들어질 것이라고 예상되지만, 실은 같은 Set 인스턴스를 반환할지도 모른다.
- ch1에서 키 값을 지우고, ch2의 사이즈를 출력한 결과는 1이 나왔다.
- 이는 같은 Set 인스턴스를 반환했다는 것을 확인할 수 있다.
- map 객체의 size도 결과는 1로 size도 1로 줄어든 것을 알 수 있다.
- 이는 모두 똑같은 Map 인스턴스를 대변하기 때문이며, keySet이 뷰 객체를 여러 개 만들어도 상관은 없지만, 그럴 필요도 없다.

## 오토박싱(auto boxing)

오토박싱은 primitive 타입과 boxing된 primitive 타입을 섞어 쓸 때 자동으로 상호 변환해주는 것이다.

하지만 오토박싱은 기본 타입과 그에 대응하는 boxing된 primitive 타입의 구분을 흐려주지만, 완전히 없애는 것은 아니다.

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```

- sum 변수의 타입을 Long으로 만들었기 때문에, 불필요한 Long 객체를 2^31개만큼 만들어 시간이 오래 소요된다.
    - 이는 Long 타입인 i가 Long 타입인 sum에 더해질 때마다 객체를 만든 것이다.
- boxing된 primitive 타입보다 primitive 타입을 사용하고, 의도치 않은 오토박싱이 코드에 담기지 않도록 주의하자.

## 참고자료

- 이펙티브 자바 3판
- https://www.youtube.com/watch?v=0yUxPUXS1pM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=6&ab_channel=%EB%B0%B1%EA%B8%B0%EC%84%A0 - 백기선님의 강좌

