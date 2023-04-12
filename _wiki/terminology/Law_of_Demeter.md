---
layout  : wiki
title   : 디미터의 법칙 
summary : 
date    : 2023-04-12 21:50:19 +0900
updated : 2023-04-13 00:04:01 +0900
tag     : terminology 
resource: 5F/F0993C-0449-4A7A-9338-AB4C44A00EA1
toc     : true
public  : true
parent  : [[/terminology]] 
latex   : false
---
* TOC
{:toc}

## 디미터의 법칙

> 디미터 법칙(Law of Demeter), 또는 최소 지식 원칙(Law of Least Knowledge)은 
> 객체에게 자료를 숨기는 대신 함수를 공개하는 것

- 객체가 다른 객체와의 상호작용을 최소화하고, 자신이 접근 가능한 객체에만 메시지를 보내는 원칙이다.
- 객체 간 결합도를 낮추고, 코드 유연성을 확장시키기 위함이다.

## 객체 지향 프로그래밍의 디미터의 법칙

> 디미터 법칙은 "낯선 자에게 말하지 말라" 또는 "오직 인접한 이웃하고만 말하라"로 요약할 수 있다. [^1]

디미터 법칙의 핵심은 객체 구조의 경로를 따라 멀리 떨어진 객체에 메시지를 보내는 설계를 피하라는 것이다. 

이는 객체는 보유하거나 메시지를 통해 얻은 정보만을 통해 결정을 내리고, 다른 객체를 탐색하여 일어나게 해선 안된다.

### 코드로 보는 문제점과 개선

```java
class Wallet {
    private Money money;

    public Wallet() {
        money = new Money();
    }

    public Money getMoney() {
        return money;
    }
}

class Money {
    private int amount;

    public Money() {
        amount = 1000;
    }

    public int getAmount() {
        return amount;
    }

    public void subtract(int value) {
        amount -= value;
    }
}

class Customer {
    private Wallet wallet;

    public Customer() {
        wallet = new Wallet();
    }

    public Wallet getWallet() {
        return wallet;
    }

    public void buy(int amount) {
        // Customer 객체가 Wallet과 Money 객체의 내부 구조에 직접 접근
        wallet.getMoney().subtract(amount);
    }
}
```

코드를 살펴보면, Customer 객체가 Wallet과 Money 객체의 내부 구조에 직접 접근하고 있다.

Customer 객체가 Wallet 객체의 getMoney() 메서드를 호출하여 Money의 subtract() 메서드를 직접 호출하고 있다.

이는 '객체 구조의 경로를 따라 멀리 떨어진 객체에 메시지를 보내는 설계'를 하고 있으므로 디미터 법칙의 요점에 벗어난 코드가 된다.

Customer, Wallet, Money 객체 간 결합도가 높아지며 Customer가 Wallet과 Money 객체의 내부 구조에 종속되어 유연성이 떨어진 코드가 된다.

이를 한번 개선해보자.

```java
class Wallet {
    private Money money;

    public Wallet() {
        money = new Money();
    }

    public Money getMoney() {
        return money;
    }

    public void pay(int amount) {
        money.subtract(amount);
    }
}

class Money {
    private int amount;

    public Money() {
        amount = 1000;
    }

    public int getAmount() {
        return amount;
    }

    public void subtract(int value) {
        amount -= value;
    }
}

class Customer {
    private Wallet wallet;

    public Customer() {
        wallet = new Wallet();
    }

    public Wallet getWallet() {
        return wallet;
    }

    public void buy(int amount) {
        // Customer 객체가 Wallet 객체의 메서드만 호출
        wallet.pay(amount);
    }
}
```

개선된 코드에서는 Customer 객체가 Wallet의 pay()메서드를 호출하여 상호작용하고 있다.

Customer 객체는 Wallet 객체의 메서드만 호출하고, Money 객체의 내부 구조에는 직접 접근하지 않는다.

이는 결합도를 낮추고, Customer 객체가 Wallet, Money 객체의 내부 구조에 독립되어 코드의 유연성 향상을 도모할 수 있다.


## 참고자료
- https://tecoble.techcourse.co.kr/post/2020-06-02-law-of-demeter/ - tecoble 디미터의 법칙
- https://johngrib.github.io/wiki/jargon/law-of-demeter/ - johnlib님의 글
- 조영호님의 오브젝트, chapter6 - '메시지와 인터페이스'

## 각주
[^1]: 조영호님의 오브젝트 183p
