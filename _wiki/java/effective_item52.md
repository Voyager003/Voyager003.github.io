---
layout  : wiki
title   : 다중정의는 신중히 사용하라 
summary : 
date    : 2024-01-19 18:21:19 +0900
updated : 2024-01-19 19:25:54 +0900
tag     : java effectivejava
resource: 2F/DC00EE-0A4E-4617-A625-2E3612DF31E9
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 메서드 다중정의의 문제점

```java
public class CollectionClassifier {

    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }
}

public static void main(String[] args) { 
    Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
        
        for(Collection<?> c : collections) {
            System.out.println(classify(c));
        }
}
```

위 코드에서 집합, 리스트, 그 외를 차례대로 출력할 것을 기대했지만 실제로는 '그 외'가 3번 출력한다. 

이유는 다중정의(overloading)된 세 classifiy 중 어느 메서드를 호출할 지 컴파일 타임에 정해지기 때문이다. 

런타임에는 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못하여 컴파일타임의 매개변수 타입(Collection<?>)을 기준으로 항상 세 번째 메서드인 classifiy(Collection<?> c)만을 호출하게 된다.

여기서 알 수 있는 것은 재정의한 메서드는 동적(런타임)으로 선택되고, 다중정의한 메서드는 정적(컴파일타임)으로 선택된다는 것이다.

메서드를 재정의(override)라는 것은 상위 클래스가 정의한 것과 똑같은 시그니처의 메서드를 하위 클래스에서 다시 정의한 것을 말한다. 

컴파일 타임에 그 인스턴스의 타입이 무엇이었는지 상관없이, 메서드 재정의 후 하위 클래스의 인스턴스에서 그 메서드를 호출하면 재정의한 메서드가 실행된다. 예로 살펴보자

```java
public class Wine {
    String name() { return "포도주"; }
}

public class SparlklingWine extends Wine{
    @Override
    String name() { return "발포성 포도주"; }
}

public class Champane extends SparlklingWine{
    @Override
    String name() { return "샴페인"; }
}

public static void main(Stringp[] args) {
    List<Wine> wineList = List.of(new Wine(), new SparlklingWine(), new Champane());
    
    for(Wine wine : wineList) {
        System.out.println(wine.name());
    }
}
```

Wine 클래스에 정의된 name()은 하위 클래스 SparlklingWine과 Champane에서 재정의된다. 

for 문에서의 컴파일타임 타입이 모두 WIne인 것에 무관하게 가장 하위에서 정의한 재정의 메서드가 실행되어 포도주, 발포성 포도주, 샴페인을 차례로 출력하게 된다.

### 의도와 같은 결과를 얻기위한 다중정의

CollectionClassifier의 의도는 매개변수의 런타임 타입에 기초해 다중정의 메서드로 자동분배하는 것이었다. 이 문제는 

```java
public class CollectionClassifier {

    public static String classify(Collection<?> c) {
        return c instanceof Set ? "집합" :
                c instanceof List ? "리스트" : "그 외";
    }
}
```

CollectionClassifier의 모든 classify 메서드를 합친 뒤 instanceof로 명시적으로 검사하면 해결할 수 있다.

## 다중정의의 주의점

- 다중정의가 혼동을 일으키는 상황을 피한다

API 사용자가 매개변수를 넘기면서 어떤 다중정의 메서드가 호출될지를 알 수 없다면 프로그램이 오동작하기 쉽다. 이를 고려하여 혼동을 일으키는 상황을 피해야한다.

- 다중정의를 만들지 않는다.

가변인수를 사용하는 메서드라면 다중정의를 하지 않는 것이 좋다.

안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.

- 다중정의 대신 메서드 명을 다르게 짓는다.

ObjectOutputStream API를 살펴보자.

```java
public void writeBoolean(boolean val) throws IOException {
    bout.writeBoolean(val);
}

public void writeShort(int val)  throws IOException {
    bout.writeShort(val);
}
```

이 클래스의 write 메서드는 모든 기본 타입과 일부 참조 타입용 변형을 가진다. 이 때 다중정의가 아닌 모든 메서드에 다른 이름을 짓는 방법을 택했다.

이 방식은 read 메서드와 짝을 맞추기 좋다.  

한편 생성자는 이름을 다르게 지을 수 없어 두 번째 생성자부터는 무조건 다중정의가 되는데, 이는 정적 팩토리라는 대안(item1)이 있다. 

이 후 내용은 Java API에서 다중정의로 인한 잘못된 사례를 살펴보는 내용이므로 생략한다.

## 참고자료

- 이펙티브자바 3판


