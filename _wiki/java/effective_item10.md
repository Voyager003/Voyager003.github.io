---
layout  : wiki
title   : equals는 일반 규약을 지켜 재정의하라 
summary : 
date    : 2023-07-03 09:37:18 +0900
updated : 2023-07-03 17:06:42 +0900
tag     : java effectivejava
resource: 3A/CC93ED-4827-403A-B1C4-BDF26C6DBDEC
toc     : true
public  : true
parent  : [[/java]] 
latex   : false
---
* TOC
{:toc}

## Java equals

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

Object 클래스의 equals는 패러미터로 객체의 참조변수를 받아 비교하여 그 결과를 boolean으로 알려주는 역할을 한다.

이 때, 두 인스턴스의 같고 다름을 참조변수의 **주소값**으로 판단한다. 이말은 즉슨, 두 개의 참조변수가 같은 인스턴스를 참조하고 있는지 판단하는 것이다. 

item10은 equals를 재정의 해야하는 경우와 재정의 해야할 경우를 구분하여 설명한다.

### 재정의 하지 않아도 되는 경우

- 각 인스턴스가 본질적으로 고유한 경우

값을 표현하는게 아닌 동작하는 개체를 표현하는 클래스로 Thread가 대표적이다.

- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없는 경우

java.util.regex.Pattern은 equals를 재정의해 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는 방법도 있다. 하지만 클라이언트가 애초에 필요하지 않다고 판단하는 경우 Object의 기본 equals만으로도 해결이 된다.

- 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞는 경우

```java
import java.util.AbstractSet;
import java.util.HashSet;

public class CustomSet<E> extends AbstractSet<E> {

    private HashSet<E> set;

    public CustomSet() {
        set = new HashSet<>();
    }

    @Override
    public boolean add(E element) {
        return set.add(element);
    }

    @Override
    public int size() {
        return set.size();
    }

    // equals 메서드 재정의
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof CustomSet)) {
            return false;
        }
        CustomSet<?> otherSet = (CustomSet<?>) obj;
        return set.equals(otherSet.set);
    }

    // hashCode 메서드 재정의
    @Override
    public int hashCode() {
        return set.hashCode();
    }
}
```

코드는 CustomSet 클래스가 AbstractSet 클래스를 상속하고 있는 경우다.

AbstractSet 클래스는 Set 인터페이스를 구현하고 있으며, equals 메서드 역시 구현되어 있다.

CustomSet 클래스에서는 AbstractSet 클래스의 equals 메서드를 상속받아 사용하며, 이를 equals 메서드에서 재정의하며, 재정의된 equals 메서드는 CustomSet 클래스의 내부 HashSet 객체의 equals 메서드를 호출하여 두 객체가 동일한지 비교한다.

이렇게 CustomSet 클래스는 AbstractSet 클래스가 구현한 equals 메서드를 상속받아 사용하며, 상위 클래스에서 재정의한 equals 메서드가 하위 클래스에도 들어맞게 되고, 따라서 CustomSet 클래스의 객체들 간에 equals 메서드를 호출하면 내부의 HashSet 객체의 equals 메서드가 호출되어 동일성 비교를 수행하게 된다.

이처럼 하위 클래스에 들어맞는 경우에는 재정의할 필요가 없다.

- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없는 경우

뭔가 이상하다. 클래스의 접근자는 default, public인데 클래스가 private 라는 것은 무엇을 의미하는 것일까?

이는 중첩 클래스나 이너(inner) 클래스를 의미하는 것 같다.

```java
public class OuterClass {
    private int value;

    private class PrivateNestedClass {
        // 중첩 클래스의 구현 내용
    }

    public OuterClass(int value) {
        this.value = value;
    }

    // equals 메서드 재정의
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof OuterClass)) {
            return false;
        }
        OuterClass other = (OuterClass) obj;
        return this.value == other.value;
    }
}
```

중첩 클래스인 PrivateNestedClass는 외부에서 해당 클래스에서 직접 접근할 수 없다.

Outerclass는 equals를 재정의하고 있는데, 중첩 클래스는 외부에서 직접 접근할 수 없기 때문에, 해당 중첩 클래스의 인스턴스와 비교할 일이 없으며, equals를 호출할 필요도 없게된다.

하지만 실수로라도 호출되는 걸 막고싶다면,

```java
@Override public boolean equals (Object o) {
    throw new AssertionError(); 
}
```

이처럼 예외를 던지도록 구현하면 될 것이다. 

그렇다면 equals 를 재정의해야하는 경우는 언제일까?

### 재정의해야하는 경우

먼저 객체 식별성(object identity)이 아닌 논리적 동치성(logical equality)을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때이다.

```java
String str1 = new String("abc");
String str2 = new String("abc");

System.out.println(str==str2); // false
```

String str1과 str2는 같은 값인 "abc"를 담고 있지만, == 비교를 한다면 메모리의 주소값을 비교하기 때문에 false를 출력하게 된다.

이 때, 논리적 동치성을 확인하도록 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함은 물론 Map 및 Set의 element로 사용할 수 있게된다.

equals를 재정의하려면 어떻게 해야할까?

## equals 재정의 방법

equlas 재정의 시 반드시 따라야하는 Object 명세에 적힌 규약은 다음과 같다.

### 반사성(reflexivity)

- null이 아닌 모든 참조 값 x에 대해, x.equlas(x)는 true다

```java
public class ReflexivityExample {
    private int value;

    public ReflexivityExample(int value) {
        this.value = value;
    }

    public static void main(String[] args) {
        ReflexivityExample obj = new ReflexivityExample(10);

        // 반사성: 객체는 자기 자신과 동일해야 한다
        boolean isReflexive = obj.equals(obj);
        System.out.println("반사성: " + isReflexive); // true
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof ReflexivityExample)) {
            return false;
        }
        ReflexivityExample other = (ReflexivityExample) obj;
        return this.value == other.value;
    }
}
```

ReflexivityExample 클래스가 main 메서드에서 equals 메서드의 반사성을 검증하는 코드이다.

인스턴스인 obj를 생성한 후, obj.equlas(obj)를 호출해 자기 자신과 동일한지 확인한다. 이 때, equals는 this==obj 조건을 검사해 자기 자신과 동일한지 확인한다.

equals는 반사성을 만족해야 하므로, obj.equlas(obj)는 true를 반환해야 한다.

### 대칭성(symmetry)

- null이 아닌 모든 참조 값 x, y에 대해, x.equlas(y)가 true면, y.equals(x)도 true다.
- 이는 두 인스턴스는 서로에 대한 동치 여부에 똑같이 답해야 한다는 것이다.

```java
public final class CaseInsensitiveString {
  private final String s;
  
  public CaseInsensitiveString(String s) {
    this.s = Objects.requireNonNull(s);
  }
  
  //대칭성 위배
  @Override
  public boolean equals(Object o) {
    if(o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(
    ((CaseInsensitiveString) o).s);
    
    if(o instanceof String) // 한 방향으로만 작동
      return s.equalsIgnoreCase((String)o);
    return false;
  }
}
```

CaseInsensitiveString 클래스는 대소문자를 구별하지 않는 문자열을 담는 클래스다.

이 클래스에서 toString()은 원본 문자열의 대소문자를 그대로 돌려주지만 equals에서는 대소문자를 무시하게 된다.

이 클래스의 equals()는 일반 문자열과도 비교를 시도하게 된다. 

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish"; 

cis.equals(s); // true
s.equals(cis); // false
```

cis.equals(s)는 true를 반환하지만, CaseInsensitiveString String의 equals는 일반 String인 s를 알고 있지만, 역으로 String equals는 그렇지 않다.

때문에 s.equals(cis)는 false를 반환하여 대칭성을 위배한다.

이제 이 코드를 Collection에 넣어보자.

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish"; 

List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);

list.contains(s); // ?
```

contain(s)는 내부적으로 equals()를 사용하기 때문에, false를 반환한다. (이 때, item10에서는 이는 순전히 구현하기 나름이라 JDK 버전이 바뀌거나 다른 JDK에서는 true를 반환 혹은 런타임 예외를 던질 수 있다고 말한다.)

equals 규약을 어기면 그 인스턴스를 사용하는 다른 인스턴스들이 어떻게 반응할 지 알 수가 없다. 그렇다면 equals를 어떻게 구현해야 할까?

```java
@Override
public boolean equals(Object o) {
	return o instanceof CaseInSensitiveString &&
    ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

위 코드는 

### 추이성(transitivity)

- null이 아닌 모든 참조 값 x,y,z에 대해, x.equals(y)가 true이고, y.equals(z)도 true면, x.equals(z)도 true다.

```java
public class Point {
    protected final int x;
    protected final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

간단하게 2차원에서 점을 표현하는 Point 클래스다. 이 클래스를 확장해 색상을 더한다고 해보자.

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    
    ...
}
```

이 때. ColorPoint 클래스의 equals()를 그대로 둔다면 Point의 구현이 상속되어 색상에 대한 정보는 무시한 채 비교를 하게된다.

색상에 대한 정보도 비교 대상에 포함되도록 equals()를 정의하려면

```java
@Override
public boolean equals(Object o) {
    if(!(o instanceof ColorPoint)) {
        return false;
    }
    return super.equals(o) && ((ColorPoint)o).color == color;
}
```

다음과 같다. 하지만 이는 일반 Point를 ColorPoint에 비교한 결과와 그 둘을 바꿔 비교한 결과가 다를 수 있다. 즉 대칭성을 위반한다.

Point의 equals는 Color를 무시하고, ColorPoint의 equals는 패러미터의 클래스 종류가 다르다며 false를 반환할 것이다. 각 인스턴스를 하나씩 만들어 실제 동작하는 모습을 보자.

```java
Point p = new Point(1,2);
ColorPoint cp = new ColorPoint(1,2,Color.RED);

p.eqauls(cp); // true
cp.eqauls(p); // false
```

하나는 true, 그 반대는 false를 반환한다. 하지만 이렇게 되면 대칭성은 맞지만 추이성은 위배되며 재귀에 빠질 위험이 생긴다. 그렇다면 ColorPoint의 equals()가 Point와 비교할 때는 색상을 무시하도록 하면 어떨까?

```java
@Override
    public boolean equals(Object o) {
        if(!(o instanceof Point))
            return false;
        //o 가 일반 Point면 색상을 무시하고 비교
        if(!(o instanceof ColorPoint))
            return o.equals(this);
        // o 가 ColorPoint면 색상까지 비교
        return super.equals(o) && ((ColorPoint)o).color == color;
    }
```

색상을 무시하도록 하여 대칭성을 지켰다. 하지만 이는 추이성을 깨버리게 되었다.

```java
ColorPoint p1 = new ColorPoint(1,2,Color.RED);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1,2,Color.BLUE);

p1.eqauls(p2); // true
p2.eqauls(p3); // true
p1.eqauls(p3); // false
```

이 또한 추이성을 명백하게 위배한 코드이다. 

이러한 현상은 모든 객체 지향 언어의 동치 관계에서 나타나는 근본적인 문제로, 구체 클래스를 확장해 새로운 값을 추가하면서 equlas 규약을 만족시킬 방법은 없다.

item10은 이를 해결하기 위해 다음과 같은 방법을 제시한다.

```java
public class ColorPoint {
    private final Color color;
    private final Point point;

    public ColorPoint(int x, int y, Color color){
        this.point = new Point(x,y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint(){
        return point;
    }

    @Override
    public boolean equals(Object o){
        if(!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(cp);
    }

}
```

Point를 상속하는 대신, Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰를 public으로 추가한 컴포지션 방식이다. 

Java의 라이브러리 중에서도 java.sql.Timestamp와 같이 util.Date를 확장하여 필드를 추가한 방식을 사용을 종종 사용한다.

```java
public class Timestamp extends java.util.Date {

    private int nanos;
    ...
}
```

하지만 이러한 방식도 Data 인스턴스와 한 컬렉션에 넣거나 서로 섞는다면 엉뚱하게 동작할 수 도 있어 API 설명에 주의 사항을 언급하고 있다.

### 일관성(consistency)

- null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.

- 두 인스턴스가 같다면 수정되지 않는 한 영원히 같아야 한다는 것이다.

- 클래스를 불변 클래스로 만들기로 했다면, equals가 한 번 같다고 한 객체와는 영원히 같다고 해야하며, 다르다고 한 객체와는 영원히 다르다고 답하도록 해야한다.

- 이 때, 클래스가 불변이든 가변이든 equals()의 판단에 신뢰할 수 없는 자원이 끼어들게 해선 안된다. (일관성이 깨짐)

### null-아님

- null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false다.
- 다시 말해 모든 인스턴스가 null과 같지 않아야 한다는 것이다.

```java
//명시적 null 검사
@Override public boolean equals(Object o) {
  if(o == null)
    return false;
}
```

이러한 명시적인 null검사는 보통 불필요하다. 동치성을 검사하기 위해서는 equals는 건네받은 인스턴스를 형변환 후에 필수 필드들의 값을 알아내야 한다. 그렇다면 형변환에 앞서 instanceof 연산자로 입력 패러미터가 올바른 타입인지 검사해야한다.

코드는 다음과 같다.

```java
//묵시적 null 검사
@Override 
public boolean equals(Object o) {
  if(!(o instanceof MyType))
    return false;
	MyType mt = (MyType) o;
  ...
}
```

instanceof는 첫 번째 피연산자가 null이라면 false를 반환한다. 따라서 입력이 null이면 타입 확인 단계에서 false를 반환하기 때문에 null검사를 명시적으로 하지 않아도 된다.

## 종합한 주의 사항

1) '==' 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - 자기 자신이라면 true를 반환하도록 한다.

2) instanceof 연산자로 입력이 올바른 타입인지 확인한다.
    - 어떤 인터페이스는 자신을 구현한 서로 다른 클래스끼리도 비교할 수 있도록 equals 규약을 수정하기도 한다.
    - 이러한 인터페이스를 구현한 클래스라면 equals에서 해당 인터페이스를 사용해야한다. (Set, List, Map, Map.Entry와 같은 Collection 인터페이스들이 이에 해당한다.

3) 입력을 올바른 타입으로 형변환한다.
    - instanceof 검사를 통과했다면, 성공한다.

4) 입력 인스턴스와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.
    - 모든 필드가 일치한다면 true, 하나라도 다르다면 false를 반환하도록 한다.
    - 인터페이스를 사용했다면 입력의 필드값을 가져올 때도 그 인터페이스의 메서드를 사용해야 한다. 
    - 이 때, 타입이 클래스라면 권한에 따라 해당 필드에 직접 접근할 수도 있다.

5) equals 재정의 시 hashCode도 반드시 재정의하자(item11)

6) 너무 복잡하게 해결하려 들지 말자
    - 필드들의 동치성 검사만 하더라도 equals 규약을 어렵지 않게 지킬 수 있다.
    - 일반적으로 별칭(alias)를 피하도록 하자.

7) Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

```java
public boolean equals(MyClass o) {
}
```

이 코드는 Object.equals를 재정의한 것이 아니라, 입력타입이 Object가 아니므로 다중정의(Overloading) 한 것이다.(item52)

기본 equals를 그대로 둔 채로 추가한 것일지라도, 타입을 구체적으로 명시한 equals는 오히려 해가된다.  

### Lombok의 @EqualsAndHashCode

```java
@EqualsAndHashCode
public class MyClass {
    
    private Long id;
    private String valuel
}

// 디컴파일
public boolean equals(Object o) {
    if (o == this) {
        return true;
    } else if ...

}    
```

해당 애노테이션을 클래스 위에 정의하면 컴파일 시점에 자동으로 equals와 hashCode 메서드가 오버라이딩 된다.



내용이 복잡해서 나중에 추가적으로 다시 정리해봐야 겠다.

## 참고자료

- 이펙티브 자바 3판

























