---
title : "[클린 코드] 챕터13 동시성1"
excerpt: "클린코드 정주행16"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2022-01-26
---

![singapore.jpg](/assets/images/singapore.jpg?raw=true)

객체는 처리의 추상화다. 스레드는 일정의 추상화다.

동시정과 깔끔한 코드는 양립하기 어렵다.

스레드를 하나만 실행하는 코드는 짜기가 쉽다.

겉으로 보기에는 멀쩡하나 깊숙한 곳에 문제가 있는 다중 스레드 코드도 짜기 쉽다.

이런 코드는 시스템이 부하를 받기 전까지 멀쩡하게 돌아간다.

- 이 장에서는 어러 스레드를 동시에 돌리는 이유를 논한다.
- 여러 스레드를 동시에 돌리는 어려움도 논한다.
- 동시성을 테스트하는 방법과 문제점을 논한다.

## 동시성이 필요한 이유?

동시성은 결합(coupling)을 없애는 전략이다.

즉, 무엇(what)과 언제(when)을 분리하는 전략이다.

스레드가 하나인 프로그램은 무엇과 언제가 서로 밀접하다. 그래서 호출 스택을 살펴보면 프로그램 상태가 곧바로 드러난다.

흔히 단일 스레드 프로그램을 디버깅하는 프로그래머는 정지점(breakpoint)을 정한 후 어느 정지점에 걸렸는지 살펴보면서 시스템 상태를 파악한다.

무엇과 언제를 분리하면 애플리케이션 구조와 효율이 극적으로 나아진다.

구조적인 관점에서 프로그램은 거대한 루프 하나가 아니라 작은 협력 프로그램 여럿으로 보인다.

따라서 시스템을 이해하기가 쉽고 문제를 분리하기도 쉽다.

- 예) 서블릿(Servlet)모델

서블릿은 웹 혹은 EJB 컨테이너(Enterprise JavaBeans)라는 우산 아래서 돌아가는데 이들 컨테이너는 동시성을 부분적으로 관리한다.

웹 요청이 들어올 때마다 웹 서버는 비동기식으로 서블릿을 실행한다.

서블릿 프로그래머는 들어오는 모든 웹 요청을 관리할 필요가 없다.

원칙적으로 각 서블릿 스레드는 다른 서블릿 스레드와 무관하게 자신만의 세상에서 돌아간다.

하지만 웹 컨테이너가 갖는 결함 분리 전략은 간단하지 않다. 그렇더라도 큰 구조적 이점 갖는다.

구조적 개선만을 위해 동시성을 채택하는 건 아니다.

어떤 시스템은 응답 시간과 작업 처리량 (throughput) 개선이라는 요구사항으로 인해 직접적인 동시성 구현이 불가피하다.

- 예) 매일 수많은 웹 사이트에서 정보를 가져와 요약하는 정보 수집기

수집기가 단일 스레드 프로그램이라면 한 번에 한 웹 사이트를 방문해 정보를 가져오고, 이 과정에서 한 사이트를 끝내야 다음 사이트로 넘어간다.

웹 사이트를 계속 추가하면 정보 수집하는 시간도 늘어나고, 매일 실행하므로 24시간 안에 끝나야 하지만 24시간을 넘기게 된다.

단일 스레드 수집기는 웹 소켓에서 입출력을 기다리는 시간이 아주 많다. 다중 스레드 알고리즘을 사용하면 수집기 성능을 높일 수 있다.

- 예) 한 번에 한 사용자를 처리하는 시스템: 한 사용자를 처리하는 시간은 1초

사용자가 소수라면 시스템이 아주 빨리 반응하지만, 사용자 수가 늘어날수록 시스템이 응답하는 속도도 늦어진다.

150명 뒤에 줄 서려는 사용자는 없다. 대신 많은 사용자를 동시에 처리하면 시스템 응답 시간을 높일 수 있다.

- 예) 정보를 대량으로 분석하는 시스템

모든 정보를 처리한 후에야 최종적인 답을 낼 수 있다. 정보를 나눠 여러 컴퓨터에서 돌리면 어떨까? 대량의 정보를 병렬로 처리한다면?

### 미신과 오해

동시성은 어렵다. 각별히 주의하지 않으면 난감한 상황에 처한다.

다음은 동시성과 관련한 일반적인 미신과 오해다.

- 동시성은 항상 성능을 높여준다.

동시성은 ‘때로’ 성능을 높여준다.

대기 시간이 아주 길어 여러 스레드가 프로세서를 공유할 수 있거나, 여러 프로세서가 동시에 처리할 독립적인 계산이 충분히 많은 경우에만 성능이 높아진다.

어느 쪽도 일상적으로 발생하는 상황은 아니다.

- 동시성을 구현해도 설계는 변하지 않는다.

단일 스레드 시스템과 다중 스레드 시스템은 설계가 판이하게 다르다.

일반적으로 **무엇**과 **언제**를 분리하면 시스템 구조가 크게 달라진다.

- 웹 또는 EJB 컨테이너를 사용하면 동시성을 이해할 필요가 없다.

실제로는 컨테이너가 어떻게 동작하는지, 어떻게 동시 수정, 데드락 등과 같은 문제를 피할 수 있는지를 알아야만 한다.

반대로 다음은 동시성과 관련된 타당한 생각 몇 가지다.

- 동시성은 다소 부하를 유발한다. 성능 측면에서 부하가 걸리며, 코드도 더 짜야 한다.
- 동시성은 복잡하다. 간단한 문제라도 동시성은 복잡하다.
- 일반적으로 동시성 버그는 재현하기 어렵다. 그래서 진짜 결함으로 간주되지 않고 일회성 문제로 여겨 무시하기 쉽다.
- 동시성을 구현하라면 흔히 근본적인 설계 전략을 재고해야 한다.

## 난관

동시성을 구현하기가 어려운 이유는 무엇일까?

간단한 클래스를 살펴보자.

```java
public class X {
    private int lastIdUsed;

    public int getNextId() {
        return ++lastIdUsed;
    }
}
```

인스턴스 X를 생성하고, lastIdUsed 필드를 42로 설정한 다음, 두 스레드가 해당 인스턴스를 공유한다. 이제 두 스레드가 getNextId();를 호출한다고 가정하자. 결과는 셋 중 하나다.

- 한 스레드는 43을 받는다. 다른 스레드는 44를 받는다. lastIdUsed는 44가 된다.
- 한 스레드는 44를 받는다. 다른 스레드는 43을 받는다. lastIdUsed는 44가 된다.
- 한 스레드는 43을 받는다. 다른 스레드는 43을 받는다. lastIdUsed는 43이 된다.

두 스레드가 같은 변수를 동시에 참조하면 세 번째와 같은 놀라운 결과가 발생한다.

두 스레드가 자바 코드 한 줄을 거쳐가는 경로는 수없이 많은데, 그중에서 일부 경로가 잘못된 결과를 내놓기 때문이다.

바이트 코드만 고려했을 때 두 스레드가 getNextId 메서드를 실행하는 잠재적인 경로는 int일 경우 최대 12870개에 달한다. long일 경우는 조합 가능한 경로 수가 2,704,156개로 증가한다.

물론 대다수 경로는 올바른 결과를 내놓지만, 문제는 잘못된 결과를 내놓는 일부 경로다.

## 동시성 방어 원칙

지금부터 동시성 코드가 일으키는 문제로부터 시스템을 방어하는 원칙과 기술을 소개한다.

### 단일 책임 원칙(Single Responsibility Principle SRP)

SRP는 주어진 메서드/클래스/컴포넌트를 변결할 이유가 하나여야 한다는 원칙이다.

동시성은 복잡성 하나만으로도 따로 분리할 이유가 충분하다.

즉, 동시성 관련 코드는 다른 코드와 분리해야 한다는 뜻이다.

동시성을 구현할 때는 다음 몇 가지를 고려한다.

- 동시성 코드는 독자적인 개발, 변경, 조율 주기가 있다.
- 동시성 코드에는 독자적인 난관이 있다. 다른 코드에서 겪는 난관과 다르며 훨씬 어렵다.
- 잘못 구현한 동시성 코드는 별의별 방식으로 실패한다. 주변에 있는 다른 코드가 발목을 잡지 않더라도 동시성 하나만으로도 충분히 어렵다.

💡 **권장사항: 동시성 코드는 다른 코드와 분리하라.**

### 따름 정리(corollary): 자료 범위를 제한하라

객체 하나를 공유한 후 동일 필드를 수정하던 두 스레드가 서로 간섭하므로 예상치 못한 결과를 내놓는다.

이런 문제를 해결하는 방안으로 공유 객체를 사용하는 코드 내 임계 영역(critical section)을 synchronized 키워드로 보호하라고 권장한다.

이런 임계영역의 수를 줄이는 기술이 중요하다.

공유 자료를 수정하는 위치가 많을수록 다음 가능성도 커진다.

- 보호할 임계영역을 빼먹는다. 그래서 공유 자료를 수정하는 모든 코드를 망가뜨린다.
- 모든 임계영역을 올바로 보호했는지(DRY 위반) 확인하느라 똑같은 노력과 수고를 반복한다.
- 그렇지 않아도 찾아내기 어려운 버그가 더욱 찾기 어려워진다.

💡 **권장사항: 자료를 캡슐화하라. 공유 자료를 최대한 줄여라.**

### 따름 정리: 자료 사본을 사용하라

공유 자료를 줄이려면 처음부터 공유하지 않는 방법이 제일 좋다.

어떤 경우에는 객체를 복사해 읽기 전용으로 사용하는 방법이 가능하다.

어떤 경우에는 각 스레드가 객체를 복사해 사용한 후 한 스레드가 해당 사본에서 결과를 가져오는 방법도 가능하다.

공유 객체를 피하는 방법이 있다면 코드가 문제를 일으킬 가능성도 아주 낮아진다.

객체의 사본으로 동기화를 피할 수 있다면 내부 잠금을 없애 절약한 수행 시간이 사본 생성과 가비지 컬렉션에 드는 부하를 상쇄할 가능성이 크다.

### 따름 정리: 스레드는 가능한 독립적으로 구현하라

자신만의 세상에 존재하는 스레드를 구현한다.

즉, 다른 스레드와 자료를 공유하지 않는다.

각 스레드는 클라이언트 요청 하나를 처리한다.

모든 정보는 비공유 출처에서 가져오며 로컬 변수에 저장한다.

그러면 각 스레드는 세상에 자신만 있는 듯이 돌아갈 수 있다. 다른 스레드와 동기화할 필요가 없으므로.

예를 들어 HttpServlet 클래스에서 파생한 클래스는 모든 정보를 doGet과 doPost 매개변수로 받는다.그래서 각 서블릿은 마치 자신이 독자적인 시스템에서 동작하는 양 요청을 처리한다.

서블릿 코드가 로컬 변수만 사용한다면 서블릿이 동기화 문제를 일으킬 가능성은 전무하다.

물론 서블릿을 사용하는 대다수 어플리케이션은 결국 데이터베이스 연결과 같은 자원을 공유하는 상황에 처한다.

💡 **권장사항: 독자적인 스레드로, 가능하면 다른 프로세서에서, 돌려도 괜찮도록 자료를 독립적인 단위로 분할하라.**

## 라이브러리를 이해하라

자바 5로 스레드 코드를 구현한다면 다음을 고려하기 바란다.

- 스레드 환경에 안전한 컬렉션을 사용한다. 자바 5부터 제공한다.
- 서로 무관한 작업을 수행할 때는 executor 프레임워크를 사용한다.
- 가능하다면 스레드가 차단(blocking)되지 않는 방법을 사용한다.
- 일부 클래스 라이브러리는 스레드에 안전하지 못하다.

### 스레드 환경에 안전한 컬렉션

`java.util.concurrent` 패키지에 있는 클래스

다중 스레드 환경에서 사용해도 안전하며, 성능도 좋다.

ConcurrentHashMap은 거의 모든 상황에서 HashMap보다 빠르다.

동시 읽기/쓰기를 지원하며, (보통 다중 스레드 환경에서 문제가 생기는) 자주 사용하는 복합 연산을 다중 스레드 상에서 안전하게 만든 메서드로 제공한다.

좀 더 복잡한 동시성 설계를 지원하고자 자바 5에는 다른 클래스도 추가되었다.

다음에 몇 가지를 소개한다.

| 클래스 | 설명 |
| --- | --- |
| ReentrantLock | 한 메서드에서 잠그고 다른 메서드에서 푸는 락(lock)이다. |
| Semaphore | 전형적인 세마포어다. 개수(count)가 있는 락이다. |
| CountDownLatch | 지정한 수만큼 이벤트가 발생하고 나서야 대기 중인 스레드를 모두 해제하는 락이다. 모든 스레드에게 동시에 공평하게 시작할 기회를 준다. |

💡 **권장사항: 언어가 제공하는 클래스를 검토하라. 자바에서는 java.util.concurrent, java.util.concurrent.atomic, java.util.concurrent.locks를 익혀라.**

## 실행 모델을 이해하라

다중 스레드 애플리케이션을 분리하는 방식은 여러 가지다.

구체적으로 논하기전에 먼저 몇 가지 기본 용어부터 이해하자.

| 용어 | 설명 |
| --- | --- |
| 한정된 자원(Bound Resource) | 다중 스레드 환경에서 사용하는 자원으로, 크기나 숫자가 제한적이다. 데이터베이스 연결, 길이가 일정한 읽기/쓰기 버퍼 등이 예다. |
| 상호 배제 (Mutual Exclusion) | 한 번에 한 스레드만 공유 자료나 공유 자원을 사용할 수 있는 경우를 가리킨다. |
| 기아(Starvation) | 한 스레드나 여러 스레드가 굉장히 오랫동안 혹은 영원히 자원을 기다린다. 예를 들어, 항상 짧은 스레드에게 우선순위를 준다면, 짧은 스레드가 지속적으로 이어질 경우, 긴 스레드가 기아 상태에 빠진다. |
| 데드락(Deadlock) | 여러 스레드가 서로 끝나기를 기다린다. 모든 스레드가 각기 필요한 자원을 다른 스레드가 점유하는 바람에 어느 쪽도 더 이상 진행하지 못한다. |
| 라이브락(Livelock) | 락을 거는 단계에서 각 스레드가 서로를 방해한다. 스레드는 계속해서 진행하려 하지만, 공명(resonance)으로 인해, 굉장히 오랫동안 혹은 영원히 진행하지 못한다. |

기본 개념을 이해했으니 이제 다중 스레드 프로그래밍에서 사용하는 실행 모델을 몇 가지 살펴보자.

### 생산자-소비자 (Producer - Consumer)

하나 이상 생산자 스레드가 정보를 생산해 버퍼(buffer)나 대기열(queue)에 넣는다.

하나 이상 소비자 스레드가 대기열에서 정보를 가져와 사용한다.

생산자 스레드와 소비자 스레드가 사용하는 대기열은 한정된 자원이다.

- 생산자 스레드는 대기열에 빈 공간이 있어야 정보를 채운다. 즉, 빈 공간이 생길 때까지 기다린다.
- 소비자 스레드는 대기열에 정보가 있어야 가져온다. 즉, 정보가 채워질 때까지 기다린다.
- 대기열을 올바로 사용하고자 생산자 스레드와 소비자 스레드는 서로에게 시그널을 보낸다.
- 생산자 스레드는 대기열에 정보를 채운 다음 소비자 스레드에게 "대기열에 정보가 있다"는 시그널을 보낸다.
- 소비자 시그널은 대기열에서 정보를 읽어 들인 후 "대기열에 빈 공간이 있다"는 시그널을 보낸다.
- 따라서 잘못하면 생산자 스레드와 소비자 스레드가 둘 다 진행 가능함에도 불구하고 동시에 서로에게 시그널을 기다릴 가능성이 존재한다.

### 읽기-쓰기 (Readers - Writer)

읽기 스레드를 위한 주된 정보원으로 공유 자원을 사용하지만, 쓰기 스레드가 이 공유 자원을 이따금 갱신한다고 하자. 이런 경우 처리율(throughput)이 문제의 핵심이다.

- 처리율을 강조하면 기아(starvation) 현상이 생기거나 오래된 정보가 쌓인다. 갱신을 허용하면 처리율에 영향을 미친다.
- 쓰기 스레드가 버퍼를 갱신하는 동안 읽기 스레드가 버퍼를 읽지 않으려면, 마찬가지로 읽기 스레드가 버퍼를 읽는 동안 쓰기 스레드가 버퍼를 갱신하지 않으려면 복잡한 균형 잡기가 필요하다.
- 대개는 쓰기 스레드가 버퍼를 오랫동안 점유하는 바람에 여러 읽기 스레드가 버퍼를 기다리느라 처리율이 떨어진다.
- 따라서 읽기 스레드의 요구와 쓰기 스레드의 요구를 적절히 만족시켜 처리율도 적당히 높이고 기아도 방지하는 해법이 필요하다.
- 간단한 전략은 읽기 스레드가 없을 때까지 갱신을 원하는 쓰기 스레드가 버퍼를 기다리는 방법이다. 하지만 읽기 스레드가 계속 이어진다면 쓰기 스레드는 기아 상태에 빠진다.
- 반면, 쓰기 스레드에게 우선권을 준 상태에서 쓰기 스레드가 계속 이어진다면 처리율이 떨어진다. 양쪽 균형을 잡으면서 동시에 갱신 문제를 피하는 방법이 필요하다.

### 식사하는 철학자들 (Dining Philosophers)

- 둥근 식탁에 철학자들이 둘러 앉고, 배가 고프면 양손에 포크를 집어 들고 스파게티를 먹어야 한다면? 왼쪽이나 오른쪽 철학자가 포크를 사용 중이라면 스파게티를 먹지 못한다.
- 여기서 철학자를 스레드로, 포크를 자원으로 바꾸어 생각해보면? 많은 기업 어플리케이션에서 겪는 문제다.
- 여러 프로세스가 자원을 얻으려 경쟁하기 때문에 주의해서 설계하지 않으면 데드락, 라이브락, 처리율 저하, 호율성 저하 등을 겪는다.

일상에서 접하는 대다수 다중 스레드 문제는 (형태가 조금씩 다를지라도) 위 세 범주 중 하나에 속한다.

각 알고리즘을 공부하고 해법을 직접 구현해보라. 그러면 나중에 실전 문제에 부닥쳤을 때 해결이 쉬워지리라.

💡 **권장사항: 위에서 설명한 기본 알고리즘과 각 해법을 이해하라**

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 인사이트(2021), P226-235.
>