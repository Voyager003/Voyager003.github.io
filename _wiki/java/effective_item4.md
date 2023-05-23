---
layout  : wiki
title   : 인스턴스화를 막으려거든 private 생성자를 사용하라 
summary : 
date    : 2023-05-23 09:58:03 +0900
updated : 2023-05-23 15:32:00 +0900
tag     : java effectivejava
resource: D1/9FD220-310F-4704-B611-BF646500FD5A
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 추상 클래스로 만드는 것으로 인스턴스화를 막을 수 없다.

- java.lang.Math, java.util.Arrays와 같은 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다.
- 하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들게 된다.
    - 이는 매개 변수를 받지 않는 public 생성자가 만들어지며, 사용자는 생성자가 자동 생성된 것인지 구분할 수 없다.
- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
    - 하위 클래스를 만들어 인스턴스화하면 되기 때문이다.
    - 하지만 추상클래스는 상속하여 사용하라는 뜻으로 오해할 수 있다.
- 이를 해결하기 위해 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

```java

public class UtilityClass {

    private UtilityClass() {
        throw new AssertionError();
    }
    ...
}
```

- 명시적 생성자가 private로 클래스 바깥에서는 접근할 수 없다.
- item3는 클래스 안에서 실수로 생성자를 호출하지 않도록 AssertionError를 던지는 코드를 예시로 들었다.
- 이는 어떤 환경에서도 클래스가 인스턴스화 되는 것을 막아준다.
- 하지만 생성자가 분명 존재하는데 호출할 수 없으니 직관적이지 못한 부분이 있는데, 이는 적절한 주석을 달아 사용자가 인식하도록 한다.
- 상속을 불가능하게 하는 효과가 있다.
    - 모든 생성자는 명시적, 묵시적이든 상위 클래스의 생성자를 호출하게 되는데, private으로 선언했기 때문에 하위 클래스가 상위 클래스의 생성자에 접근할 수 없다.


```java
// java.util.Arrays
public class Arrays {

    // Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {}
    ...
}

// java.lang.Math
public final class Math {

    /**
     * Don't let anyone instantiate this class.
     */
    private Math() {}
    ...
}
```

- 예시에서 말했던 유틸리티 클래스의 API를 살펴보면 non-instantiability 주석을 확인할 수 있다.

## 참고자료

- 이펙티브 자바 3판
