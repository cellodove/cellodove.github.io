---
title : "[Android] Multi Module로 Android project 구성하기"
excerpt: "[Android] Multi Module로 Android project 구성하기"

categories :
  - Android

toc: true
toc_sticky: true
last_modified_at: 2022-06-05
---

![multi_module_image1.jpg](/assets/images/multi_module_image1.jpg?raw=true)

프로젝트는 언제나 어디서나 누구를 통해서나 항상 깔끔하고 구조화 되어있어야한다.

그렇다면 어떻게 해야 항상 의존성 규칙을 지키면서 구조화된 기능을 구현할 수 있을까? 클린 아키텍처와 멀티 모듈을 사용하면 모든 사람들이 규칙을 이해할수 있도록(반 강제) 구조를 표현할 수 있다.

## 모듈이란?

모듈은 안드로이드 공식 사이트에서 다음과 같이 정의 되어있다.

모듈은 소스 파일 및 빌드 설정으로 구성된 모음이며, 이를 통해 프로젝트를 별개의 기능 단위로 분할할 수 있다. 프로젝트에는 하나 이상의 모듈이 포함될 수 있으며, 하나의 모듈이 다른 모듈을 종속 항목으로 사용할 수 있다. 각 모듈을 독립적으로 빌드, 테스트, 디버그할 수 있다.

![multi_module_image2.png](/assets/images/multi_module_image2.png?raw=true)

안드로이드 스튜디오에서 새로운 프로젝트를 만들면 생기는 app도 모듈의 한 종류이다.

모듈을 추가하는 방법은 간단하다. 안드로이드 스튜디오에서 [File > New > New Module]을 누른 뒤, 원하는 모듈을 선택하면 된다.

![multi_module_image3.png](/assets/images/multi_module_image3.png?raw=true)

모듈의 종류는 여러가지가 있다. 그중 클린 아키텍처로 쓰이는 모듈 3가지가 있다.

### Application

안드로이드 프로젝트를 만들 때 기본으로 생성되는 app 모듈은 Application 모듈이다. 빌드의 결과로 APK 파일을 생성한다. 앱을 실행하기 위해선 Application 모듈이 반드시 필요하다.

### Android Library

안드로이드 프로젝트에서 지원되는 모든 파일 형식을 포함할 수 있다. 다른 Application 모듈의 종속 항목으로 추가할 수 있다. 빌드의 결과로는 AAR 파일이 생성된다.

### Java or Kotlin Library

이름 그대로 순수한 Java 혹은 Kotlin으로만 이루어진 모듈이다. 안드로이드 프레임워크로부터 독립적인 기능을 구현할 때 사용한다. 빌드의 결과로는 JAR 파일이 생성된다.

## 멀티 모듈 프로젝트란?

멀티 모듈 프로젝트란 말 그대로 여러 개의 모듈을 이용해 동작하는 프로젝트를 뜻한다.

보통 특정 기능을 하나의 모듈로 묶어서 모듈화를 진행한다. 그렇다면 멀티모듈을 할때의 장점은 무엇일까?

간단한 앱이라면 멀티 모듈로 프로젝트를 구성할 필요가 없다. 하지만 다수의 개발자가 참여하고 큰 사이즈의 프로젝트라면 코드도 많고 복잡한 구조를 가진다. 이때 프로젝트가 커지면서 여러가지 문제가 발생할 수 있다.

### 의존성

![multi_module_image4.png](/assets/images/multi_module_image4.png?raw=true)

위 그림은 MVVM패턴을 보여준다. 프로젝트를 진행하며 위의 사진과 같이 MVVM 패턴에 맞게 의존성 규칙을 정했다고 가정해보자. ViewModel에서는 Repository를 통해 데이터를 가져오고, Repository에서는 데이터베이스에 접근해서 데이터를 가져온다는 등의 규칙이다.

의존성 규칙을 잘 정했다고 하더라도 app 모듈 안에서 모든 기능을 구현하는 모놀리틱 프로젝트에서는 규칙을 위반하는 실수가 나오기 쉽다. 예를 들면, Activity에서 데이터베이스 인스턴스에 접근하여 데이터를 가져올 수 있다. 이는 개발자의 실수이긴 하나, 하나의 모듈로 프로젝트가 구성되어 있기 때문에 모든 코드에 접근할 수 있어서 생기는 문제이다.

관심사를 분리하기 위해선 의존성 규칙을 잘 지켜주어야 한다. 멀티 모듈 프로젝트에서는 build.gradle 파일에 사용할 모듈의 의존성을 직접 추가해줌으로써 사용하지 않는 모듈들은 접근조차 할 수 없게 된다.

### 빌드 속도

프로젝트가 커지면 빌드속도도 같이 증가한다. 하지만 멀티 모듈 구조로 프로젝트를 구정하면 빌드 속도를 단축 시킬 수 있다.

![multi_module_image5.jpeg](/assets/images/multi_module_image5.jpeg?raw=true)

프로젝트를 빌드할 때 변경된 모듈만 빌드하기 때문에 모듈이 많을수록 빌드 시간이 단축된다.

### 코드 재사용성

예를 들어서 이전에 만든 CustomView를 모듈 A와 모듈B에 추가한다고 생각보자

프로젝트에 추가 할 때, 만든 CustomView Class 파일을 그대로 Copy & Paste 하여 붙여서 사용할 것이다.

![multi_module_image6.png](/assets/images/multi_module_image6.png?raw=true)

만약 Module A가 동작하는 동안에 Module A에서 사용하고 있는 CustemView에 오류가 생긴다면 어떻게 될까?

CustomView를 그저 복사 붙여 넣기 하여 사용한 것이기 때문에 Module A에서 사용하는 CustomView의 오류를 해결해야 할 뿐더러 Module B에서 사용하는 CustomView 클래스 마저도 각각 수정을 해줄 수 밖에 없다.

바로 이러한 이유로 코드의 재사용성을 높이기 위해 코드를 모듈화 하여 종속시키는 작업이 필요하다.

![multi_module_image7.png](/assets/images/multi_module_image7.png?raw=true)

다음과 같이 Library로서 모듈화를 하여 각각 Module A 와 Module B에 종속시킴으로서 하나의 라이브러리로서

Module A와 Module B에 나눠서 작업하지 않아도 된다는 장점이 있다.

멀티 모듈로 프로젝트를 구성한다고 해서 장점만 있는 것은 아니다. 처음 멀티 모듈을 구성할 때는 다소 어색하고 어려울 수 있다. 또한 클래스들과 디렉토리가 굉장히 많아진다. 아래의 사진은 4개의 모듈로 구성된 프로젝트의 build.gradle 파일들이다. 관리해야 하는 파일들이 많아지고, 특정 파일을 찾을 때 디렉토리 사이에서 방황할 수 있다는 단점이 있다.

![multi_module_image8.png](/assets/images/multi_module_image8.png?raw=true)

## 마치며

처음에는 어떻게 구조를 잡아야할지, 의존성은 어떻게 넣어야할지 어려우나 관심사와 의존성 문제를 해결할수있는 좋은 방안인것 같다.

## 참조

> [https://developer.android.com/studio/projects?hl=ko](https://developer.android.com/studio/projects?hl=ko)
>
> [https://leveloper.tistory.com/201](https://leveloper.tistory.com/201)
>
> [https://www.youtube.com/watch?v=H4qh0n9Zu5k](https://www.youtube.com/watch?v=H4qh0n9Zu5k)
>
> [https://www.freecodecamp.org/news/how-modularisation-affects-build-time-of-an-android-application-43a984ce9968](https://www.freecodecamp.org/news/how-modularisation-affects-build-time-of-an-android-application-43a984ce9968)
>
> [https://footcode.postype.com/post/3673100](https://footcode.postype.com/post/3673100)
>