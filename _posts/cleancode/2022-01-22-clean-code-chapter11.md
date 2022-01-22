---
title : "[클린 코드] 챕터11 시스템"
excerpt: "클린코드 정주행14"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2022-01-22
---


![buildings.jpg](/assets/images/buildings.jpg?raw=true)

“복잡성은 죽음이다. 개발자에게서 생기를 앗아가며, 제품을 계획하고 제작하고 테스트하기 어렵게 만든다”

## 도시를 세운다면?

우리가 도시를 세운다면? 온갖 세세한 사항을 혼자서 직접 관리할 수 있을까? 아마도 불가능하리라.

이미 세워진 도시라도 한 사람의 힘으로는 무리다.

그럼에도 불구하고 도시는 잘 돌아간다. 왜?

도시에는 수도 관리팀, 전력 관리팀, 교통 관리 팀, 치안 관리팀, 건축물 관리팀과 같이 각 분야를 관리하는 팀이 있듯, 큰 그림을 그리는 사람도 있으며, 작은 사항에 집중하는 사람들도 있다. 이를 통해 도시가 잘 돌아갈 수 있다.

도시는 적절한 추상화와 모듈화 덕에 큰 그림을 이해하지 못할지라도 개인과 개인이 관리하는 '구성요소'는 효율적으로 돌아간다.

깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다.

이 장에서는 높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법을 살펴본다.

## 시스템 제작과 시스템 사용을 분리하라

우선 제작은 사용과 아주 다르다는 사실을 명심한다.

**소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 ‘연결’하는)준비 과정과 (준비 과정 이후에 이어지는)런타임 로직을 분리해야 한다.**

맨 처음 살펴볼것은 관심사(concern)이다. 관심사 분리는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나다.

초기화 지연 (Lazy Initialization) 혹은 계산 지연 (Lazy Evaluation) 방식의 한계

```java
public Service getService() {
    if (service == null) {
        service = new MyServiceImpl(...); // 모든 상황에 적합한 기본값일까?
        return service;
    }
}
```

- 장점
  - 실제로 필요할 때까지 객체를 만들지 않으므로 불필요한 부하가 걸리지 않는다. 따라서 어플리케이션을 시작하는 시간이 그만큼 빨라진다.
  
  - 어떤 경우에도 null 포인터를 반환하지 않는다.

- 단점
  - getService 메서드가 MyServiceImpl에 명시적으로 의존하고 있기 때문에 런타임 로직에서 MyServiceImpl을 전혀 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 안된다.
  
  - MyServiceImpl이 무거운 객체라면 단위 테스트에서 getService 메서드를 호출하기 위해 호출하기 적절한 테스트 전용 객체(Test double이나 Mock object)를 service 필드에 할당해야 한다.
  
  - 일반 런타임 로직에 객체 생성 로직을 섞어놓아서 service가 null인 경로와 null이 아닌 경로 모두 테스트 해야 한다. 즉 메서드의 책임이 둘이기 때문에 단일책임원칙(SRP)를 깬다.

체계적이고 탄탄한 시스템을 만들고 싶다면 흔히 쓰는 좀스럽고 손쉬운 기법으로 모듈성을 깨서는 절대로 안 된다. 객체를 생성하거나 의존성을 연결할때도 마찬가지다.

설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.

또한 주요 의존성을 해소하기 위한 방식, 즉 전반적이며 일관적인 방식도 필요하다.

### Main 분리

- 시스템 생성과 시스템 사용을 분리하는 한 가지 방법
- 생성과 관련한 코드는 모두 main이나 main이 호출하는 모듈로 옮기고, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.

![cl_chap_11_1.png](/assets/images/cl_chap_11_1.png?raw=true)

main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다.

애플리케이션은 그저 객체를 사용할 뿐이다.

애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다. 단지 모든 객체가 적절히 생성되었다고 가정한다.

### 팩토리

- 때로는 객체가 생성되는 시점을 애플리케이션이 결정할 필요도 생긴다.
- 예를 들어, 주문 처리 시스템에서 어플리케이션은 LineItem 인스턴스를 생성해 Order에 넘긴다. 이때는 Abstract Factory 패턴을 사용한다. 그러면 LineItem을 생성하는 시점은 어플리케이션이 결정하지만 LineItem을 생성하는 코드는 어플리케이션이 모른다.

![cl_chap_11_2.png](/assets/images/cl_chap_11_2.png?raw=true)

여기서도 마찬가지로 모든 의존성이 main에서 OrderProcessing 어플리케이션으로 향한다.

즉, OrderProcessing 어플리케이션은 LineItem이 생성되는 구체적인 방법은 모른다.

그 방법은 LineItemFactoryImplementation이 안다.

그럼에도 OrderProcessing 어플리케이션은 LineItem 인스턴스가 생성되는 시점을 완벽하게 통제하며, 필요하다면 OrderProcessing 어플리케이션에서 사용하는 생성자 인수도 넘길 수 있다.

### 의존성 주입

사용과 제작을 분리하는 강력한 메커니즘 하나가 의존성 주입(Dependency Injection)이다.

의존성 주입은 제어 역전 (Inversion of Control) 기법을 의존성 관리에 적용한 메커니즘이다.

제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다. 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙 (SRP)을 지키게 된다.

의존성 관리 맥락에서는 객체는 의존성 자체를 인스턴스로 만드는 책임을 지지 않는다. 대신에 이런 책임을 다른 '전담' 메커니즘에 넘겨야 한다. 그렇게 함으로써 제어를 역전한다.

초기 설정은 시스템 전체에서 필요하므로 대개 '책임질' 메커니즘으로 'main' 루틴이나 특수 컨테이너를 사용한다.

## 확장

군락은 마을로, 마을은 도시로 성장한다. 처음에는 좁거나 사실상 없던 길이 포장되며 점차 넓어진다. 작은 건물과 공터는 큰 건물로 채워지고 결국 곳곳에 고층 건물이 들어선다.

‘처음부터 올바르게’ 시스템을 만들 수 있다는 믿음은 미신이다.대신에 우리는 오늘 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다. 내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다.

이것이 반복적이고 점진적인 애자일 방식의 핵심이다.

테스트 주도 개발(TDD), 리팩토링, 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.

소프트웨어 시스템은 물리적인 시스템과 다른다. 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.

### 횡단(cross-cutting) 관심사

원론적으로 모듈화되고 캡슐화된 방식으로 영속성 방식을 구상할 수 있다. 하지만 현실적으로는 영속성 방식을 구현한 코드가 온갖 객체로 흩어진다.

여기서 횡단 관심사라는 용어가 나온다. 영속성 프레임워크 또한 모듈화할 수 있다. 도메인 논리도 (독자적으로) 모듈화할 수 있다. 문제는 이 두 영역이 세밀한 단위로 겹친다는 점이다.

AOP에서 관점이라는 모듈 구성 개념은 “특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다”라고 명시한다.

영속성을 예로 들면, 프로그래머는 영속적으로 저장할 객체와 속성을 선언한 후 영속성 책임을 영속성 프레임워크에 위임한다. 그러면 AOP 프레임워크는 대상 코드에 영향을 미치지 않는 상태로 동작 방식을 변경한다.

<aside>
💡 영속성  

컴퓨터 공학에서 영속성이란, 한 객체가 자신을 생성한 작업이 종료되었음에도 불구하고 지속적으로 존재하는 상태를 말한다.

</aside>

<aside>
💡 AOP  

관점 지향 프로그래밍 (Aspect Oriented Programming)

</aside>

자바에서 사용하는 관점 혹은 관점과 유사한 메커니즘 세 개를 살펴보자

## 자바 프록시

자바 프록시는 단순한 상황에 적합하다. 개별 객체나 클래스에서 메서드 호출을 감싸는 경우가 좋은 예다. 하지만 JDK에서 제공하는 동적 프록시는 인터페이스만 지원한다. 클래스 프록시를 사용하려면 바이트 코드 처리 라이브러리가 필요하다.

- 자바 프록시 예제

```java
import java.utils.*;

// 은행 추상화 
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}
```

```java
// BackInpl.java
import java.utils.*;

// 추상화를 위한 POJO("Plain Old Java Object") 구현
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
       return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}
```

```java
// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// 프록시 API가 필요한 InvocationHandler 
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;

    public BankHandler (Bank bank) {
        this.bank = bank;
    }

    // InvocationHandler에 정의된 메서드
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());

            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());

            return null;
        } else {
            ...
        }
    }

    // 세부사항은 여기에 이어진다.
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// 다른 곳에 위치하는 코드
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```

- 프록시로 감쌀 인터페이스 Bank와 비즈니스 논리를 구현하는 POJO BankImpl을 정의했다.
- 코드의 양과 크기가 크다는 것이 프록시의 두 가지 단점이다.
- 프록시는 시스템 단위로 실행 '지점'을 명시하는 메커니즘도 제공하지 않는다.

## 순수 자바 AOP

다행스럽게도 대부분의 프록시 코드는 판박이라 도구로 자동화할 수 있다.

순수 자바 관점을 구현하는 Spring AOP 등과 같은 여러 자바 프레임워크는 내부적으로 프록시를 사용한다.

Spring은 비즈니스 논리를 POJO로 구현했다.

POJO는 순수하게 도메인에 초점을 맞추어 다른 프레임워크에 의존하지 않아 테스트하기 쉽고 간단하다.

```java
import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;

    @Embeddable
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }

    @Embedded
    private Address address;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
    private Collection<Account> accounts = new ArrayList<Account>();
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```

![cl_chap_11_3.png](/assets/images/cl_chap_11_3.png?raw=true)

클라이언트에서 bank의 getAccount를 호출한다고 믿지만 실제로는 Bank POJO의 기본 동작을 확장한 중첩 DECORATOR 객체 집합의 가장 외곽과 통신한다.

## AspectJ 관점

- AspectJ 언어

언어 차원에서 관점을 모듈화 구성으로 지원하는 자바 언어 확장

- 관심사를 관점으로 분리하는 가장 강력한 도구 집합 제공한다 하지만 새 도구를 사용하고 새 언어 문법과 사용법을 익혀야 한다.
- AspectJ Annotation ( @Aspect, @Before 등) - 쉽게 접근 가능하게 한다.

## 테스트 주도 시스템 아기텍처 구축

애플리케이션 도메인 논리를 POJO로 작성한다면, 즉 코드 수준에서 아키텍쳐와 분리가 가능하다면 진정한 테스트 주도 아키텍처 구축이 가능하다.

필요에 따라 새로운 기술을 채택해 단순한 아키텍처를 복잡한 아키텍쳐로 키울 수 있다.

소프트웨어 구조가 관점을 효과적으로 분리한다면 확장이 가능하다.

최선의 시스템 구조는 각기 POJO (또는 다른) 객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다.

이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

## 의사 결정을 최적화하라

모듈을 나누고 관심사를 분리하면 지렵적인 관리와 결정이 가능해진다.

아주 큰 시스템에서는 한 사람이 모든 결정을 내리기 어렵다.

가장 적합한 사람에게 책임을 맡기면 가장 좋다. 그리고 가장 마지막 순간까지 결정을 미루어 최대한 정보를 모아 최선의 결정을 내리는 것은 때때로 좋은 선택일 수 있다.

관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 캡슐화 하기 쉽고, 컴포넌트를 엮기 쉽다.

하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.

표준은 확실한 이득을 가져올 경우 사용하라

## 시스템은 도메인 특화 언어가 필요하다

DSL은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다. DSL로 짠 코드는 도메인 전무가가 작성한 구조적인 산문처럼 읽힌다.

좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 ‘의사소통 간극’을 줄여준다.

효과적으로 사용한다면 DSL은 추상화 수준을 코드 관용구나 디자인 패턴 이상으로 끌어올린다. 그래서 개발자가 적절한 추상화 수준에서 코드 의도를 표현할 수 있다.

도메인 특화 언어 (DSL)을 사용하면 고차원 정책에서 저차원 세부사항에 이르기까지 모든 추상화 수준과 모든 도메인을 POJO로 표현할 수 있다.

## 결론

시스템 역시 깨끗해야 한다.

깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어트린다.

도메인 논리가 흐려지면 제품 품질이 떨어진다. 버그가 숨어들기 쉬워지고, 스토리를 구현하기 어려워지는 탓이다.

기민성이 떨어지면 생산성이 낮아져 TDD가 제공하는 장점이 사라진다.

모든 추상화 단계에서 의도는 명확히 표현해야 한다. 그러려면 POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사르 분리해야 한다.

시스템을 설계하든 개별 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용해야 한다는 사실을 명심하자.

## 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 인사이트(2021), P194-213.
>