---
layout  : wiki
title   : 템플릿 메서드 패턴 
summary : 
date    : 2023-05-17 16:17:07 +0900
updated : 2023-05-18 16:00:33 +0900
tag     : pattern java
resource: E5/812EBD-8687-4A0C-A163-562E172249BA
toc     : true
public  : true
parent  : [[/Pattern]]
latex   : false
---
* TOC
{:toc}

## Template Method Pattern(템플릿 메서드 패턴)

먼저 템플릿과 템플릿 메서드 패턴의 정의를 살펴보자.

> 코드에서 어떤 부분은 변경을 통해 그 기능이 다양해지고 확장하려는 성질이 있고, 어떤 부분은 고정되어 있고 변하지 않으려는 성질이 있음을 말해준다  
>
> ...중략...
> 
> 템플릿이란 이렇게 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다. [^1]

다음은 템플릿 메서드 패턴에 대한 정의이다.

> 메서드에 알고리즘의 뼈대를 정의하고 일부 단계를 하위 클래스로 연기한다. 템플릿 방법을 사용하면 하위 클래스가 알고리즘 구조를 변경하지 않고 알고리즘의 특정 단계를 재정의할 수 있다. [^2]

> 템플릿 메서드 패턴은 상속을 통해 기능을 확장해서 사용하는 부분이다. 변하지 않는 부분은 슈퍼클래스에 두고 변하는 부분은 추상 메서드로 정의하여 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 것이다. [^3]

코드로 살펴보자.

## 예시

```java
// Coffee.java
public class Coffee {

    void prepareRecipe() {
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugarAndMilk();
    }
    public void boilWater() {
        System.out.println("물 끓이기");
    }
    public void brewCoffeeGrinds() {
        System.out.println("커피 브루잉");
    }
    public void pourInCup() {
        System.out.println("컵에 따르기");
    }
    public void addSugarAndMilk() {
        System.out.println("설탕과 우유를 추가");
    }
}

// Tea.java
public class Tea {
    
    void prepareRecipe() {
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();
    }
    public void boilWater() {
        System.out.println("물 끓이기");
    }
    public void steepTeaBag() {
        System.out.println("차 우리기");
    }
    public void pourInCup() {
        System.out.println("컵에 따르기");
    }
    public void addLemon() {
        System.out.println("레몬 추가");
    }
}
```

- 위 코드는 커피와 차를 만드는 과정을 나타낸 클래스이다.
- 코드에서 boilWater()와 purtInCup()은 두 클래스에서 중복적으로 나타난다.
- 템플릿 메서드 패턴을 적용해보자.

```java
// CaffeeineBeverage.java
public abstract class CaffeineBeverage {
    
    final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    abstract void brew();           
    abstract void addCondiments();  

    void boilWater() {
        System.out.println("물 끓이기");
    }
    void pourInCup() {
        System.out.println("컵에 따르기");
    }
}
```

- 두 클래스에서 공통적인 로직을 뽑아내 추상 클래스를 만들었다.
- final void prepareRecipe()는 알고리즘을 갖고 있는 메서드로 이를 '템플릿 메서드'라 한다.
- 이어서 알고리즘의 세부 항목에서 차이가 있는 곳은 추상 메서드로 정의했다.

이를 Coffee와 Tea 클래스에 적용하면 다음과 같다.

```java
public class Coffee extends CaffeineBeverage {
    public void brew() {
        System.out.println("필터로 커피를 우려내는 중");
    }
    public void addCondiments() {
        System.out.println("설탕과 커피를 추가하는 중");
    }
}

public class Tea extends CaffeineBeverage {
    public void brew() {
        System.out.println("차를 우려내는 중");
    }
    public void addCondiments() {
        System.out.println("레몬을 추가하는 중");
    }
}
```

- 알고리즘의 세부 항목에서 차이가 있는 부분은 abstract 메서드로 정의했다.

### hook operation(hook 메서드)

hook operation은 필요하다면 subclass에서 확장할 수 있는 기본적인 행동을 제공하는 연산(메서드)이다.
이 때, 기본적으로는 아무 내용도 정의하지 않는다.

```java
public abstract class CaffeineBeverage {
    final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        
        // 고객이 원하는 경우에만 첨가한다.
        if (customerWantsCondiments()) {
            addCondiments();
        }
    }

    abstract void brew();           
    abstract void addCondiments();  

    void boilWater() {
        System.out.println("물 끓이는 중");
    }
    void pourInCup() {
        System.out.println("컵에 따르는 중");
    }

    // hook operation
    boolean customerWantsCondiments() {
        return true;
    }
}
```

- hook operation은 추상 클래스에서 선언되는 메서드로 서브클래스에서 선택적으로 오버라이드할 수 있는 부분이다.
- 서브클래스에서 이 메서드를 구현하거나 구현하지 않을 수 있다.
- 기본적으로 내용이 비어있거나, 기본적인 내용만 구현되어 있다.

## 문제점

템플릿 메서드 패턴은 상속을 사용하기 때문에 상속에서 오는 단점들을 가지고 있다.

- 부모 클래스 의존 문제

상속받고 있다는 것은 특정한 super class를 의존하고 있다는 것을 의미하는데 이 때, super class의 기능을 사용하지 않더라도 super class를 의존하여 강한 의존을 하고있다. 이는 super class를 수정하면, 상속을 받는 sub class에도 영향을 줄 수 있다. (LSP, 리스코프 치환원칙 위반)

- 템플릿 유지

템플릿 메서드의 알고리즘이 추가되어 복잡해진다면, 템플릿을 유지하기 어려워진다. 

## 각주

[^1] : 토비의 스프링 3.1, 3장 템플릿

[^2] Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithms structure. - GOF Design Patterns

[^3] : 토비의 스프링 3.1, 3장 템플릿

## 참고자료

- 토비의 스프링 3.1
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard - 김영한님의 스프링 핵심원리 고급편
- https://johngrib.github.io/wiki/pattern/template-method/ - 예제 코드