# 챕터8 경계

![Boundary.jpg](/assets/images/Boundary.jpg?raw=true)

시스템에 들어가는 모든 소프트웨어를 직접 개발하는 경우는 드물다.

오픈소스를 이용하거나, 사내 다른팀이 제공하는 컴포넌트를 이용하거나, 어떤 식으로든 이 외부 코드를 우리 코드에 깔끔하게 통합해야만 한다.

## 외부 코드 사용하기

- 패키지 제공자나 프레임워크 제공자는 적용성을 최대한 넓히려 애쓴다.
- 사용자는 자신의 요구에 집중하는 인터페이스를 바란다.

### 예시) java.util.Map

Map은 굉장히 다양한 인터페이스로 수많은 기능을 제공한다.

Map이 제공하는 기능성과 유연성은 확실히 유용하지만 그만큼 위험도 크다.

예를들어 Map의 `.clear` 매서드는 Map 사용자라면 누구나 Map 내용을 지울 권한이 있다.

설계시 Map에 특정 객체 유형만 저장하기로 결정했지만 Map은 객체 유형을 제한하지 않는다. 마음만 먹으면 사용자는 어떤 객체 유형도 추가할 수 있다.

```java
Map sensors = new HashMap();
Sensor s = (Sensor) sensors.get(sensorId);
```

위 코드를 보면 Map이 반환하는 Object를 올바른 유형으로 변환할 책임은 Map을 사용하는 클라이언트에 있다는 것을 알 수 있다.

깨끗한 코드도 아니고 의도도 분명히 드러나지 않는다. 그 대신 제네릭을 사용하면 코드 가독성이 크게 높아져 좋다.

```java
Map<String, Sensor> sensors = new HashMap<Sensor>();
...
Sensor s = sensors.get(sensorId);
```

그렇지만 이 방법도 사용자에게 필요하지 않은 기능까지 제공한다는 문제는 해결하지 못한다.

프로그램에서 Map<String,Sensor> 인스턴스를 여기저기로 넘긴다면, Map 인터페이스가 변할 경우, 수정할 코드가 상당히 많아진다.

그래서 좀더 깔끔하게 제네릭을 Sensor객체 안에 넣으면 사용자는 제네릭이 사용되었는지 신경 쓸 필요가 없다.

```java
public class Sensors { 
    private Map sensors = new HashMap();

    public Sensor getById(String id) { 
        return (Sensor) sensors.get(id); 
    } 
    // 이하 생략 
}
```

경계 인터페이스인 Map을 Sensor 안으로 숨긴다. 따라서 Map 인터페이스가 변하더라도 나머지 프로그램에는 영향을 미치지 않는다. 제네릭을 사용하든 않하든 더 이상 문제가 안 된다. Sensor 클래스 안에서 객체 유형을 관리하고 반환하기 때문이다.

Map 클래스를 사용할 때마다 위와 같이 캡슐화하라는 소리가 아니다. Map을 여기저기 넘기지 말라는 말이다. Map과 같은 경계 인터페이스를 이용할 때는 이를 이용하는 클래스나 클래스 계열 밖으로 노출되지 않도록 주의한다. Map 인스턴스를 공개 API의 인수로 넘기거나 반환값으로 사용하지 않는다.

## 경계 살피고 익히기

외부 코드를 사용하면 적은 시간에 더 많은 기능을 출시하기 쉬워진다. 외부 패키지 테스트는 우리 책임이 아니지만 우리 자신을 위해 우리가 사용할 코드는 테스트하는 편이 바람직하다.

타사 라이브러리를 가져왔으나 사용법이 분명치 않다고 가정하자. 대개는 하루나 이틀 문서를 읽으며 사용법을 결정한다. 그런 다음 우리쪽 코드를 작성해 라이브러리가 예상대로 동작하는지 확인한다.

외부 코드를 익히기는 어렵다. 외부코드를 통합하기도 어렵다.

곧바로 우리쪽 코드를 작성해 외부 코드를 호출하는 대신 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히면 어떨까? 짐 뉴커크는 이를 학습 테스트라 부른다.

학습 테스트는 프로그램에서 사용하려는 방식대로 외부 API를 호출한다. 통제된 환경에서 API를 제대로 이해하는지를 확인하는 셈이다. 학습 테스트는 API를 사용하려는 목적에 초점을 맞춘다.

## log4j 익히기

로깅 기능을 직접 구현하는 대신 아파치의 log4j 패키지를 사용하려 한다고 가정한다. 문서를 자세히 읽기 전에 먼저 첫 번째 테스트 케이스를 작성한다.

```java
@Test
public void testLogCreate() { 
    Logger logger = Logger.getLogger("MyLogger");
    logger.info("hello");
}
```

테스트를 돌렸더니 Appender라는 뭔가가 필요하다는 오류가 발생해서 ConsoleAppender라는 클래스가 필요하다는 것을 알았다.

```java
@Test
public void testLogAddAppender() { 
    Logger logger = Logger.getLogger("MyLogger"); 
    ConsoleAppender appender = new ConsoleAppender(); 
    logger.addAppender(appender); 
    logger.info("hello"); 
}
```

이번에는 Appender에 출력 스트림이 없다는 사실을 발견한다. 구글에서 검색한 후 다음과 같이 시도한다.

```java
@Test 
public void testLogAddAppender() { 
    Logger logger = Logger.getLogger("MyLogger"); 
    logger.removeAppAppenders(); 
    logger.addAppender(new ConsoleAppender( 
        new PatternLayout("%p %t %m%n"), ConsoleAppender.SYSTEM_OUT));
    logger.info("hello"); 
}

```

이제야 제대로 돌아간다. "hello"가 들어간 로그 메시지가 콘솔에 찍힌다. ConsoleAppender에게 콘솔에 쓰라고 알려야 한다니 뭔가 수상하다.

흥미롭게도 ConsoleAppender.SYSTEM_OUT 인수를 제거했더니 문제가 없다. PatternLayout을 제거했더니 출력 스트림이 없다는 오류가 뜬다. 버그이거나 일관성 부족으로 보인다.

문서를 좀 더 자세히 읽어보고 구글을 뒤져보면서 log4j가 돌아가는 방식을 이해하고 여기서 얻은 지식을 간단한 단위 테스트 케이스 몇 개로 표현했다.

```java
public class LogTest { 
    private Logger logger; 

    @Before public void initialize() { 
        logger = Logger.getLogger("logger"); 
        logger.removeAllAppenders(); 
        Logger.getRootLogger().removeAllAppenders(); 
    } 

    @Test 
    public void basicLogger() { 
        BasicConfigurator.configure(); 
        logger.info("basicLogger"); 
    } 

    @Test 
    public void addAppenderWithStream() { 
        logger.addAppender(new ConsoleAppender(new PatternLayout("%p %t %m%n"), ConsoleAppender.SYSTEM_OUT)); 
        logger.info("addAppenderWithStream"); 
    } 

    @Test 
    public void addAppenderWithoutStream() { 
        logger.addAppender(new ConsoleAppender(new PatternLayout("%p %t %m%n"))); 
        logger.info("addAppenderWithoutStream"); 
    } 
}
```

지금까지 간단한 콘솔 로거를 초기화하는 방법을 익혔으니, 이제 모든 지식을 독자적인 로거 클래스로 캡슐화한다. 그러면 나머지 프로그램은 log4j 경계 인터페이스를 몰라도 된다.

## 학습 테스트는 공짜 이상이다

### 학습 테스트에 드는 비용은 없다

어쨌든 API를 배워야 하므로 오히려 필요한 지식만 확보하는 손쉬운 방법이다. 학습 테스트는 이해도를 높여주는 정확한 실험이다.

### 학습 테스트는 공짜 이상이다

투자하는 노력보다 얻는 성과가 더 크다. 패키지 새 버전이 나온다면 합습 테스트를 돌려 차이가 있는지 확인한다.

### 학습 테스트는 패키지가 예상대로 도는지 검증한다

일단 통합한 이후라고 하더라도 패키지가 우리 코드와 호환되리라는 보장은 없다. 패키지 작성자에게 코드를 변경할 필요가 생길지도 모른다.

패키지 작성자는 버그를 수정하고 기능도 추가한다. 패키지 새 버전이 나올 때마다 새로운 위험이 생긴다.

새 버전이 우리 코드와 호환되지 않으면 학습 테스트가 이 사실을 곧바로 밝혀낸다.

---

학습 테스트를 이용한 학습이 필요하든 그렇지 않든, 실제 코드와 동일한 방식으로 인터페이스를 사용하는 테스트 케이스가 필요하다.

이런 경계 테스트가 있다면 패키지의 새 버전으로 이전하기 쉬워진다. 그렇지 않다면 낡은 버전을 필요 이상으로 오래 사용하려는 유횩에 빠지기 쉽다.

## 아직 존재하지 않는 코드를 사용하기

경계와 관련해 또 다른 유형은 아는 코드와 모르는 코드를 분리하는 경계다.

때로는 우리 지식이 경계를 너머 미치지 못하는 코드 영역도 있다.

때로는 알려고 해도 알 수가 없다.

때로는 더 이상 내다보지 않기로 결정한다.

### 예시) 송신기

저쪽 팀이 아직 API를 설계하지 않았으므로 구체적인 방법은 모른다. 따라서 구현을 미루고 이쪽 코드를 진행하기 위해 자체적으로 인터페이스를 정의한다.

![transmit.png](/assets/images/transmit.png?raw=true)

인터페이스는 주파수와 자료 스트림을 입력으로 받는다. 즉, 우리가 바라는 인터페이스다.

우리가 바라는 인터페이스를 구현하면 우리가 인터페이스를 전적으로 통제한다는 장점이 생긴다. 코드 가독성도 높아지고 코드 의도도 분명해진다. 따라서 우리가 통제하지 못하고 정의되지도 않은 송신기 API에서 CommunicationsController를 분리했다.

저쪽 팀이 송신기 API를 정의한 후에는 TransmitterAdapter를 구현해 간극을 매웠다. ADAPTER 패턴으로 API 사용을 캡슐화해 API가 바뀔 때 수정할 코드를 한 곳으로 모았다.

이와 같은 설계는 테스트도 쉽다. 적절한 FakeTransmitter 클래스를 사용하면 CommunicationsController 클래스를 테스트할 수 있다. Transmitter API 인터페이스가 나온 다음 경계 테스트 케이스를 생성해 우리가 API를 올바로 사용하는지 테스트할 수도 있다.

## 깨끗한 경계

경계는 변경이 많다. 소프트웨어 설계가 우수하다면 변경하는데 많은 투자와 재작업이 필요하지 않다. 엄청난 시간과 노력과 재작업을 요구하지 않는다. 통제하지 못하는 코드를 사용할 때는 너무 많은 투자를 하거나 향후 변경 비용이 지나치게 커지지 않도록 각별히 주의해야 한다.

경계에 위치하는 코드는 깔끔히 분리한다. 또한 기대치를 정의하는 테스트 케이스도 작성한다. 이쪽 코드에서 외부 패키지를 세세하게 알아야 할 필요는 없다.

통제가 불가능한 외부 패키지에 의존하는 대신 통제가 가능한 우리 코드에 의존하는 편이 훨씬 좋다. 자칫하면 외부 코드에 휘둘리고 만다.

외부 패키지를 호출하는 코드를 가능한 줄여 경계를 관리하자. Map에서 봤듯이, 새로운 클래스로 경계를 감싸거나 아니면 ADAPTER 패턴을 이용해 우리가 원하는 인터페이스를 패키지가 제공하는 인터페이스로 변환하자. 어느 방법이든 코드 가독성이 높아지며, 경계 인터페이스를 사용하는 일관성도 높아지며, 외부 패키지가 변했을 때 변경할 코드도 줄어든다.

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 인사이트(2013), P144-152.
>