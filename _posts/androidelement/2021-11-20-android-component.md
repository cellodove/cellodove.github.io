---
title : "[Android] Component 컴포넌트"
excerpt: "[Android] Component 컴포넌트"

categories :
  - Elements

toc: true
toc_sticky: true
last_modified_at: 2021-11-20 
---


![Gear.jpg](/assets/images/Gear.jpg?raw=true)

## 컴포넌트

Component는 안드로이드 앱을 구성하는 요소이다.

안드로이드는 4개의 핵심 Component를 제공한다. Component는 독립적인 생명 주기에 의해서 실행된다.

- 액티비티(Activity)
  - 화면 UI를 담당하는 컴포넌트
- 브로드캐스트 리시버(Broadcast Receiver)
  - 시스템 또는 사용자가 발생하는 메시지를 수신하는 컴포넌트
- 서비스(Service)
  - 백그라운드 코드 처리를 담당하는 컴포넌트
- 컨텐트 프로바이더(Content Provider)
  - 앱끼리 데이터를 공유하기 위한 컴포넌트

## 특징

### 컴포넌트 여러 개를 조합하여 하나의 앱을 만든다

앱에서 컴포넌트의 물리적인 모습은 클래스이다. 하지만 앱에서 만든 모든 클래스가 컴포넌트는 아니다. 안드로이드의 클래스는 컴포넌트와 일반 클래스로 나뉘는데, 일반 클래스의 생명주기는 개발자 코드로 관리한다. 컴포넌트의 생명주기는 안드로이드 시스템이 관리한다.

### 컴포넌트는 앱 내에서 독립적인 실행 단위이다

컴포넌트 클래스는 독립적인 수행 단위로 동작하기 때문에 직접 개발자 코드로 생성해서 실행할 수 있다.

예를 들어  A클래스에서 B클래스를 실행하려면

```kotlin
var bClass = B()
```

위와 같은 구문으로 객체를 생성해서 실행한다. 하지만 만약 B클래스가 컴포넌트 클래스라면 인텐트(Intent)를 매개로 하여 결합하지 않은 상태에서 독립적으로 실행할 수 있다.

### main 함수 같은 어플리케이션의 진입 지점이 따로 없다

앱은 수행 시점이 다양하다. 컴포넌트 클래스들은 프로세스가 구동되었을때 최초로 실행되는 시점이 될 수 있다.

예를 들어 사용자가 직접 클릭하여 메시지 앱을 실행한적이 없더라도 스마트폰에 메시지가 전달된다면 메시지 수신기능을 하는 컴포넌트로부터 앱이 실행될 수 있다.

### 어플리케이션 라이브러리 개념이 있다

안드로이드는 앞서 말한것과 같이 컴포넌트 기반이기때문에 위에 말한 메시지가 전달되면 메시지 앱이 실행되는것처럼 다른 외부 앱의 컴포넌트도 실행할 수 있다. 개발자가 만들지 않았는데 그 앱의 기능처럼 느껴진다면, 그것은 라이브러리이다.

예를 들어 메신저앱에서 메시지를 보낼때 안드로이드 갤러리를 실행시켜 원하는 사진을 선택하여 보내거나, 카메라를 실행시켜 사진을찍어 사진을 보내는 것이다.

## 참조

> 강성윤, 『깡샘의 안드로이드 프로그래밍』, 루비페이퍼(2019)
>
> [https://hongbeomi.medium.com/component란-무엇인가-f92e708d8643](https://hongbeomi.medium.com/component%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-f92e708d8643)
>