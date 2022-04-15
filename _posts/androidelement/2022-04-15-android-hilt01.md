---
title : "[Android] Hilt를 사용한 DI(의존성 주입)1"
excerpt: "의존성 주입이란 무엇인가"

categories :
  - Elements

toc: true
toc_sticky: true
last_modified_at: 2022-04-15
---

![hillt_title_image1.jpg](/assets/images/hillt_title_image1.jpg?raw=true)

Hilt를 보기전에 먼저 의존성 주입에 대해 알아보자

## DI(의존성 주입이란)

외부에서 참조 객체를 생성하여 넘겨주는 것을 의미한다.

일반적인 객체 생성은 클래스 안에서 사용할 객체를 생성하지만 DI를 적용한 객체 생성은, 외부에서 생성된 객체를 주입 받는 방식이다.

프로그래밍에 널리 사용되는 기법으로, 안드로이드 개발에 적합하다.

DI를 구현하면 다음과 같은 이점을 누릴 수 있다.

- 코드 재사용 가능
- 리팩토링 편의성
- 테스트 편의성

## DI의 작동방식

### DI란?

위에서도 말했지만 클래스에는 흔히 다른 클래스 참조가 필요하다.

예를 들어 `Car`클래스는 `Engine`클래스 참조가 필요할 수 있다. 이처럼 필요한 클래스를 종속 항목이라고 하며, 이 예에서 `Car`클래스가 실행되기 위해서는 `Engine`클래스의 인스턴스가 있어야 한다.

클래스에서 필요한 객체를 얻는 세 가지 방법은 다음과 같다.

1. 클래스안에서 필요한 종속 항목을 구성한다. `Car`클래스는 안에`Engine`인스턴스를 생성하여 초기화한다.
2. 다른 곳에서 객체를 가져온다. `Context`getter 및 `getSystemService()`와 같은 일부 Android API는 이러한 방식으로 작동한다.(외부에서 생성해두고 그걸 가져와 쓴다.)
3. 객체를 매개변수로 제공받는다. 앱은 클래스가 구성될 때 이러한 종속 항목을 제공하거나 각 종속 항목이 필요한 함수에 전달할 수 있다. 위의 예에서 `Car`생성자는 `Engine`을 매개변수로 받는다.

여기서 3번째가 DI삽입이다. 이 접근 방법을 사용하면 클래스 인스턴스가 자체적으로 종속 항목을 얻는 대신 클래스의 종속 항목을 받아서 사용한다.

DI없이 클래스 생성은 다음과 같다.

```kotlin
class Car {

    private val engine = Engine()

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.start()
}
```

![1-car-engine-no-di.png](/assets/images/1-car-engine-no-di.png?raw=true)

이런 코드는 다음과 같은 이유로 문제가 될 수 있다.

- car와 engine은 밀접하게 연결되어있다. 한가지 유형의 engine을 사용하므로 서브 클래스또는 대체 구현을 쉽게 사용할 수 없다.  그때마다 새로운 car를 생성해야한다.
- 강력한 의존성은 테스트를 더욱 어렵게 만든다. car는 실제 engine 인스턴스를 사용하므로 다양한 테스트 사례에서 engine을 수정할 수 없다.

다음은 DI형태로 만든 코드이다.

```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val engine = Engine()
    val car = Car(engine)
    car.start()
}
```

![1-car-engine-di2.png](/assets/images/1-car-engine-di2.png?raw=true)

DI 기반 코드의 이점은 다음과 같다.

- car의 재사용이 가능하다. engine의 다양한 구현을 car에 전달할 수 있다.
- car의 테스트 편의성. engine의 테스트 더블을 생성하여 다양한 테스트에 맞게 구성할 수 있다.

활동 및 프래그먼트와 같은 특정 Android 프레임워크 클래스는 시스템에서 인스턴스화하므로 생성자 삽입이 불가능하다. 그래서 그 대신 필드 삽입을 사용하면 종속 항목은 클래스가 생성된 후 인스턴스화되는걸 이용한다.

```kotlin
class Car {
    lateinit var engine: Engine

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.engine = Engine()
    car.start()
}
```

## 자동 종속 항목 삽입

예전에는 라이브러리를 사용하지 않고 다양한 클래스의 종속 항목을 직접 생성, 제공 및 관리했다. 이를 직접 종속 항목 삽입 **또는 수동 종속 항목 삽입**이라고 한다.

car예에서는 종속 항목이 하나만 있었지만 종속 항목과 클래스가 많아지면 수동으로 종속 항목을 삽입하는 작업이 더 지루해질 수 있다.

수동 종속 항목 삽입에는 다음과 같은 문제도 몇 가지 있다.

- 대규모 앱의 경우 모든 종속 항목을 가져와 올바르게 연결하려면 대량의 상용구 코드가 필요할 수 있다 구체적인 예로, 실제 자동차를 만들려면 엔진, 변속기, 섀시 및 기타 부품이 필요할 수 있다. 그리고 엔진에는 실린더와 점화 플러그가 필하다.
- 종속 항목을 전달하기 전에 구성할 수 없을 때(예를 들어 지연 초기화를 사용하거나 객체 범위를 앱의 흐름으로 지정할 때)는 메모리에서 종속 항목의 전체 기간을 관리하는 맞춤 컨테이너(또는 종속 항목 그래프)를 작성하고 유지해야 한다.

종속 항목을 생성하고 제공하는 프로세스를 자동화하여 이 문제를 해결하는 라이브러리가 있다. 이는 두 가지 카테고리로 분류된다.

- 런타임 시 종속 항목을 연결하는 리플렉션 기반 솔루션
- 컴파일 타임에 종속 항목을 연결하는 코드를 생성하는 정적 솔루션

Dagger는 Google에서 유지 관리하며 자바, Kotlin 및 Android용으로 널리 사용되는 종속 항목 삽입 라이브러리다. Dagger는 종속 항목 그래프를 자동으로 생성하고 관리하여 앱에서의 DI 사용을 용이하게 한다.

## 종속 항목 삽입의 대안

종속 항목 삽입의 대안은 **서비스 로케이터**를 사용하는 것이다. 또한 서비스 로케이터 디자인 패턴은 구체적인 종속 항목에서 클래스 분리를 향상시킨다.

종속 항목을 생성하고 저장한 후 필요에 따라 이러한 종속 항목을 제공하는 서비스 로케이터라는 클래스를 생성한다.

```kotlin
object ServiceLocator {
    fun getEngine(): Engine = Engine()
}

class Car {
    private val engine = ServiceLocator.getEngine()

    fun start() {
        engine.start()
    }
}

fun main(args: Array) {
    val car = Car()
    car.start()
}
```

DI와 비교하면

- 서비스 로케이터에 필요한 종속 항목 컬렉션은 코드를 테스트하기가 더 어렵다. 모든 테스트가 동일한 전역 서비스 로케이터와 상호작용해야 하기 때문이다.
- 종속 항목은 API 노출 영역이 아닌 클래스 구현에서 인코딩된다. 따라서 클래스가 외부에서 필요한 것이 무엇인지 알기가 더 어려워진다. 결과적으로 `Car` 또는 서비스 로케이터에서 사용 가능한 종속 항목을 변경하면 참조가 실패하여 런타임 오류 또는 테스트 실패가 발생할 수 있다.
- 전체 앱의 전체 기간이 아닌 다른 기간으로 범위를 지정하려는 경우 객체의 전체 기간을 관리하기가 더 어렵다.

## **Android 앱에서 Hilt 사용**

Hilt는 Android에서 종속 항목 삽입을 위한 Jetpack의 권장 라이브러리이다.

Hilt는 프로젝트의 모든 Android 클래스에 컨테이너를 제공하고 수명 주기를 자동으로 관리함으로써 애플리케이션에서 DI를 실행하는 표준 방법을 정의해준다.

Hilt는 Dagger가 제공하는 컴파일 타임 정확성, 런타임 성능, 확장성 및 Android 스튜디오 지원의 이점을 누리기 위해 인기 있는 DI 라이브러리 Dagger를 기반으로 빌드되었다.

## 결론

종속 항목 삽입은 앱에 다음과 같은 이점을 제공한다.

- 클래스 재사용 가능 및 종속 항목 분리
- 리팩터링 편의성
- 테스트 편의성

## 참조

> [https://developer.android.com/training/dependency-injection?hl=ko](https://developer.android.com/training/dependency-injection?hl=ko)
>