---
layout  : wiki
title   : Java Annotation
summary : 
date    : 2023-05-19 10:11:46 +0900
updated : 2023-05-19 11:50:49 +0900
tag     : java
resource: 8D/8C5376-90AE-44C0-A2C3-BBDF58139241
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

[백기선님의 스터디](https://github.com/whiteship/live-study/issues/12) 12주차 과제

## Annotation과 정의하는 방법

애노테이션 등장 전에는 소스코드와 프로그램 설정파일(.xml)을 따로 작성하여 코드와 설정파일 분리에 따른 어려움이 있었다.

프로젝트에서 여러 개발자들이 함께 작업할 경우 설정정보를 공유해야 하는데, xml 파일을 사용하는 경우 하나의 xml파일을 두고 사용하여 사용/수정에 어려움을 겪었다. 

이를 해결하기 위해 JDK 1.5에 등장한 java.lang.annotation.Annotation(애노테이션)은 주석을 뜻하는 단어로, 소스코드 안에 다른 프로그램을 위한 정보를 미리 약속된 형식으로 포함시킨 것이다. 이는 프로그래밍 언어에 영향을 미치지 않고 다른 프로그램에게 정보를 제공할 수 있게 되었다.

```java
// 정의 방법
public @interface AnnotationName {
    타입 요소 이름();
}
```

- 애노테이션은 단어 그대로 표시해놓는 주석이다.
- 컴파일러 수준에서 해석되기 때문에 동적으로 실행되는 코드는 들어가지 않는다. 
- 때문에 와전히 정적이어야 하며, 동적으로 런타임 중에 바뀌어야 하는 요소는 애노테이션에 사용할 수 없다.

### Java 표준, 메타 애노테이션

- 표준 애노테이션 : Java에서 기본적으로 제공하는 애노테이션이다.
    - @Override : 현재 메서드가 super class의 메서드를 오버라이드한 것임을 컴파일러에게 알린다.
    - @Deprecated : 다음 버전에 지원되지 않을 수도 있기 때문에 사용하지 않을 것을 권장하는 타겟에 명시한다.
    - @SuppressWarnings : 컴파일러의 특정 경고메시지가 나타나지 않게 한다.
    - @SafeVarargs : 제네릭 타입의 가변인자 매개변수 사용 시, 경고를 무시한다. (JDK 1.7)
    - @FunctionalInterface : 컴파일러에게 함수형 인터페이스(1개의 추상 메서드를 갖고 있는 인터페이스)라는 것을 알린다. (JDK 1.8)

- 메타 애노테이션 : 애노테이션으로 애노테이션을 정의할 때 적용 대상이나 유지 기간을 지정하는데 사용한다.

  - @Target : 애노테이션이 적용 가능한 대상을 지정한다.
    - 지정할 수 있는 적용 대상은 다음과 같다.
    ```jav
    @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
    ```
    - TYPE : 타입 선언 시, 애노테이션을 붙일 수 있다는 의미
    - FIELD : 필드, enum 상수
    - METHOD : 메서드
    - PARAMETER : 메서드의 매개변수
    - CONSTRUCTOR : 생성자
    - LOCAL_VARIABLE : 지역변수
    - ANNOTATION_TYPE : 애노테이션
    - PACKAGE : 패키지
    - TYPE_PARAMETER : 타입 변수에만 사용 가능 
    - TYPE_USE : 해당 타입의 변수를 선언할 때 붙일 수 있다는 의미
    - 이는 괄호('{}')를 이용해 여러 개의 값을 지정할 수 있다. 

  - @Retention : 애노테이션의 유지 기간을 지정한다.
    - SOURCE : 소스파일에만 존재하고, 클래스 파일에는 존재하지 않는다. 
      - 바이트 코드에도 남아있지 않는다.
    - CLASS : 컴파일된 클래스 파일에 존재하고, 실행 시 사용 불가능하다. (Default)
      - 바이트 코드에는 남아있지만, 클래스파일을 JVM이 실행할 때 클래스에 대한 정보를 class loader가 메모리에 적재하게 되고, 사용 시점에 메모리에서 읽어올 때 애노테이션 정보를 제외하고 읽어온다.
    - RUNTIME : 컴파일된 클래스 파일에 존재하고, 런타임 시에도 참조할 수 있다.
      - 메모리에 적재된 클래스 정보를 읽어올 때, 애노테이션 정보를 그대로 포함하는 것을 의미한다.
  - @Documented : 애노테이션에 대한 정보가 javadoc으로 작성한 문서에 포함되도록 한다.
  - @Inherited : 애노테이션이 sub class에 상속되도록 한다.
  - @Repeatable : 애노테이션을 여러 번 명시할 수 있다.
    ```java
    @Repeatable(AnnotationName.class)@Repeatable(ToDos.class)
    @interface ToDo {
        String value();
    }
    
    @ToDo("..")
    @ToDo("override inherited methods")
    class MyClass{
       ..
    }
    ```
    - 애노태에션 여러 개가 하나의 대상에 적용될 수 있기 때문에, 하나로 묶어서 다룰 수 있는 애노테이션을 추가로 정의해야 한다
    ```java
    @interface ToDos{   // 여러 개의 ToDo 애노테이션을 담을 컨테이너 애노테이션
	    ToDo[] value(); // ToDo애너테이션 배열타입의 요소를 선언하고, "이름이 반드시 value"
    }

    @Repeatable(ToDos.class)
    @interface ToDo {
        String value();
    }
    ```
    
## Annotation processor

소스코드 레벨에서 소스코드에 있는 애노테이션을 읽어 컴파일러가 컴파일 하는 중에 새로운 소스코드를 생성하거나 기존 소스코드를 바꾸는 것

Java의 라이브러리인 Lombok을 예로 들어보자.

```java
// Lombok의 @Data를 명시한 코드
@Data
public class lombokEx {
    private int num;
}

// @Data 애노테이션이 적용된 실제 코드
public class lombokEx {
  private int num;

  public lombokEx() {
  }

  public int getNum() {
    return this.num;
  }

  public void setNum(final int num) {
    this.num = num;
  }

  public boolean equals(final Object o) {
    if (o == this) {
      return true;
    } else if (!(o instanceof lombokEx)) {
      return false;
    } else {
      lombokEx other = (lombokEx)o;
      if (!other.canEqual(this)) {
        return false;
      } else {
        return this.getNum() == other.getNum();
      }
    }
  }

  protected boolean canEqual(final Object other) {
    return other instanceof lombokEx;
  }

  public int hashCode() {
    int PRIME = true;
    int result = 1;
    int result = result * 59 + this.getNum();
    return result;
  }

  public String toString() {
    return "lombokEx(num=" + this.getNum() + ")";
  }
}
```

- @Data 애노테이션이 명시되면 Java는 이 애노테이션이 명시된 곳을 찾는다.
- 이 때, 컴파일 시점에 애노테이션 프로세서를 사용하여 Abstract Syntax Tree를 조작한다.
- 이렇게 조작된 AST를 컴파일하게 되고, 이를 통해 getter, setter, equals, hashCode, toString를 생성한다.

## 참고자료
- https://five-cosmos-fb9.notion.site/37d183f38389426d9700453f00253532
- https://b-programmer.tistory.com/264
 