---
layout  : wiki
title   : 리플렉션보다는 인터페이스를 사용하라 
summary : 
date    : 2024-02-13 09:33:59 +0900
updated : 2024-02-13 10:09:53 +0900
tag     : java effectivejava
resource: 5A/A6D443-5CEF-4D70-B718-12684A5AC9D3
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## 리플렉션(Reflection)

java.lang.reflect 패키지를 이용하면 프로그램에서 임의 클래스에 접근할 수 있다.

Class 인스턴스가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 인스턴스를 가져올 수 있고, 이 인스턴스들로부터 그 클래스의 멤버 명, 필드 타입, 메서드 시그니처를 가져올 수 있다.

추가적으로 가져온 생성자, 메서드, 필드 인스턴스를 이용해 각각에 연결된 실제 생성자, 메서드, 필드를 조작할 수 있다. 

```java
// 리플렉션 예시
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ReflectionExample {

    public static void main(String[] args) throws Exception {
    
        // 클래스 가져오기
        Class<?> myClass = MyClass.class;

        // 생성자 가져오기
        Constructor<?> constructor = myClass.getConstructor(int.class);

        // 인스턴스 생성
        Object myObject = constructor.newInstance(10); // 10은 예로 주어진 인자

        // 필드 가져오기
        Field field = myClass.getDeclaredField("myField");
        field.setAccessible(true);

        // 필드 타입 가져오기
        Class<?> fieldType = field.getType();
        System.out.println("myField의 타입: " + fieldType.getName());

        // 필드 값 설정
        field.set(myObject, 20);

        // 필드 값 가져오기
        int value = (int) field.get(myObject);
        System.out.println("myField 값: " + value);

        // 메서드 가져오기
        Method method = myClass.getDeclaredMethod("myMethod");
        
        // 메서드 시그니처 가져오기
        String methodSignature = method.toString();
        System.out.println("myMethod의 시그니처: " + methodSignature);

        // 메서드 호출
        method.invoke(myObject);
    }
}

class MyClass {
    private int myField;

    public MyClass(int myField) {
        this.myField = myField;
    }

    public void myMethod() {
        System.out.println("myMethod 호출");
    }
}
```

리플렉션을 이용하면 컴파일 타임에 존재하지 않는 클래스도 이용할 수 있다.

Java에서 프로그램을 실행하는 동안 클래스를 동적으로 로드하고 인스턴스화할 수 있는 런타임 환경을 제공하여 동적으로 클래스를 로드하고 인스턴스화하여 해당 클래스의 메타데이터를 검사하고 조작할 수 있는 것이다.

## 단점과 사용법

- 컴파일 타임의 타입 검사가 주는 이점을 이용할 수 없다.

예외 검사도 마찬가지로, 리플렉션 기능을 사용해 존재하지 않거나 접근할 수 없는 메서드를 호출하려고 시도한다면 런타임에서 오류가 발생한다.

- 코드가 지저분하고 장황해짐

위 코드에서 봤듯이 읽기 어려우며 장황한 코드가 되기 쉽다.

- 성능 저하

리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.

이런 점 때문에 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만을 취할 수 있다. item65에서 제시하는 방법은 인스턴스 생성에만 사용하고, 생성된 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하는 방식이다. 

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.Arrays;
import java.util.Set;

public class RelectionEx {
    public static void main(String[] args) {
    
        // 클래스명 Class 객체로 변환
        Class<? extends Set<String>> cl = null;
    
        try {
            cl = (Class<? extends Set<String>>) // 비검사 형변환
                    Class.forName(args[0]); 
        } catch (ClassNotFoundException e) {
            fatalError("class not found");
        }

        // 생성자
        Constructor<? extends Set<String>> cons = null;
        try {
            cons = cl.getDeclaredConstructor();
        } catch (NoSuchMethodException e) {
            fatalError("can not found constructor");
        }

        // 집합의 인스턴스
        Set<String> s = null;
        try {
            s = cons.newInstance();
        } catch (IllegalAccessException e) {
            fatalError("can not access constructor"); 
        } catch (InvocationTargetException e) {
            fatalError("class throw exception:" + e.getCause());
        } catch (InstantiationException e) {
            fatalError("can not instantiate class");
        } catch (ClassCastException e) {
            fatalError("class not implement Set");
        }

        s.addAll(Arrays.asList(args).subList(1, args.length));
    }
}
```

예시에서는 Set<String> 인터페이스의 인스턴스를 생성하는데, 정확한 클래스는 명령줄의 첫 번째 인수로 확정하고, 생성한 Set에 두 번째 이후의 인수들을 추가한 뒤 다음 화면에 출력한다.

예는 리플렉션의 단점을 두 가지 보여주는데

- 런타임에 여섯 가지나 되는 예외를 던질 수 있다.

인스턴스를 리플렉션 없이 생성했다면 컴파일타임에 잡아낼 수 있었던 예외이기 때문이다.

- 클래스 이름만으로 인스턴스를 생성하기 위한 긴 코드를 작성

리플렉션을 사용하지 않았다면 생성자 호출 한 줄로 끝났을 것이다. 

또한 Java 7부터 지원하는 ReflectiveOperatinoException을 잡도록 하여 코드 길이를 줄일 수 있다.

객체가 생성되면 그 후의 코드는 여타의 Set 인스턴스 사용할 때와 똑같으며, 실제 프로그램에서는 이런 제약에 영향받는 코드는 일부에 지나지 않는다.

컴파일하면 비검사 형변환 경고가 발생하지만, 비검사 형변환 경고는 지워주는 것이 좋다.

이러한 특성을 이해하여 되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일 타임에 알 수 있는 상위 클래스로 형변환하여 사용하도록 하자!

## 참고자료

- 이펙티브자바 3판

