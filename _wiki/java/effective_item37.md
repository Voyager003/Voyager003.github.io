---
layout  : wiki
title   : ordinal 인덱싱 대신 EnumMap을 사용하라 
summary : 
date    : 2023-11-20 09:28:18 +0900
updated : 2023-11-20 11:23:58 +0900
tag     : java effectivejava
resource: 14/A70EE9-AC43-433F-98F4-3BFE4BE06226
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## ordinal 인덱싱의 문제점

[item35](https://voyager003.github.io/wiki/java/effective_item35/) 에서 ordinal 메서드를 사용함으로써 생기는 문제점을 살펴본 적이 있다.

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;
    
    Plant(String name, LifeCycle lifeCycle) { 
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

정원의 식물을 표현한 예제이다. 이 클래스에서 식물들을 배열 하나로 관리하고, 생애주기별(한해살이, 여러해살이, 두해살이)로 묶는다고 해보자.

이 때 식물의 집합을 배열 하나에 넣고 생애주기의 ordinal() 메서드로 반환받은 값을 그 배열의 인덱스로 사용하는 상황이다.

```java
Set<Plant>[] plantsByLifeCycle = 
    (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
    
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p)
}
```

코드는 동작하지만 문제가 있다. 

먼저 배열은 제네릭과 호환되지 않아 비검사 형변환이 필요하다.

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
```

코드를 살펴보면 Set 배열을 제네릭 Set 배열로 캐스팅하고 있으며, 이는 비검사 형변환에 해당한다.

이는 Set<Plant>[]과 같은 배열을 만들면 실제로는 raw Set 배열(Set[])이 되고, 타입 소거(type erasure)로 인해 실행 시에 제네릭 정보가 손실되는 것을 의미한다.

또한 배열은 각 인덱스의 의미를 모르기 때문에 출력 결과에 직접 레이블을 달아야하고, 정확한 정숫값을 사용한다는 것을 개발자가 직접 보증해야한다는 문제점이 발생한다. 

이를 해결하기 위해 item37이 제시한 해결법은 바로 EnumMap이다.

## EnumMap

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle 
    = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashMap<>());

for (Plant p : garden) 
    plantsByLifeCycle.get(p.lifeCycle).add(p);
```

열거형 타입을 map의 key로 사용한 EnumMap을 사용해 위 코드를 개선한 코드이다.

EnumMap은 내부적으로 열거형의 순서와 일치하는 크기의 배열을 사용하여 데이터를 저장한다. 

이 배열을 사용하면 열거형의 모든 값에 대해 직접적이고 효율적인 접근이 가능하고, 배열은 열거형 값의 순서에따라 인덱스가 할당되기 때문에 열거형 값 간의 대응 관계가 명확해진다. 

EnumMap의 API를 살펴보면 

```java
/**
 * Creates an empty enum map with the specified key type.
 *
 * @param keyType the class object of the key type for this enum map
 * @throws NullPointerException if {@code keyType} is null
 */
public EnumMap(Class<K> keyType) {
    this.keyType = keyType;
    keyUniverse = getKeyUniverse(keyType);
    vals = new Object[keyUniverse.length];
}
```

생성자에서 빈 EnumMap을 생성한다. keyType은 EnumMap의 키로 사용되는 열거형의 클래스 객체로, 중요한 부분은 Class<K> keyType로써, 제네릭 타입 K가 한정적 타입 토큰으로 사용되고 있다.

한정적 타입 토큰은 제네릭 클래스나 메서드에서 사용되는 타입 정보를 런타임에도 유지하기 위한 방법 중 하나인데, 열거형은 컴파일 타임에만 알려져 있기 때문에 런타임에서도 이 정보를 사용하려면 Class<K>를 통해 런타임에 열거형의 타입을 전달해야 한다.

### Stream을 사용한 EnumMap

Java 8의 Stream을 사용해 map을 사용하면 코드를 좀더 줄일 수 있다.

```java
System.out.println(Arrays.stream(garden)
        .collect(Collectors.groupingBy(p -> p.lifeCycle)));
```

하지만 Collectors.groupingBy는 스트림의 요소를 특정 기준으로 그룹화하여 Plant 객체를 LifeCycle 열거형 값에 따라 그룹화하여 결과적으로 Map<Plant.LifeCycle, List<Plant>> 형태의 맵이 생성된다.

고유한 map 구현체를 사용했기 때문에 EnumMap을 사용하여 얻은 공간과 성능 이점이 사라진다는 문제가 있다. 이를 위해 매개변수에 원하는 맵 구현체를 명시해 호출할 수 있다.

```java
System.out.println(Arrays.stream(garden)
        .collect(Collectors.groupingBy(p -> p.lifeCycle,
            () -> new EnumMap<>(LifeCycle.class), Collectors.toSet())));
```

이 때 유의할 점은 스트림을 사용하면 EnumMap만을 사용했을 때와 약간 다르게 동작한다는 것이다.

EnumMap만 사용했을때는 항상 식물의 생애주기(LifeCycle) 당 중첩 맵을 한개씩 만들지만, 스트림을 사용한 버전에서는 해당 생애주기에 속하는 식물이 있을때만 만들게 된다. 무슨 의미인지 코드로 살펴보자.

```java
// Only Enum
System.out.println("only Enum");
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);

// Stream
System.out.println("using Stream");
Map<LifeCycle, List<Plant>> collect = Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle));
System.out.println(collect);

// Stream with enumMap
System.out.println("stream with EnumMap");
EnumMap<LifeCycle, Set<Plant>> collect1 = Arrays.stream(garden)
    .collect(groupingBy(p -> p.lifeCycle,
        () -> new EnumMap<>(LifeCycle.class), toSet()));
System.out.println(collect1);

// 출력 결과
only Enum
{ANNUAL=[basil, rosemary], PERENNIAL=[], BIENNIA=[parsley, caraway]}

using Stream
{ANNUAL=[basil, rosemary], BIENNIA=[caraway, parsley]}

stream with EnumMap
{ANNUAL=[basil, rosemary], BIENNIA=[parsley, caraway]}
```

이처럼 스트림 사용 시, 열거형을 사용했을 때와 비교했을 때 생성되는 맵의 갯수가 다른 것을 확인할 수 있다.

## 마무리

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html - enummap docs

