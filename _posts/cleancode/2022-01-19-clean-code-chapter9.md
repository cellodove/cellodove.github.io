---
title : "[클린 코드] 챕터9 단위 테스트"
excerpt: "클린코드 정주행12"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2022-01-19
---


![verification.jpg](/assets/images/verification.jpg?raw=true)

옛날에는 단위 테스트란 자기 프로그램이 ‘돌아간다’는 사실만 확인하는 일회성 코드에 불과했다.

지금이라면 꼬치꼬치 따지며 코드가 제대로 도는지 확인하는 테스트 코드를 작성했으리라.

## TDD 법칙 세가지

TDD(테스트 주도 개발)는 실제 코드를 짜기 전에 단위 테스트부터 짜라고 요구한다.

다음 세 가지 법칙을 살펴보자.

- 첫째 법칙

실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.

- 둘째 법칙

컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.

- 셋째 법칙

현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

위 세가지 규칙을 따르면 테스트 코드와 실제 코드와 함께 나온다.

이렇게 일하면 실제 코드를 사실상 전부 테스트하는 테스트 케이스가 나온다.

하지만 실제 코드와 맞먹을 정도로 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기

일회용 테스트 코드를 짜오다가 새삼스레 자동화된 단위 테스트 슈트를 짜기란 쉽지 않다.

테스트를 안 하느니 지저분한 테스트 코드라도 있는 편이 좋다고 생각할 수 도있다.

하지만 테스트 코드를 실제 코드와 동일한 품질 유지를 하지 못하면 오히려 테스트 코드가 없을때보다 더 못하다.

실제 코드가 진화하고 바뀔때마다 테스트 코드도 변경된다.

이런 상황에서 테스트 코드가 지저분하다면 실제 코드를 수정하는 시간보다 테스트 코드를 수정하는 시간이 더 길어진다.

규모가 커질수록 테스트 코드를 유지하고 보수하는 비용도 늘어난다. 그렇다고 테스트 코드를 안쓰면 프로그램의 결함도가 높아질 수 밖에 없다.

의도하지 않은 결함 수가 많아지면 개발자는 변경을 주저한다. 변경하면 득보다 해가 크다 생각해 더 이상 코드를 정리하지 않는다. 그러면서 코드가 망가지기 시작한다.

테스트 코드는 실제 코드 못지 않게 중요하다. 테스트 코드는 이류 시민이 아니다. 테스트 코드는 사고와 설계와 주의가 필요하다. 실제 코드 못지 않게 깨끗하게 짜야 한다.

### 테스트는 유연성, 유지보수성, 재사용성을 제공한다

테스트 코드를 깨끗하게 유지하지 않으면 결국 잃어버린다. 그리고 테스트 케이스가 없으면 실제 코드를 유연하게 만드는 버팀목도 사라진다.

**코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 단위 테스트이다.**

테스트 케이스가 없다면 모든 변경이 잠정적인 버그다. 테스트 케이스가 있으면 변경이 쉬워진다.

따라서 테스트 코드가 지저분하면 코드를 변경하는 능력이 떨어지며 코드 구조를 개선하는 능력도 떨어진다. 테스트 코드가 지저분할수록 실제 코드도 지저분해진다. 결국 테스트 코드를 잃어버리고 실제 코드도 망가진다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만들려면? 세 가지가 필요하다.

1. 가독성
2. 가독성
3. 가독성

어쩌면 가독성은 실제 코드보다 테스트 코드에 더더욱 중요하다. 가독성을 높이려면 여느 코드와 마찬가지로 명로성, 단순성, 풍부한 표현력이 필요하다.

테스트 구조에따라 BUILD-OPERATE-CHECK 패턴이 유용할때가 있다.

각 테스트는 명확히 세부분으로 나눠진다. 첫 부분은 테스트 자료를 만든다. 두 번째 부분은 테스트 자료를 조작하며, 세 번째 부분은 조작한 결과가 올바른지 확인한다.

다음 예를 보자

```java
public void testGetPageHierarchyAsXml() throws Exception{
    makePages("PageOne", "PageTwo", "PageThree");

    submitRequet("Root", "Type:Pages");

    assertResponseContains("<name> PageOne </name>", "<name> PageTwo </name>","<name> ChildOne </name>"); 
}
```

잡다하고 세세한 코드를 거의 다 없앴다는 사실에 주목한다. 테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다.

### 도메인에 특화된 테스트 언어

위에 있는 코드는 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다.

흔히 쓰는 시스템 조작 API를 사용하는 대신 API 위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용하므로 테스트 코드를 짜기도 읽기도 쉬워진다.

이렇게 구현한 함수와 유틸리티는 테스트 코드에서 사용하는 특수 API가 된다. 즉, 테스트를 구현하는 당사자와 나중에 테스트를 읽어볼 독자를 도와주는 테스트 언어이다.

### 이중 표준

테스트 API코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다.

단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다. 실제 환경이 아니라 테스트 환경에서 돌아가는 코드이기 때문이다. 실제 환경과 테스트 환경은 요구사항이 판이하게 다르다.

다음 두 예제를 비교해본다.

```java
public void turnOnCoolerAndBlowerIfTooHot() throws Exception{
    hw.setTemp(WAY_TOO_HOT); 
    contoller.tic();
    assertTrue(hw.heaterState()); 
    assertTrue(hw.blowerState()); 
    assertTrue(hw.coolerState()); 
    assertTrue(hw.hiTempAlarm()); 
    assertTrue(hw.loTempAlarm()); 
}
```

```java
public void turnOnCoolerAndBlowerIfTooHot(){
    tooHot();
    assertEquals("HBchl", hw.getState());
}

public String getState(){
    String state = "";
    state += heater ? "H" : "h"; 
    state += blower ? "B" : "b"; 
    state += cooler ? "C" : "c";
    state += hiTempAlarm ? "H" : "h"; 
    state += loTempAlarm ? "L" : "l"; 
}
```

첫 번째 코드는 모든 assert문을 비교해서 봐야하고 tic() 메소드가 무슨일을 하는지도 모른다.

두 번째 코드는 tooHot() 이라는 메소드를 만들어서 가독성을 올렸다.

그리고 여러번의 assert문 대신에 추상적인 표현을 써서 한번에 표현했다.

실제 코드의 기준이라면 그릇된 정보를 피하라는 규칙의 위반에 가깝지만 테스트 코드와 실제 코드의 요구 사항은 다르다.

그리고 테스트 코드의 getState() 메소드의 경우에 StringBuffer 대신 String을 사용했다.

실제 성능을 중시한다면 StringBuffer를 사용해야 한다.

## 테스트 당 assert 하나

JUnit으로 테스트 코드를 짤 때는 함수마다 assert문 하나만 사용하면 좋다.

assert 문이 단 하나인 함수는 결론이 하나라서 코드를 이해하기 쉽고 빠르다.

하지만 불행하게도, 함수에서 하나의 assert문을 사용하기위해 테스트를 분리하면 중복되는 코드가 많아진다.

TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있다.

given/when/then 패턴 (준비, 실행, 검증)에서 given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스에 두면 된다.

아니면 완전히 독자적인 테스트 클래스를 만들어 @Before 함수에 given/when 부분을 넣고 @Test 함수에 then 부분을 넣어도 된다.

하지만 모두가 배보다 배꼽이 더 큰경우가있다. 이것저것 감안해 보면 asser문을 여럿 사용해서 사용하는편이 좋을 수 도있다.

‘단일 assert문’이라는 규칙이 훌륭한 지침이라 생각한다. 하지만 때로는 주저 없이 함수 하나에 여러 assert 문을 넣기도한다. 단지 assert문 개수는 최대한 줄여야 좋다는 생각이다.

### 테스트 당 개념 하나

어쩌면 “테스트 함수마다 한 개념만 테스트하라”는 규칙이 더 낫겠다.

이것저것 잡다한 개념을 연속으로 테스트하는 긴 함수는 피한다.

한 테스트 함수에서 assert문이 여럿이라는 사실이 문제가 아니라 여러 개념을 테스트한다는 사실이 문제다.

그러므로 가장 좋은 규칙은 **“개념 당 assert문 수를 최소로 줄여라”**와 **“테스트 함수 하나는 개념 하나만 테스트하라”**라 하겠다.

## F.I.R.S.T

깨끗한 테스트는 다음 다섯 가지 규칙을 따르는데, 각 규칙에서 첫 글자를 따오면 FIRST가 된다.

### 빠르게(Fast)

테스트는 빨라야 한다. 테스트는 빨리 돌아야 한다는 말이다.

테스트가 느리면 자주 돌릴 엄두를 못 낸다.

자주 돌리지 않으면 초반에 문제를 찾아내 고치지 못한다. 코드를 마음껏 정리하지도 못한다. 결국 코드 품질이 망가지기 시작한다.

### 독립적으로(Independent)

각 테스트는 서로 의존하면 안된다.

한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안 된다.

각 테스트는 독립적으로 그리고 어떤 순서로 실행해도 괜찮아야 한다.

테스트가 서로 의존하면 하나가 실패할때 나머지도 잇달아 실패하므로 원인을 진단하기 어려워지며 후반 테스트가 찾아내야 할 결함이 숨겨진다.

### 반복가능하게(Repeatable)

테스트는 어떤 환경에서도 반복 가능해야 한다.

실제 환경, QA환경, 집으로 가는길에 노트북의 환경에서도 실행할 수 있어야한다.

환경이 지원되지 않기에 테스트를 수행하지 못하는 상황에 직면할 수 도있다.

### 자가검증하는(Self-Validating)

테스트는 부울값으로 결과를 내야 한다. 성공 아니면 실패다.

통과 여부를 수작업으로 비교하게 만들면 안 된다.

테스트가 스스로 성공과 실패를 가늠하지 않는다면 판단은 주관적이 되며 지루한 수작업 평가가 필요하게 된다.

### 적시에(Timely)

테스트는 적시에 작성해야 한다. 단위 테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다.

실제 코드를 구현한 다음에 테스트 코드를 만들면 실제 코드가 테스트하기 어렵다는 사실을 발견할지도 모른다.

테스트가 불가능하도록 실제 코드를 설계할지도 모른다.

## 결론

이 장은 주제를 수박 겉핥기 정도로만 훑었다.

테스트 코드는 실제 코드만큼이나 프로젝트 건강에 중요하다.

테스트 코드는 실제 코드의 유연성, 유지보수성, 재사용성을 보존하고 강화하기 때문이다.

그러므로 테스트 코드는 지속적으로 깨끗하게 관리하자. 표현력을 높이고 간결하게 정리하자.

테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. 테스트 코드를 깨끗하게 유지하자.

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 인사이트(2021), P154-168.
>