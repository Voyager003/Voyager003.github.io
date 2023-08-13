---
layout  : wiki
title   : Spring 테스트 전략
summary : 
date    : 2023-08-07 20:45:42 +0900
updated : 2023-08-13 23:15:55 +0900
tag     : spring
resource: 48/1D2CE5-6975-4D07-84CA-A179F2E116BA
toc     : true
public  : true
parent  : [[/spring]] 
latex   : false
---
* TOC
{:toc}

## 개요

Spring은 다양한 테스트 기능을 제공하는데, 단위 테스트 및 통합 테스트를 어떻게 작성하는지 정리할 겸 글을 작성했다.

설명에 앞서 통합 테스트와 단위 테스트에 대한 정의와 Test double에 대해 숙지하고 가자.

### 단위 테스트(Unit Test) vs 통합 테스트(Integration Test)

먼저 단위 테스트의 대한 설명이다.

> 응용 프로그램에서 테스트 가능한 가장 작은 소프트웨어를 실행하여 예상대로 동작하는지 확인하는 테스트이다.

단위 테스트에서의 단위는 엄격하게 정해져 있지 않지만, 일반적으로 클래스, 메서드 수준으로 정해진다. 단위의 크기가 작을수록 단위의 복잡성은 낮아진다.

해당 부분만 독립적으로 테스트하여 어떤 코드를 리팩토링하더라도 빠르게 디버깅이 가능하다. 다음은 Junit5로 작성한 단위 테스트 코드 예시이다.

```java
@DisplayName("자동차가 전진한다")
@Test
public void moveCar() {
    // given
    Car car = new Car("dani");

    // when
    car.move(4);

    // then
    assertThat(car.getPosition()).isEqualTo(1);
}
```
 
이처럼 특정 클래스, 메서드(코드의 경우 move)를 테스트하는 것이 단위 테스트이다.

다음은 통합 테스트이다.

> 단위 테스트보다 더 큰 동작을 달성하기 위해 여러 모듈들을 모아 이들이 의도대로 협력하는지 확인하는 테스트이다.

단위 테스트와 달리 개발자가 변경할 수 없는 부분(외부 라이브러리)까지 묶어 검증할 때 사용한다. 이는 DB에 접근하거나 전체 코드와 다양한 환경이 제대로 작동하는지 확인하는데 필요한 모든 작업을 수행할 수 있다.

통합 테스트로 얻을 수 있는 장점은 Spring Container에 등록된 Bean을 가지고 테스트를 하기 때문에, 운영 환경과 유사하게 테스트 가능하며, API 테스트 시 요청에서 응답까지 전체적인 테스트를 할 수 있다. 

그에 따라 단점도 수반하는데, 그만큼 Bean을 등록하여 테스트를 진행하기 때문에 테스트 시간이 오래걸리며 무겁다는 점이다. 

### Test Double

Xunit Test Patterns의 저자 Gerad Meszaros가 만든 용어로 테스트를 진행하기 어려운 경우, 테스트를 진행할 수 있도록 만들어주는 객체를 말한다. 

Test Double의 종류는 다음과 같다.

- Dummy : 기본적인 Test Double로 인스턴스화 된 객체가 필요하지만 기능은 필요하지 않는 경우 사용한다. 인스턴스화된 객체가 필요해서 구현한 가짜 객체일 뿐, Dummy 객체는 정상적인 동작을 보장하지 않는다. 예시는 다음과 같다.

```java
interface DataService {
    int fetchData();
}

// Dummy 객체 구현
class DummyDataService implements DataService {

    @Override
    public int fetchData() {
        // 실제 데이터를 반환하지 않고, 단순히 0을 반환하는 dummy 메서드
        return 0;
    }
}
```

실제 객체는 DataService 인터페이스의 구현체가 필요하지만, 구현체의 동작이 필요하지 않을 수 있다. 이처럼 동작하지 않아도 테스트에는 영향을 미치지 않는 객체를 Dummy라 한다.

- Fake : 실제 동작을 가지지만, 실제 시스템에서 사용되는 것 보다 단순하거나 가벼운 형태로 구현된 객체이다. 주로 개발 환경에서 실제 DB나 외부 서비스와 연동하지 않고 테스트를 위해 가짜 데이터나 동작을 제공한다. 코드로 보자.

```java
// 인터페이스 정의
interface Database {
    void saveData(String data);
    List<String> getAllData();
}

// Fake 객체 구현
class FakeDatabase implements Database {
    private List<String> fakeData = new ArrayList<>();

    @Override
    public void saveData(String data) {
        fakeData.add(data);
    }

    @Override
    public List<String> getAllData() {
        return new ArrayList<>(fakeData);
    }
}

// 비즈니스 로직 클래스
class DataProcessor {
    private Database database;

    public DataProcessor(Database database) {
        this.database = database;
    }

    public void processAndSave(String data) {
        // 데이터 처리 로직
        // ...
        database.saveData(data);
    }
}

// 테스트
public static void main(String[] args) {
        // Fake 객체를 이용하여 DataProcessor를 테스트
        Database fakeDatabase = new FakeDatabase();
        DataProcessor dataProcessor = new DataProcessor(fakeDatabase);

        dataProcessor.processAndSave("Sample Data");

        List<String> savedData = fakeDatabase.getAllData();
        System.out.println("Saved Data: " + savedData); // 기대 결과: Saved Data: [Sample Data]
    }
```

Database 인터페이스를 구현한 Fake 객체인 FakeDatabase를 생성했다. 이 클래스는 내부적으로 가짜 데이터를 저장하기 위한 fakeData 리스트를 가진다. 

테스트에서 Fake 객체인 FakeDatabase를 사용해 DB와 상호작용을 했다. 실제 DB를 사용하지 않으면서 테스트를 한 것이다. 

이처럼 실제 객체와 동일한 역할을 하도록 만들어 사용하는 객체를 Fake라 한다.

- Stub : 특정 메서드 호출에 대해 미리 정의된 값을 반환하도록 구현된 객체다.

```java
// 인터페이스 정의
interface WeatherService {
    String getWeatherCondition(String location);
}

// Stub 객체 구현
class WeatherServiceStub implements WeatherService {

    @Override
    public String getWeatherCondition(String location) {
        // 특정 위치에 대한 날씨 상태를 Stub으로 반환
        if (location.equals("Seoul")) {
            return "Sunny";
        } else if (location.equals("New York")) {
            return "Cloudy";
        }
        return "Unknown";
    }
}

// 비즈니스 로직 클래스
class WeatherReporter {
    private WeatherService weatherService;

    public WeatherReporter(WeatherService weatherService) {
        this.weatherService = weatherService;
    }

    public String reportWeather(String location) {
        String weather = weatherService.getWeatherCondition(location);
        return "The weather in " + location + " is " + weather;
    }
}

// 테스트
public class TestExample {
    public static void main(String[] args) {
        // Stub 객체를 이용하여 WeatherReporter를 테스트
        WeatherServiceStub weatherServiceStub = new WeatherServiceStub();
        WeatherReporter weatherReporter = new WeatherReporter(weatherServiceStub);

        String report = weatherReporter.reportWeather("Seoul");
        System.out.println(report); // 기대 결과: The weather in Seoul is Sunny
    }
}
```

WeatherServiceStub은 WeatherService를 구현해 특정 위치에 대한 날씨 조건을 Stub으로 반환하고, WeatherReporter는 WeatherSErvice를 주입받아 날씨 정보를 보고하는 기능을 제공한다. 

Stub 객체인 WeatherServiceStub을 사용해 특정 위치에 대한 날씨 정보를 제어하고 WeatherReporter의 동작을 테스트한다. 이처럼 테스트를 위해 의도한 결과를 반환되도록 하기 위한 객체를 Stub이라 한다.

- Spy : Stub의 역할을 가지면서 실제 객체의 메서드 호출 횟수나 입력값 및 출력값을 확인하면서 객체의 내부 상태를 검증하기 위해 사용되는 객체이다.

```java
// 실제 객체 클래스
class DataService {
    private List<Integer> data = new ArrayList<>();

    public void addData(int value) {
        data.add(value);
    }

    public List<Integer> getData() {
        return new ArrayList<>(data);
    }
}

// Spy 객체 구현
class DataServiceSpy extends DataService {
    private int addDataCallCount = 0;

    @Override
    public void addData(int value) {
        super.addData(value);
        addDataCallCount++;
    }

    public int getAddDataCallCount() {
        return addDataCallCount;
    }
}

// 테스트 코드
public class TestExample {
    public static void main(String[] args) {
        // Spy 객체를 이용하여 DataService를 테스트
        DataServiceSpy dataServiceSpy = new DataServiceSpy();

        dataServiceSpy.addData(5);
        dataServiceSpy.addData(10);

        List<Integer> data = dataServiceSpy.getData();
        int callCount = dataServiceSpy.getAddDataCallCount();

        System.out.println("Data: " + data); // 기대 결과: Data: [5, 10]
        System.out.println("Add Data Call Count: " + callCount); // 기대 결과: Add Data Call Count: 2
    }
}
```

DataService는 실제 데이터를 추가하고 조회하는 메서드를 포함한 클래스이며, DataServcieSpy는 메서드 호출 횟수를 관찰하는 역할을 한다. addData 메서드를 호출하면서 호출 횟수를 관찰하고 데이터를 조회환 결과와 호출 횟수를 출력한다.

이처럼 자기 자신이 호출된 상황을 확인할 수 있는 객체를 Spy라 한다.

- Mock : 특정 동작을 시뮬레이션하거나 원하는 결과를 조작해 테스트하는 데 사용된다. 특히 메서드 호출을 검증하거나 기대한 상호작용을 확인하기 위해 사용된다. 
  
```java
// 인터페이스 정의
interface EmailService {
    void sendEmail(String to, String subject, String body);
}

// Mock 객체를 사용한 테스트
public class TestExample {
    public static void main(String[] args) {
        // Mock 객체 생성
        EmailService emailServiceMock = mock(EmailService.class);

        // Mock 객체에 기대한 동작 설정
        when(emailServiceMock.sendEmail("test@example.com", "Hello", "Hi there!")).thenReturn(true);

        // 테스트 대상 객체 생성 및 Mock 객체 주입
        NotificationService notificationService = new NotificationService(emailServiceMock);

        // 테스트 수행
        boolean result = notificationService.sendNotification("test@example.com", "Hello", "Hi there!");

        // Mock 객체 동작 검증
        verify(emailServiceMock, times(1)).sendEmail("test@example.com", "Hello", "Hi there!");

        System.out.println("Notification Result: " + result); // 기대 결과: Notification Result: true
    }
}

// 테스트 대상 클래스
class NotificationService {
    private EmailService emailService;

    public NotificationService(EmailService emailService) {
        this.emailService = emailService;
    }

    public boolean sendNotification(String to, String subject, String body) {
        emailService.sendEmail(to, subject, body);
        return true;
    }
}
```

Mock 객체를 사용해 EmailService의 메서드 호출과 결과를 시뮬레이션하고 기대한 동작을 검증했다. 

어찌보면 Stub과 같아 보이지만 Martin Fowler는 이 둘이 다르다고 설명한다.

먼저 Stub의 경우 상태 검증(State Verification)을 하고 Mock의 경우 행위 검증(Behavior Verification)에 초점을 맞춘다는 점이다.

예를 들어 테스트 중에 은행 시스템에서 계좌 잔액 조회를 호출하고 돈을 인출하는 동작을 테스트한다고 가정해보자. 

여기서 Stub을 사용하면 계좌 잔액 조회 결과를 조작하여 어떤 상황에서든 인출이 가능하도록 만들 수 있다. 

그러나 Mock을 사용하면 호출된 메서드가 예상한 대로 호출되는지, 호출 횟수가 올바른지 등을 검증할 수 있다.

상태와 행위를 검증한다는 차이점에 초점을 두자.

## 



## 참고자료

- https://tecoble.techcourse.co.kr/post/2021-05-25-unit-test-vs-integration-test-vs-acceptance-test/ - Tecoble의 단위테스트 vs 통합테스트
- https://martinfowler.com/bliki/TestDouble.html - Test Double
- https://martinfowler.com/articles/mocksArentStubs.html - Mocks aren't stub

