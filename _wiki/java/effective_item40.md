---
layout  : wiki
title   : @Override 애노테이션을 일관되게 사용하라
summary : 
date    : 2023-11-30 09:37:08 +0900
updated : 2023-11-30 10:08:46 +0900
tag     : java effectivejava
resource: AE/81C434-89D0-48EB-9934-9B588B5BA0A6
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## @Override

```java
@Target(value=METHOD)
@Retention(value=SOURCE)
public @interface Override
```

@Override 애노테이션은 메서드 선언이 부모 타입(supertype)의 메서드 선언을 재정의하기 위한 것임을 나타낸다. 

이 애노테이션이 명시된 경우 컴파일러는 다음 조건 중 하나 이상이 충족되지 않는 한 오류 메시지를 생성해야 한다.

- 메서드가 부모타입으로 선언된 메서드를 재정의하거나 구현해야 한다.
- 메서드에 Object에 선언된 공용 메서드의 재정의와 동등한 시그니처가 있는 경우

@Override를 일관되게 사용한다면 여러 버그들을 예방할 수 있다. 코드로 살펴보자.

```java
public class Biagram {
    private final char first;
    private final char second;

    public Biagram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Biagram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Biagram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Biagram(ch, ch));
            }
        }
        System.out.println(s.size()); // 260
    }
}
```

main 메서드에서 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복하여 추가한 뒤에, 집합의 크기를 출력하고 있다. 

Set은 중복을 허용하지 않으므로 26이 출력되기를 기대했지만 예상과는 달리 260이 출력된다.

원인은 equals를 재정의(overriding)한게 아니라 다중 정의(overloading)했기 때문이다. Object의 equals를 재정의하려면 매개변수 타입을 Object로 해야하지만 그렇지 않았다.

결과적으로 Object에서 상속한 equals와 별개인 equals를 정의한 셈이다.

컴파일러가 이 오류를 찾아낼 수 있지만, Object.equals를 재정의한다는 의도를 다음과 같이 명시해야 한다.

```java
@Override
public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
    ^
```

상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애노테이션을 명시하자. 예외는 한 가지로,

구체 클래스에서 상위 클래스의 추상 매서드를 정의할 때는 @Override를 명시하지 않아도 된다. 이는 구체 클래스의 경우 구현하지 않은 추상 메서드가 남아있다면 컴파일러가 이를 알리기 때문이다.

추상 클래스, 인터페이스에서는 상위 클래스나 상위 인터페이스의 메서드를 재정의하는 모든 메서드에 @Override를 명시하자.

## 마무리 

보면서 틀리거나 이상한 내용이 있으면 언제든지 피드백 부탁드립니다.

실제로 틀린 내용이지만 이를 고치지 않으면 잘못된 지식을 알고 있는 것인데 그게 두렵습니다.

읽어 주셔서 감사합니다.

## 참고 자료

- 이펙티브자바 3판
- https://docs.oracle.com/javase%2F7%2Fdocs%2Fapi%2F%2F/java/lang/Override.html - docs
