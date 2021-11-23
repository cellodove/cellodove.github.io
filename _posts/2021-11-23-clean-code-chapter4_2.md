---
title : "[클린 코드] 챕터4 주석2"
excerpt: "클린코드 정주행7"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2021-11-23
---

![note2.jpg](/assets/images/note2.jpg?raw=true)

## 4.4 나쁜 주석

대다수의 주석이 이 범주에 속한다. 일반적으로 대다수 주석은 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등 프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다.

### 주절거리는 주석

특별한 이유없이 의무감으로 혹은 프로세스에서 하라고 하니까 마지못해 주석을 단다면 전적으로 시간낭비다. 주석을 달기로 결정했다면 충분한 시간을 들여 최고의 주석을 달도록 노력한다.

주석을 제대로 달았다면 상당히 유용했을 코드다. 하지만 저자가 서둘렀거나 부주의했다. 그냥 주절거려놓아서 판독이 불가능하다.

```java
public void loadProperties()
{
  try
  {
    String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
    FileInputStream propertiesStream = new FileInputStream(propertiesPath);
    loadedProperties.load(propertiesStream);
  }
  catch(IOException e)
  {
    // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다.
  }
}
```

catch 블록에 있는 주석은 무슨 뜻일까? 저자에게야 의미가 있겠지만 그 의미가 다른 사람들에게 전해지지 않는다. Exception이 발생하면 속성 파일이 없다는 뜻이란다. 그러면 모든 기본값을 메모리로 읽어 들인 상태란다.

누가 모든 기본값을 읽어 들이는가?

loadProperties.load를 호출하기 전에 읽어 들이는가?

아니면 loadProperties.load가 파일을 읽어 들이기 전에 모든 기본값부터 읽어 들이는가?

저자가 catch 블록을 비워놓기 뭐해 몇 마디 덧붙였을 뿐인가?

아니면 나중에 돌아와서 기본값을 읽어 들이는 코드를 구현하려 했는가?

답을 알아내려면 다른 코드를 뒤져보는 수밖에 없다.이해가 안 되어 다른 모듈까지 뒤져야 하는 주석은 독자와 제대로 소통하지 못하는 주석이다.

### 같은 이야기를 중복하는 주석

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예의를 던진다.
public synchronized void waitForClose(final long timeoutMillis)
throws Exception
{
  if(!closed)
  {
    wait(timeoutMillis);
    if(!closed)
      throw new Exception("MockResponseSender could not be closed");
  }
}
```

주석의 내용이 코드의 내용과 중복하는 경우가있다. 자칫하면 코드보다 주석을 읽는 시간이 더 오래 걸린다. 주석이 코드보다 더 많은 정보를 제공하지 못한다. 코드를 정당화하는 주석도 아니고, 의도나 근거를 설명하는 주석도 아니다. 코드보다 읽기 쉽지도 않다. 실제로 코드보다 부정확해 독자가 함수를 대충 이해하고 넘어가게 만든다.

```java
/**
 * 이 컴포넌트를 위한 컨테이너 이벤트 Listener
 */
protected Loader loader = null;
```

이러한 주석은 코드만 지저분하고 정신없게 만든다.

### 오해할 여지가 있는 주석

때때로 의도는 좋았으나 프로그래머가 정확하지 못한 주석을 달기도 한다.

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예의를 던진다.
public synchronized void waitForClose(final long timeoutMillis)
throws Exception
{
  if(!closed)
  {
    wait(timeoutMillis);
    if(!closed)
      throw new Exception("MockResponseSender could not be closed");
  }
}
```

위의 주석은 중복이 상당히 많으면서도 오해할 여지가 살짝 있다. this.closed가 true로 바뀌는 시점에서 메소드는 반환되지 않는다. this.closed가 true이어야 메소드는 반환한다. 아니면 무조건 타임아웃을 기다렸다가 this.closed가 그래도 true가 아니면 예외를 던진다.

코드보다 읽기도 어려운 주석에 담긴 '살짝 잘못된 정보'로 인해 this.closed가 true로 변하는 순간에 함수가 반환되리라는 생각으로 어느 프로그래머가 경솔하게 함수를 호출할지 모른다.

### 의무적으로 다는 주석

모든 함수에 Javadocs를 달라거나 모든 변수에 주석을 달라는 규칙은 어리석기 그지없다. 이런 주석은 코드를 복잡하게 만들며, 거짓말을 퍼뜨리고, 혼동과 무질서를 초래한다.

아래와 같은 주석은 아무런 가치가 없다. 오히려 코드만 헷갈리게 만들며, 거짓말할 가능성을 높히며, 잘못된 정보를 제공할 여지만 만든다.

```java
/**
 *
 * @param title CD 제목
 * @param author CD 저자
 * @param tracks CD 트랙 숫자
 * @param durationInMinutes CD 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
    CD cd = new CD();
    cd.title = title;
    cd.author = author;
    cd.tracks = tracks;
    cd.duration = durationInMinutes;
    cdList.add(cd);
}
```

### 이력을 기록하는 주석

때때로 사람들은 모듈을 편집할 때마다 모듈 첫머리에 주석을 추가한다. 그리하여모듈 첫머리 주석은 지금까지 모듈에 가한 변경을 모두 기록하는 일종의 일지 혹은 로그가 된다. 첫머리 주석만 십여 쪽을 넘어서는 모듈도 보았다.

```java
* 변경 이력 (11-Oct-2001부터)
* ------------------------------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키징
* 05-Nov-2001: getDescription() 메소드 추가
* 12-Nov-2001: IBD가 setDesription() 메소드를 요구한다.
* ...
```

예전에는 모든 모듈 첫머리에 변경 이력을 기록하고 관리하는 관례가 바람직 했다. 당시는 원시 코드 제어 시스템이 없었으니까. 하지만 이제는 혼란만 가중할 뿐이다. 완전히 제거하는 편이 낫다.

### 있으나 마나 한 주석

때떄로 있으나 마나 한 주석을 접한다. 쉽게 말해, 너무 당연한 사실을 언급하며 새로운 정보를 제공하지 못하는 주석이다.

```java
/*
 * 기본 생성자
 */
protected AnnualDateRule() {
}

/** 달 중 날짜 */
private int dayOfMonth;
```

위와 같은 주석은 지나친 참견이라 개발자가 주석을 무시하는 습관에 빠진다. 코드를 읽으면서 자동으로 주석을 건너뛴다. 결국 코드가 바뀌면서 주석은 거짓말로 변한다.

### 무서운 잡음

때로는 Javadocs도 잡음이다. 다음은 잘 알려진 오픈 소스 라이브러리에서 가져온 코드다. 아래 나오는 Javadocs는 어떤 목적을 수행할까? 없다. 단지 문서를 제공해야 한다는 잘못된 욕심으로 탄생한 잡음일 뿐이다.

```java
/** The name. */
private String name;

/** The vsersion. */
private String vsersion;
```

### 함수나 변수로 표현할 수 있다면 주석을 달지 마라

```java
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (module.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

이 코드에서 주석을 없애고 다시 표현하면 다음과 같다.

```java
ArrayList moduleDependencies = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

코드를 작성한 저자는 주석을 먼저 달고 주석에 맞춰 코드를 작성했을지도 모르겠다. 하지만 위와 같이 주석이 필요하지 않도록 코드를 개선하는 편이 더 나았다.

### 위치를 표시하는 주석

때때로 프로그래머는 원시 파일에서 특정 위치를 표시하려고 주석을 사용한다.

```java
//Action//////////////////////////
```

극히 드물지만 위와 같은 배너 아래 특정 기능을 모아놓으면 유용한 경우도 있긴 있다. 하지만 일반적으로 위와 같은 주석은 가독성만 낮추므로 제거해야 마땅하다. 특히 뒷부분에 슬래시(/)로 이어지는 잡음은 제거하는 편이 낫다.

너무 자주 사용하지 않는다면 배너는 눈에 띄며 주의를 환기한다. 그러므로 반드시 필요할 때만, 아주 드물게 사용하는 편이 좋다. 배너를 남용하면 독자가 흔한 잡음으로 여겨 무시한다.

### 닫는 괄호에 다는 주석

때로는 프로그래머들이 닫는 괄호에 특수한 주석을 달아놓는다.

```java
try{
  while((line = in.readLine) != null){
    lineCount++;
    charCount += line.length();
    String words[] = line.split("a");
    wordCount += words.length;
  } //while
  System.out.println("wow");
  System.out.println("wow");
  System.out.println("wow");
} //try
```

중첩이 심하고 장황한 함수라면 의미가 있을지도 모르겠지만 작고 캡슐화된 함수에는 잡음일 뿐이다. 그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신 함수를 줄이려 시도하자.

### 공로를 돌리거나 저자를 표시하는 주석

```java
/* 릭이 추가함 */
```

주석이 있으면 다른 사람들이 코드에 관해 누구한테 물어볼지 아니까 위와 같은 주석이 이용하다 여길지도 모르겠다. 하지면 현실적으로 이런 주석은 그냥 오랫동안 코드에 방치되어 점차 부정확하고 쓸모없는 정보로 변하기 일쑤다.

### 주석으로 처리한 코드

주석으로 처리한 코드만큼 밉살스러운 관행도 드물다. 다음과 같은 코드는 작성하지 마라!

```java
InputStreamResponse response = new InputStreamResponse();
response.setBody(formatter.getResultStream(),formatter.getByteCount())
//InputSteam resultStream = formatter.getResultStream();
//StreamReader reader = new StreamReader(resultsSteam)
```

주석으로 처리된 코드는 다른 사람들이 지우기를 주저한다. 이유가 있어 남겨놓았으리라고, 중요하니까 지우면 안 된다고 생각한다.

이제는 알아서 전에썼던 코드를 시스템이 기억해준다. 주석으로 처리할 필요가 없다. 그냥 코드를 삭제하라. 잃어버릴 염려는 없다.

### HTML 주석

원시 코드에서 HTML 주석은 혐오 그 자체다. HTML 주석은 편집기/IDE에서조차 읽기가 어렵다. Javadocs와 같은 도구로 주석을 뽑아 웹 페이지에 올릴 작정이라면 주석에 HTML 태그를 삽입하는 책임은 프로그래머가 아니라 도구가 가져야 한다.

### 전역 정보

주석을 달아야 한다면 근처에 있는 코드만 기술하라. 코드 일부에 주석을 달면서 시스템 전반적인 정보를 기술하지 마라

```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnewssePort = fitnessePort;
}
```

심하게 중복되었다는 사실 외에도 주석은 기본 포트 정보를 기술한다. 하지만 함수 자체는 포트 기본값을 전혀 통제하지 못한다. 그러니까 아래함수가 아닌 시스템어딘가에 있는 다른 함수를 설명한다는 소리이다.

즉, 포트 기본값을 설정하는 코드가 변해도 아래 주석이 변하리라는 보장은 전혀 없다.

### 너무 많은 정보

주석에다 흥미로운 역사나 관련 없는 정보를 장황하게 늘어놓지 마라.

### 모호한 관계

주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다. 이왕 공들여 주석을 달았다면 적어도 독자가 주석과 코드를 읽어보고 무슨 소린지 알아야 하지 않겠는가

```java
/*
 * 모든 픽셀을 담을 만큼 충분한 배열로 시작 (여기에 필터 바이트를 더한다.)
 * 그리고 헤더 정보를 위해 200바이트를 더한다.
 */
this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200]
```

여기서 필터바이트란 무엇일까? +1과 관련이 있을까? 아니면 *3과 관련이 있을까? 아니면 둘 다? 200을 추가하는 이유는? 주석을 다는 목적은 코드만으로 설명이 부족해서다. 하지만, 주석 자체가 다시 설명을 요구하니 잘못된 주석이다.

### 함수 헤더

짧은 함수는 긴 설명이 필요 없다. 짧고 한 가지만 수행하며 이름을 잘 붙인 함수가 주석으로 헤더를 추가한 함수보다 훨씬 낫다.

### 비공개 코드에서 Javadocs

공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다. 시스템 내부에 속하는 클래스와 함수에 Javadocs를 생성할 필요는 없다. 코드만 보기 싫고 산만해질 뿐이다.

### 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 케이앤피북스(2010), P105-119.
>