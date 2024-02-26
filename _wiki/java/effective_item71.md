---
layout  : wiki
title   : 필요 없는 검사 예외 사용은 피하라 
summary : 
date    : 2024-02-26 09:57:32 +0900
updated : 2024-02-26 11:59:40 +0900
tag     : java effectivejava
resource: 76/DE23DA-71A0-4E04-884E-D5E93C7B36E6
toc     : true
public  : true
parent  : [[/java]]
latex   : false
---
* TOC
{:toc}

## Java의 검사 예외(checked exception)

Java에서 어떤 메서드가 검사 예외를 던질 수 있다고 선언됐다면, 이를 호출하는 코드에서는 catch 블록을 두어 예외를 잡아 처리하거나 문제를 전파해야하며, 메서드는 스트림 안에서 직접 사용할 수 없어 Java 8부터는 부담이 더욱 커졌으며 API 사용자에게 부담을 준다.

따라서 API를 제대로 사용해도 발생할 수 있는 예외이거나, 의미있는 조치를 취할 수 있는 경우에 해당하지 않는다면 `비검사 예외(unchecked exception)` 를 사용하는 것이 좋다.

## 검사 예외를 피하는 법

- 적절한 결과 타입을 담은 Optional을 반환

검사 예외를 던지는 대신 단순히 비어있는 Optional을 반환하는 방법이다.

```java
import java.util.Optional;

public class ExampleService {
    public Optional<String> fetchData() {
        try {
            // 만약 데이터를 성공적으로 가져오면 해당 데이터를 Optional로 감싸서 반환
            String data = fetchDataFromExternalSource();
            return Optional.ofNullable(data);
        } catch (Exception e) {
            // 데이터를 가져오는 도중 예외가 발생하면 빈 Optional을 반환
            return Optional.empty();
        }
    }

    private String fetchDataFromExternalSource() throws Exception {
        return "Sample Data";
    }
}
```

Optional은 예외 발생 이유를 알려주는 부가 정보를 담을 수 없다는 특징이 있다. 그에 반면 예외를 사용하면 구체적 예외 타입과 타입이 제공하는 메서드를 활용해 부가 정보를 제공할 수 있다.

- 검사 예외를 던지는 메서드를 2개로 쪼개 비검사 예외로 변경

```java
if (obj.actionPermitted(args)) {
    obj.action(args);
} else {
    thorw new UnChecked.. // 예외 처리 대처
}
```

예외를 던져질지 판단하는 여부를 boolean(actionPermitted)값으로 반환하는 방법이다. 

여기서 actionPermitted는 상태 검사 메서드에 해당하므로 [item69](https://voyager003.github.io/wiki/java/effective_item69/#%EB%8B%A4%EB%A5%B8-%EC%84%A0%ED%83%9D%EC%A7%80) 에서 다뤘듯이 외부 동기화없이 여러 스레드가 동시 접근할 수 있거나 외부 요인에 의해 상태가 변할 수 있다면 적절하지 않은 방법이다.

추가적으로 실패 시 스레드를 중단하길 원한다면 

```java
obj.action(args);
```

위 처럼 한 줄로 작성해도 무방하다.

## 참고자료

- 이펙티브자바 3판

