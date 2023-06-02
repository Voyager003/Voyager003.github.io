---
layout  : wiki
title   : 전략 패턴 
summary : 
date    : 2023-06-02 10:37:43 +0900
updated : 2023-06-02 15:07:31 +0900
tag     : java pattern
resource: E1/4646A2-B0FF-43DC-83C3-A64FC59FF63E
toc     : true
public  : true
parent  : [[/Pattern]]
latex   : false
---
* TOC
{:toc}

## Strategy Pattern(전략 패턴)

전략 패턴의 정의를 알아보자.

> 동일 계열의 알고리즘군을 정의하고, 각 알고리즘을 캡슐화하며, 이들을 상호교환이 가능하도록 만든다. 알고리즘을 사용하는 클라이언트와 상관없이 독립적으로 알고리즘을 다양하게 변경할 수 있게 한다. [^1]

> 전략 패턴은 OCP 관점에 보면 확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식이다. [^2]

![alt](https://upload.wikimedia.org/wikipedia/commons/3/39/Strategy_Pattern_in_UML.png)

그림으로 살펴보면, 변하지 않는 부분을 Context라는 곳에 코드를 배치하고, 변하는 부분을 Strategy라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 하여 문제를 해결하는 방식으로 상속이 아닌 위임으로 문제를 해결하는 것이다.

코드로 살펴보자.

## 예제

```java
// 전략 인터페이스
interface SortStrategy {
    void sort(int[] numbers);
}

// Bubble Sort 전략 클래스
class BubbleSortStrategy implements SortStrategy {

    @Override
    public void sort(int[] numbers) {
        System.out.println("Bubble Sort를 사용하여 정렬합니다.");
        // Bubble Sort 로직 구현
    }
}

// Quick Sort 전략 클래스
class QuickSortStrategy implements SortStrategy {

    @Override
    public void sort(int[] numbers) {
        System.out.println("Quick Sort를 사용하여 정렬합니다.");
        // Quick Sort 로직 구현
    }
}

// Context 클래스
class SortContext {

    private SortStrategy strategy;

    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }

    public void executeSort(int[] numbers) {
        strategy.sort(numbers);
    }
}

// 메인 클래스
public class Main {
    public static void main(String[] args) {
        int[] numbers = {5, 2, 9, 1, 3};

        SortContext context = new SortContext();

        // Bubble Sort 전략을 사용하여 정렬
        context.setStrategy(new BubbleSortStrategy());
        context.executeSort(numbers);

        // Quick Sort 전략을 사용하여 정렬
        context.setStrategy(new QuickSortStrategy());
        context.executeSort(numbers);
    }
}
```

- SortContext는 변하지 않는 로직을 가진 템플릿 역할을 하는 코드로, 전략 패턴에서는 Context(컨텍스트, 문맥)라 한다.
    - Context는 크게 변하지 않지만, Context에서 strategy를 통해 전략이 변경된다.
- SortContext는 코드 내부에 SortStrategy strategy 필드를 가진다.
    - 필드에 변하는 부분인 Strategy의 구현체가 주입이 된다.
- 결과적으로 SortContext는 Strategy 인터페이스에만 의존하여, 템플릿 메서드 패턴과 달리 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향을 주지않는다.

### 익명 클래스와 람다 표현식을 사용한 전략 패턴

```java
interface SortStrategy {
    void sort(int[] numbers);
}

class SortContext {
    private SortStrategy strategy;

    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }

    public void executeSort(int[] numbers) {
        strategy.sort(numbers);
    }
}

public class Main {
    public static void main(String[] args) {
        int[] numbers = {5, 2, 9, 1, 3};

        SortContext context = new SortContext();

        // 익명 클래스를 사용하여 Bubble Sort 전략을 정의
        context.setStrategy(new SortStrategy() {
        
            @Override
            public void sort(int[] numbers) {
                System.out.println("Bubble Sort를 사용하여 정렬합니다.");
                // Bubble Sort 로직 구현
            }
        });

        // Bubble Sort 전략으로 정렬 실행
        context.executeSort(numbers);
    }
}
```

- 익명 클래스를 사용해 클래스를 생성하지 않고, 전략 패턴을 구현할 수 있다.

```java
context.setStrategy((int[] nums) -> { 
    System.out.println("Bubble Sort를 사용하여 정렬합니다.");
    ...
});
```

- 인터페이스에 메서드가 1개만 있는 경우에 람다 표현식을 적용할 수 있다.

제시한 방법은 Context와 Strategy를 실행 전에 원하는 모양으로 조립하고, 이 후에 Context를 실행하는 선 조립, 후 실행 방식이다.

이는 Spring에서 애플리케이션 개발 시, 로딩 시점에 의존관계 주입을 통해 필요한 의존관계를 모두 맺고 실제 요청을 처리하는 것과 원리가 같다.

하지만 이 방식은 Context 안에서 이미 구체적인 Strategy 클래스를 사용하도록 고정되어 있다. Context가 Strategy의 인터페이스 뿐만 아니라 특정 구현 클래스를 직접 알고 있다는 것은, 전략 패턴 및 OCP 원칙에 들어맞다고 볼 수는 없다.

선 조립방식보다 유연하게 작동하는 전략 패턴을 만들 수 없을까?

### Client와 Context 분리

Client가 구체적인 Strategy를 선택하고 오브젝트로 만들어 Context에 전달하여 Context가 Strategy 구현 클래스의 오브젝트를 사용하도록 만들어보자.

```java
public class Main {
    public static void main(String[] args) {
        int[] numbers = {5, 2, 9, 1, 3};

        SortContext bubbleSortContext = new SortContext(new BubbleSortStrategy());
        bubbleSortContext.executeSort(numbers);

        SortContext quickSortContext = new SortContext(new QuickSortStrategy());
        quickSortContext.executeSort(numbers);
    }
```

- Strategy 클래스의 오브젝트(SortContext)를 생성하고, Context를 호출하여 전략 오브젝트를 전달하도록 했다.
- Client는 Context 실행 시점에 원하는 Strategy를 전달이 가능하게 되었다.

살펴본 두 가지 방식 모두 각자 장단점을 가지고 있다. 

필드에 Strategy를 저장하는 방식은 선 조립, 후 실행 방법이기 때문에 Strategy를 신경쓰지 않고 단순 실행만 하면 되는 반면에, 패러미터에 Strategy를 전달받는 방식은 실행할 때마다 유연하게 변경가능하지만 실행 할 때마다 Strategy를 계속 지정해 줘야 한다는 점이 번거롭다. 

소프트웨어의 복잡성을 해소할 은탄환은 없기 때문에, 상황을 고려해 필요에 따라 사용하면 좋을 것이다.


## 참고자료

- 토비의 스프링 3.1, 템플릿
- https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard - 김영한님의 강의

## 각주
[^1] : GoF 디자인패턴

[^2] : 토비의 스프링 3.1
