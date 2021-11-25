---
title : "[Android] Context 컨텍스트"
excerpt: "[Android] Context 컨텍스트"

categories :
  - Components

toc: true
toc_sticky: true
last_modified_at: 2021-11-25 
---

![stones.jpg](/assets/images/stones.jpg?raw=true)

## context란?

Context는 어플리케이션 환경에 관한 전체 정보를 받을 수 있는 추상 클래스이다.

Activity, Service등의 컴포넌트들 뿐만아니라 RecyclerView와 같은 화면 요소를 사용하기 위해서는 context가 필요하다.

Context는 시스템을 사용하기 위한 정보(프로퍼티)와 도구(메소드)가 담여 있는 클래스이다. 그렇기에 어플리케이션과 관련된 정보에 접근하거나 어플리케이션과 연관된 시스템 레벨의 함수를 호출할때 Context가 필요하다.

대부분의 컨텍스트는 컴포넌트 실행시 함께 생성되고, 생성된 컴포넌트가 가지고 있는 메소드를 호출하여 각각의 도구들을 사용할 수 있다.

## ContextWrapper와 ContextImpl

Context는 추상 클래스이기 때문에 이를 사용하기 위해서는 구현체가 있어야 한다. 기본 구현체는 ContextImpl이고, 이 구현체는 사용자에게 구출하지 않고 ContextWrapper로 감싸져 있다.

- Context의 구조

![contextwrapper.png](/assets/images/contextwrapper.png?raw=true)

ContextWrapper클래스는 생성자로 Context구현체를 받아서 Context동작을 Context 구현체의 동작으로 위임한다.

즉, ContextWrapper를 상속하는 Activity, Service, Application과 같은 컴포넌트에서 ContextWrapper의 함수를 호출하면 Context구현체의 동작을 수행하게 하는 것이다.

이를 Prox 패던이라고 하는데, 이 패턴을 사용하면 Context 구현체과 이를 사용하는 쪽의 결합도가 낮아져 쉽게 Context 구현체를 변경할 수 있다는 장점이 있다.

이를 공식 문서에서는 원본 Context의 변경없이 Context 동작을 수정할 수 있다고 설명하고 있다.

- 컨텍스트 클래스 다이어그램

![contextDiagram.png](/assets/images/contextDiagram.png?raw=true)

- 사용 예

![contextUse.png](/assets/images/contextUse.png?raw=true)

안드로이드에서 컨텍스트는 앱을 실행하기 위해 잘 짜여진 설계도의 개념으로 앱에서 사용하는 기본 기능이 담겨 있는 기본 클래스이다.

예를 들어 액티비티는 컨텍스트를 상속받아 구현된다. 액티비티처럼 컨텍스트를 상속받은 컴포넌트들은 코드상에서 BaseContext를 호출하는 것만으로 안드로이드의 기본 기능을 사용할 수 있다.

액티비티 안에서 startActivity()메서드를 통해 다른 액티비티를 호출할 수 있는 것도 모든 액티비티가 startActivity()가 설계되어 있는 컨텍스트를 상속받아서 구현되어 있기 때문이다.

## 컨텍스트의 종류

컨텍스트는 어플리케이션 컨텍스트와 베이스 컨텍스트가 있다.

- 어플리케이션 컨텍스트(Application Context)

어플리케이션 컨텍스트는 어플리케이션과 관련된 핵심 기능을 담고 있는 클래스이다.

앱을 통틀어서 하나의 인스턴스만 생성된다. 액티비티나 서비스 같은 컴포넌트에서 applicationContext를 직접 호출해서 사용할 수 있는데 호출하는 지점과 관계없이 모두 동일한 컨텍스트가 호출된다.

- 베이스 컨텍스트(Base Context)

베이스 컨텍스트는 안드로이드의 4대 메이저 컴포넌트인 액티비티, 서비스, 컨텐트 프로바이더, 브로드캐스트리시버의 기반 클래스이다.

각각의 컴포넌트에서 baseContext또는 this로 컨텍스트를 사용할 수 있고 컴포넌트의 개수만큼 컨텍스트도 함께 생성되기 때문에 호출되는 지점에 따라 서로 다른 컨텍스트가 호출된다.

## Context Type

- **Application**

싱글톤으로 어플리케이션 객체가 생성될 때 함께 Application Context가 생성되기 때문에 Application Context는 동일한 앱 안에서 항상 동일한 인스턴스를 반환한다.

- **Activity, Service**

액티비티나 서비스가 생성될 때마다 각자의 Context 인스턴스가 생성된다.

- **Broadcast Receiver**

자기 자신이 Context는 아니다. 리시버가 브로드캐스트 처리를 할 때마다 Context를 onReceive()의 인자로 전달 받아서 사용한다. 전달 받은 Context의 생명주기를 따르기 때문에 액티비티 컨텍스트로 브로드케스트 실행 시 액티비티가 종료되면 브로드캐스트도 함께 종료된다.

- **Content Provider**

자기 자신이 Context는 아니다. 동일한 응용 프로그램에 대해 호출 시 동일한 싱글톤 Context객체를 반환하고, 서로 다른 응용프로그램에 대해 호출 시 다른 Context 객체를 반환한다.

## 컴포넌트별 컨텍스트의 기능

아래표는 각 컴포넌트의 컨텍스트에서 지원하는 기능이다.

표에서처럼 화면과 관련된 기능은 액티비티의 컨텍스트에서만 제공하고 있다. 화면에 다이얼로그 창을 띄운다든지, 리소스 파일을 화면에 그리는 것은 액티비티만 할 수 있다.

![contextChart.png](/assets/images/contextChart.png?raw=true)

Context의 생명주기가 다르기 때문에 Activity와 분리된 작업에 Activity Context를 사용하면 예상치 못한 동작이 발생할 수 있다.

반대로 Activity에 종속된 작업을 Application Context를 이용하면 Activity가 종료되어도 Context를 이용한 작업이 종료되지 않아 memory leak이 발생할 수도 있다.

그렇기 때문에 Context를 사용할 때 각 Context의 특징과 Context가 필요한 작업의 특성을 고려해 Context를 알맞게 선택해 사용해야 한다.

## 참조

> 고돈호, 『이것이 안드로이드다 with 코틀린』, 한빛미디어(주)(2021), P301-302.
>
> [https://s2choco.tistory.com/10](https://s2choco.tistory.com/10)
>