---
title : "[Android] Intent 인텐트"
excerpt: "[Android] Intent 인텐트"

categories :
  - Elements

toc: true
toc_sticky: true
last_modified_at: 2022-01-09 
---

![Lighthouse.jpg](/assets/images/Lighthouse.jpg?raw=true)

Intent는 다른 컴포넌트에서 작업을 요청하는데 사용할 수 있는 메시징 개체이다. Intent는 여러 가지 방법으로 컴포넌트간의 통신을 용이하게 한다.

**Intent는 그대로 직역하면 '의도'라는 뜻이다. 개발자가 어떤 의도를 가지고 메서드를 실행할 것인지를 Intent에 담아서 안드로이드에 전달하면 안드로이드는 해당 Intent를 해석하고 실행한다.**

액티비티를 실행하기 위해서는 단순히 컨텍스트가 제공하는 메서드를 호출하면 되는데, 이때 실행할 액티비티가 명시된 Intent를 해당 메서드에 전달해야 한다.

## 사용

- Activity 시작

액티비티를 시작하려면 Intent를 startActivity()로 전달하면 된다. Intent는 시작할 액티비티를 설명하고 필요한 데이터를 담는다.

- Service 시작

서비스를 시작하려면 Intent를 startService()에 전달하면 된다. Intent는 시작할 서비스를 설명하고 필요한 데이터를 담는다.

- broadcast 전달

브로드캐스트는 모든 앱이 수신할 수 있는 메시지이다. Intent를 sendBroadcast() 또는 sendOrderedBroadcast()에 전달하면 다른 앱에 브로드캐스트를 전달할 수 있다.

## 타입

- **명시적 인텐트**

명시적 인텐트는 인텐트에 객체나 컴포넌트 이름을 지정하여 호출할 대상을 확실히 알 수 있는 경우에 사용한다. 주로 어플리케이션 내부에서 사용한다.

**특정 컴포넌트나 액티비티가 명확하게 실행되어야할 경우에 사용한다.**

- **암시적 인텐트**

인텐트의 액션과 데이터를 지정했지만, 호출할 대상이 달라질 수 있는 경우에 암시적 인텐트를 사용한다.

설치된 어플리케이션들의 정보를 알고 있는 안드로이드 시스템이 인텐트를 이용해 요청한 정보를 처리할 수 있는 적절한 컴포넌트를 찾아본 다음 사용자에게 그 대상과 처리 결과를 보여주는 과정을 거친다.

예를 들면 문서 편집기가 있다. PDF를 보기 위해 파일을 클릭하면 해당 안드로이드 폰에 PDF를 보여줄 수 있는 어플리케이션들이 반응을 한다. 그러면 안드로이드 시스템은 이 어플리케이션들중에서 어떤것을 선택할것인지 사용자가 선택할 수 있도록 창을 띄운다.

PDF를 보기위해 이미 여러 뷰어 앱이 존재하는데 리더앱을 만드는것은 시간과 비용상 좋지 않다.

**이미 기존에 어떤 기능들을 지원하는 앱들이 있는 경우 암시적 인텐트를 사용해 그 앱들을 사용하면 되는것이다.**

![intent.jpg](/assets/images/intent.jpg?raw=true)

## 동작 과정

![intentProsess.png](/assets/images/intentProsess.png?raw=true)

1. 실행할 대상의 액티비티 이름과 전달할 데이터를 담아서 인텐트를 생성한다.
2. 생성한 인텐트를 startActivity() 메서드에 담아서 호출하면 액티비티 매니저에 전달된다.
3. 액티비티 매니저는 인텐트를 분석해서 지정한 액티비티를 실행시킨다.
4. 전달된 인텐트는 최종 목적지인 타깃 액티비티까지 전달된다.
5. 타깃 액티비티에서는 전달받은 인텐트에 데이터가 있으면 Intent.extras 형식으로 사용할 수 있다.

## 참조

> 고돈호, 『이것이 안드로이드다 with 코틀린』, 한빛미디어(주)(2021), P303.
>
> [https://developer.android.com/guide/components/intents-filters?hl=ko](https://developer.android.com/guide/components/intents-filters?hl=ko)
>
> [https://limkydev.tistory.com/35](https://limkydev.tistory.com/35)
>