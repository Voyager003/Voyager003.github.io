---
layout  : wiki
title   : 생성자에 매개변수가 많다면 Builder를 고려하라
summary : 
date    : 2023-05-03 15:39:29 +0900
updated : 2023-05-08 20:40:37 +0900
tag     : java effectivejava 
resource: 7F/1D39A7-4450-435F-9C25-DD1C1989A89C
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 개요

item2는 생성자에 매개변수가 많을 때 Builder(빌더)를 고려하라고 제시한다.

코드를 통해 생성자 패턴은 무슨 제약이 있는지 살펴보고, 빌더가 이를 어떻게 해결하는지 살펴보자.

## Telescoping Constructor Pattern(점층적 생성자 패턴)

> 객체 생성 과정을 나누어 처리하여 객체를 생성하는 패턴

점층적 생성자 패턴을 만드는 법은 다음과 같다.

- 필수 인자를 받는 필수 생성자를 하나 만든다.
- 1개의 선택적 인자를 받는 생성자를 추가한다.
- 2개의 선택적 인자를 받는 생성자를 추가한다.
- 이를 반복한다.
- 모든 선택적 인자를 다 받는 생성자를 추가한다.

조건에 맞도록 작성한 예시 코드는 다음과 같다.

```java
public class NutritionFacts1 {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts1(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts1(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts1(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts1(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts1(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

이제 NutritionFacts1의 인스턴스를 생성하려면 원하는 매개변수를 포함한 생성자를 호출하면 된다. 이를 통해 얻을 수 있는 이점을 찾아보자면, 객체 생성 과정 중 일부 단계에서 필수적인 패러미터가 누락되는 것을 방지할 수가 있을 것이다.

이점에 반해 item2는 다음과 같은 단점을 말한다.

- 사용자가 설정하길 원치 않는 매개변수까지 포함하기 쉬운데, 이러한 경우에도 값 지정이 강제된다.
- 매개변수가 많아질수록 클라이언트 코드를 작성하거나 읽기 어려워진다.
    - 코드를 읽을 때, 각 인자가 무엇을 의미하는지 바로 이해하기 어려우며 타입이 같은 매개변수가 연달아 늘어져 있다면 찾기 힘든 어려운 버그로 이어질 수도 있다.

그렇다면 item2의 두 번째 대안을 살펴보자.

## JavaBeans Pattern(자바빈즈 패턴)

> 매개변수가 없는 생성자로 객체를 만든 후 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 패턴

자바빈즈 패턴을 만드는 법은 다음과 같다.

- 기본 생성자가 있어야한다.
- Getter와 Setter 메서드가 있어야 한다.

조건에 맞도록 작성한 예시 코드는 다음과 같다.

```java
@Getter @Setter
public class NutritionFacts2 {

    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts2() {

    }
}

// 객체 호출
NutritionFacts2 cola2 = new NutritionFacts2();
cola2.setServingSize(240);
cola2.setServings(0);
cola2.setCalories(150);
cola2.setFat(25);
```

자바빈즈 패턴의 setter를 통해 각 인자의 의미를 파악하기 쉬워졌고, 여러 생성자를 만들지 않아도 되는 이점을 찾아볼 수 있다. 하지만 심각한 단점이 있다.

먼저 객체 하나를 생성하려면 메서드를 여러 개 호출해야하게 되는데, 이는 객체의 일관성이 무너지게 된다. 또한 Setter의 존재로 변경 불가능한 클래스를 만들 수 없게 된다.

이를 타파하기 위한 패턴이 앞서 소개한 빌더 패턴이다.

## Builder Pattern(빌더 패턴)

빌더 패턴은 Builder 객체를 통해 객체 생성을 진행한다. 빌더 객체는 필수적인 값들은 미리 정의되어 있으며, 선택적인 값들은 사용자가 추가할 수 있도록 설정된다.

코드로 살펴보자.

```java
public class NutritionFacts3 {

    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        
        public NutritionFacts3 build() {
            return new NutritionFacts3(this);
        }
    }
    
    private NutritionFacts3(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}

// 호출
NutritionFacts3 cola = new NutritionFacts3
    .Builder(240, 8)    // 필수값
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();           // build() 가 객체를 생성하여 반환
```

빌더 패턴으로 생성한 객체 cola를 살펴보면, 각 인자가 어떤 의미인지 쉽게 파악할 수 있게 되었다. 또한 setter로 인한 인스턴스 변경의 우려를 없애 변경 불가능한 객체를 만들 수 있다. 

## Lombok의 @Builder

Lombok은 빌더를 애노테이션의 형태로 제공한다.

```java
@Builder
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
}

// 생성자 
@Builder
public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
```

사용 시 주의점은 @Builder를 클래스에 적용하는 것은 @AllArgsConstructor(access = AccessLevel.PACKAGE)클래스에 추가하고 @Builder가 all-args-constructor에 주석을 적용한 것과 같다.

@AllArgsConstructor는 클래스에 기본 생성자를 생성하지 않기 때문에, 기본 생성자를 요구하는 라이브러리에서 필요한 경우 문제가 될 수 있으며, 생성자 패러미터에 final 키워드를 사용할 수 없어 변경 불가능한 클래스를 만들기 어렵다.

따라서 가급적 직접 만든 생성자에 @Builder를 명시하는 것을 권장한다.


## 참고자료

- 이펙티브 자바 3판
- https://johngrib.github.io/wiki/pattern/builder/#effective-java%EC%9D%98-%EB%B9%8C%EB%8D%94-%ED%8C%A8%ED%84%B4 - 기계인간님의 Builder 패턴


