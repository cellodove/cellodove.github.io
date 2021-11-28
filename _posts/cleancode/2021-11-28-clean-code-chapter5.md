---
title : "[클린 코드] 챕터5 형식 맞추기"
excerpt: "클린코드 정주행8"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2021-11-28
---

![cat.jpg](/assets/images/cat.jpg?raw=true)

코드를 열었을 때 독자들이 코드가 깔끔하고, 일관적이며, 꼼꼼하다고 감탄하면 좋겠다.

질서 정연하다고 탄복하면 좋겠다.

모듈을 읽으며 두 눈이 휘둥그레 놀라면 좋겠다.

전문가가 짰다는 인상을 심어주면 좋겠다.

프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야 한다. **코드 형식을 맞추기 위한 간단한 규칙을 정하고 그 규칙을 착실히 따라야 한다.** **팀으로 일한다면 팀이 합의해 규칙을 정하고 모두가 그 규칙을 따라야 한다.** 필요하다면 규칙을 자동으로 적용하는 도구를 활용한다.

## 5.1 형식을 맞추는 목적

코드 형식은 중요하다! 너무 중요해서 무시하기 어렵다. 코드 형식은 의사소통의 일환이다. 의사소통은 전문 개발자의 일차적인 의무다.

어쩌면 '돌아가는 코드'가 전문 개발자의 일차적인 의무라 여길지도 모르겠다. 하지만 이 글을 읽으면서 생각이 바뀌었기 바란다. **오늘 구현한 기능이 다음 버전에서 바뀔 확률은 아주 높다. 그런데 오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다.**

오랜 시간이 지나 코드의 흔적을 더 이상 찾아보기 어려울 정도로 코드가 바뀌어도 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장성에 계속해서 영향을 미친다.

원래 코드는 사라질지라도 개발자의 스타일과 규율은 사라지지 않는다. 그렇다면 원활한 소통을 장려하는 코드 형식은 무엇일까?

## 5.2 적절한 행 길이를 유지하라

세로 길이부터 살펴보자. 소스 코드는 얼마나 길어야 적당할까? 자바에서 파일 크기는 클래스 크기와 밀접하다. 500줄을 넘지 않고 대부분 200줄 정도인 파일로도 커다란 시스템을 구출할 수 있다. 반드시 지킬 엄격한 규칙은 아니지만 바람직한 규칙으로 삼으면 좋겠다. **일반적으로 큰 파일보다 작은 파일이 이해하기 쉽다.**

### 신문 기사처럼 작성하라

아주 좋은 신문 기사를 떠올려보라. 독자는 위에서 아래로 기사를 읽는다. 가장 상단에 기사를 몇 마디로 요약하는 표제가 나온다. 독자는 표제를 보고 기사를 읽을지 말지 결정한다.

첫 문단은 전체 기사 내용을 요약한다. 세세한 사실은 숨기고 커다란 그림을 보여준다. 쭉 읽으면서 내려가면 세세한 사실이 조금씩 드러난다. 날짜, 이름, 발언, 주장, 기타 세부 사항이 나온다.

소스 파일도 신문 기사와 비슷하게 작성한다. 이름은 간단하면서도 설명적으로 짓는다. 이름만 보고도 올바른 모듈을 살펴보고 있는지 아닌지를 판단할 정도로 신경 써서 짓는다. 소스파일 첫 부분은 고차원 개념과 알고리즘을 설명한다. 아래로 내려갈수록 의도를 세세하게 묘사한다. 마지막에는 가장 저차원 함수와 세부 내역이 나온다.

신문은 다양한 기사로 이뤄진다. 대다수 기사가 아주 짧다. 어떤 기사는 조금 길다. 한 면을 채우는 기사는 거의 없다. 신문이 읽을 만한 이유가 여기에 있다.

**신문이 사실, 날짜, 이름등을 무작위로 뒤섞은 긴 기사 하나만 싣는다면 아무도 읽지 않으리라.**

### 개념은 빈 행으로 분리하라

거의 모든 코드는 왼쪽에서 오른쪽으로 그리고 위에서 아래로 읽힌다. 각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이는 빈 행을 넣어서 분리해야 마땅하다.

각 함수 사이에 빈 행이 들어가는 너무나 간단한 규칙이지만 코드의 세로 레이아웃에 영향을 미친다.

```java
// 빈 행 X
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
        Pattern.MULTILINE + Pattern.DOTALL);
    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));}
    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

```java
// 빈 행 O
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
    public static final String REGEXP = "'''.+?'''";
    private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
        Pattern.MULTILINE + Pattern.DOTALL
    );

    public BoldWidget(ParentWidget parent, String text) throws Exception {
        super(parent);
        Matcher match = pattern.matcher(text);
        match.find();
        addChildWidgets(match.group(1));
    }

    public String render() throws Exception {
        StringBuffer html = new StringBuffer("<b>");
        html.append(childHtml()).append("</b>");
        return html.toString();
    }
}
```

빈 행은 새로운 개념을 시작한다는 시각적 단서다. 코드를 읽어 내려가면 빈 행 바로 다음 줄에 눈길이 멈춘다.

단지 새로 여백만 다를 뿐이지만 첫 번째 코드는 전체가 한 덩어리로 보이고 두번째 코드는 행 묶음이 분리되어 보인다.

### 세로 밀집도

세로 여백이 개념을 분리한다면 세로 밀집도는 연관성을 의미한다. 즉, 서로 밀접한 코드행은 세로로 가까이 놓아야 한다는 뜻이다.

아래 코드를 보면 의미 없는 주석으로 두 인스턴스 변수를 떨어뜨려 놓았다.

```java
public class ReportConfig {
    /*
     * 리포터 리스너의 클래스 이름 
     */
    private String m_className;
    
    /*
     * 리포터 리스너의 속성
     */
    private List<Property> m_properties = new ArrayList<>();
    public void addProperty(Property property){
        m_properties.add(property); 
    }
}
```

이 코드가 훨씬 읽기 쉽다. 코드가 '한눈'에 들어온다. 머리나 눈을 움직일 필요가 없다.

```java
public class ReportConfig {
    
    private String m_className;
    private List<Property> m_properties = new ArrayList<>();
    
    public void addProperty(Property property){
        m_properties.add(property);
    }
}
```

### 수직 거리

함수 연관 관계와 동작 방식을 이해하려고 이 함수에서 저 함수로 오가며 소스 파일을 위아래로 뒤지는 등 뺑뺑이를 돌았으나 결국은 미로 같은 코드 때문에 혼란만 가중된 경험이 있는가?

함수나 변수가 정의된 코드를 찾으려고 상속 관계를 줄중이 거슬러 올라간 경험이 있는가? 결코 달갑지 않은 경험이다. 시스템이 무엇을 하는지 이해하고 싶은데, 이 조각 저 조각이 어디에 있는지 찾고 기억하느라 시간과 노력을 소모한다.

서로 밀접한 개념은 세로로 가까이 두어야 한다. 물론 두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않는다. 하지만 타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다.

같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성을 표현한다. **연관성이 깊은 두 개념이 멀리 떨어져 있으면 코드를 읽는 사람이 소스 파일과 클래스를 여기저기 뒤지게 된다.**

- 변수 선언

변수는 사용하는 위치에 최대한 가까이 선언한다. 우리가 만든 함수는 매우 짧으므로 지역 변수는 각 함수 맨 처음에 선언한다.

- 인스턴스 변수

반면에 인스턴스 변수는 클래스 맨 처음에 선언한다. 변수 간에 세로로 거리를 두지 않는다. 잘 설계한 클래스는 많은 클래스 메소드가 인스턴스 변수를 사용하기 때문이다.

인스턴스 변수를 선언하는 위치는 아직도 논쟁이 분분하나 잘 알려진 위치에 인스턴스 변수를 모은다는 사실이 중요하다. **변수 선언을 어디서 찾을지 모두가 알고 있어야 한다.**

- 종속 함수

**한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 또한 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다.** 그러면 프로그램이 자연스럽게 읽힌다. 규칙을 일관적으로 적용한다면 독자는 방금 호출한 함수가 잠시 후에 정의되리라는 사실을 예측한다.

- 개념적 유사성

어떤 코드는 서로를 끌어당긴다. 개념적 친화도가 높기 때문이다. 친화도가 높을수록 코드를 가까이 배치한다.

친화도가 높은 요인은 여러 가지다. 앞서 보았듯이, 한 함수가 다른 함수를 호출해 생기는 직접적인 종속성이 한 예다. 변수와 그 변수를 사용하는 함수도 한 예다. 하지만 그 외에도 친화도를 높이는 요인이 있다. 비슷한 동작을 수행하는 일군의 함수가 좋은 예다.

### 세로 순서

일반적으로 함수 호출 종속성을 아래 방향으로 유지한다. 다시 말해서, 호출되는 함수를 호출하는 함수보다 나중에 배치한다. 그러면 소스 코드 모듈이 고차원에서 저차원으로 자연스럽게 내려간다.

신문 기사와 마찬가지로 가장 중요한 개념을 가장 먼저 표현한다. 가장 중요한 개념을 표현할 때는 세세한 사항을 최대한 배제한다. 세세한 사항은 가장 마지막에 표현한다. 그러면 독자들이 소스 파일에서 첫 함수 몇개만 읽어도 개념을 파악하기 쉬워진다.

## 5.3 가로 형식 맞추기

한 행은 가로로 얼마나 길어야 적당할까? 항상 그랬듯이 짧은 행이 바람직하다. 예전에는 오른쪽으로 스크롤할 필요가 절대로 없도록 코드를 짰다. 하지만 요즘은 모니터가 아주 크다. 개인적으로 120자 정도로 행 길이를 제한한다.

### 가로 공백과 밀집도

가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.

```java
private void measureLine(String line) { 
    lineCount++;

    // 흔히 볼 수 있는 코드인데, 할당 연산자 좌우로 공백을 주어 왼쪽,오른쪽 요소가 확실하게 구분된다.
    int lineSize = line.length();
    totalChars += lineSize; 

    // 반면 함수이름과 괄호 사이에는 공백을 없앰으로써 함수와 인수의 밀접함을 보여준다
    // 괄호 안의 인수끼리는 쉼표 뒤의 공백을 통해 인수가 별개라는 사실을 보여준다.
    lineWidthHistogram.addLine(lineSize, lineCount);
    recordWidestLine(lineSize);
}
```

할당 연산자를 강조하려고 앞뒤에 공백을 주었다. 할당문은 왼쪽 요소와 오른쪽 요소가 분명히 나뉜다. 공백을 넣으면 두 가지 주요 요소가 확실히 나눠진다는 사실이 더욱 분명해진다.

반면에 함수 이름과 이어지는 괄호 사이에는 공백을 안 넣었다. 함수와 인수는 서로 밀접하기 때문이다.

연산자의 우선순위를 강조하기 위해서도 공백을 사용한다.

```java
return b*b - 4*a*c;
return (-b - Math.sqrt(determinant)) / (2*a);
```

수식을 읽기가 아주 편하다.

### 가로 정렬

어셈블리어 프로그래머였던 시절에 필자는 특정 구조를 강조하고자 가로 정렬을 사용했다. 나중에 자바로 프로그램을 짜면서도 필자는 계속해서 선언부의 변수 이름이나 할당문의 오른쪽 피연산자를 빠뜨리지 않고 나란히 정렬했다.

```java
public class FitNesseExpediter implements ResponseSender {
    private Socket socket;
    private InputStream input;
    private OutputStream output;
    private Reques request;
    private Response response;
    private FitNesseContex context; 
    protected long requestParsingTimeLimit;
    private long requestProgress;
    private long requestParsingDeadline;
    private boolean hasError;
    ...
}
```

보기엔 깔끔해 보일지 모르나, 코드가 엉뚱한 부분을 강조해 변수 유형을 자연스레 무시하고 이름부터 읽게 된다. 게다가 Code Formatter 대부분들은 이렇게 해놔봤자 무시하고 원래대로 돌려놓는다 그러므로 선언문과 할당문을 별도로 정렬할 필요가 없다.

정렬이 필요할 정도로 목록이 길다면 목록의 길이가 문제이지 정렬이 부족해서가 아니다. 선언부가 길다는 것은 클래스를 쪼개야 한다는 것을 의미한다.

### 들여쓰기

우리는 범위로 이루어진 계층을 표현하기 위해 코드를 들여쓴다. 들여쓰기를 잘해놓으면 코드읽기가 한결 쉬워진다. 구조가 한 눈에 들어온다. 들여쓰기가 없다면 인간이 코드를 읽기란 거의 불가능 하리라.

- 들여쓰기 무시하기

간단한 if문, while문, 짧은 함수에서 들여쓰기를 무시하고픈 유혹이 생긴다. 하지만 들여쓰기로 제대로 범위를 표현한 코드가 가독성이 더 높다.

```java
//들여쓰기 X
public class CommentWidget extends TextWidget {
    public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";

    public CommentWidget(ParentWidget parent, String text){super(parent, text);}
    public String render() throws Exception {return ""; } 
}
```

```java
//들여쓰기 O
public class CommentWidget extends TextWidget {
    public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";

    public CommentWidget(ParentWidget parent, String text){
        super(parent, text);
    }

    public String render() throws Exception {
        return ""; 
    } 
}
```

### 가짜 범위

때로는 빈 while 문이나 for 문을 접한다. 가능한 피해야겠다. 피하지 못할 경우는 빈 블록을 올바로 들여 쓰고 괄호로 감싼다.

## 5.4 팀 규칙

모든 프로그래머는 자신이 선호하는 규칙이 있다. 하지만 팀에 속한다면 자신이 선호해야 할 규칙은 바로 팀 규칙이다.

팀은 한 가지 규칙에 합의해야 한다. 그리고 모든 팀원은 그 규칙을 따라야 한다. 그래야 소프트웨어가 일관적인 스타일을 보인다.

좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이뤄진다는 사실을 기억하기 바란다. 스타일은 일관적이고 매끄러워야 한다. 한 소스 파일에서 보았던 형식이 다른 소스 파일에도 쓰이리라는 신뢰감을 독자에게 줘야 한다. 온갖 스타일을 뒤섞어 소스 코드를 필요 이상으로 복잡하게 만드는 실수는 반드시 피한다.

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 케이앤피북스(2010), P126-143.
>