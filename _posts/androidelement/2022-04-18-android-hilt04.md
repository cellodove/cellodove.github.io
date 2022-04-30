---
title : "[Android] Hilt를 사용한 DI(의존성 주입)4"
excerpt: "Hilt를 사용하여 테스트 작성"

categories :
  - Elements

toc: true
toc_sticky: true
last_modified_at: 2022-04-18
---

![checklist.jpg](/assets/images/checklist.jpg?raw=true)

앱이이제 Hilt로 완전히 마이그레이션됐다. 그렇다면이제 hilt를 사용했을때의 장점이였던 테스트 용이성을 확인해보자.

예제에있는 UI테스트를 해보겠다.

Hilt는 프로덕션 코드에서 발생하는 것처럼 UI 테스트에 종속 항목을 삽입한다.

**Hilt는 각 테스트의 새로운 구성요소 집합을 자동 생성하므로 Hilt를 사용한 테스트에는 유지관리가 필요하지 않다.**

## 라이브러리 추가

먼저 라이브러리를 추가해준다.

```groovy
dependencies {
    // Hilt testing dependencies
    androidTestImplementation "com.google.dagger:hilt-android-testing:$hilt_version"
    kaptAndroidTest "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```

## 커스텀 **TestRunner**

힐트를 사용한 테스트를 진행할때는 힐트를 지원하는 `Application` 에서 실행되어야한다. 라이브러리에는 이미 UI테스트를 실행하는데 사용할 수 있는 `HiltTestApplication` 클래스가 포함되어 있다.

[계측 테스트](https://developer.android.com/training/testing/ui-testing?hl=ko)또는 [Robolectric 테스트](http://robolectric.org/)에서 실행되도록 테스트 애플리케이션을 설정해야 한다.

Hilt에만 관련된 것이 아니라 테스트에서 실행할 맞춤 애플리케이션을 지정하는 방법에 관한 일반적인 내용이다.

테스트에 사용할 `Application` 을 지정하려면 프로젝트에서 새 테스트 러너를 생성한다.

`androidTest` 폴더안에 CustomTestRunner파일을 만든다.

![hilt_di4_image1.JPG](/assets/images/hilt_di4_image1.jpg?raw=true)

`newApplication` 함수를 재정의하고 생성된 Hilt 테스트 애플리케이션의 이름을 전달한다.

- CustomTestRunner

```kotlin
class CustomTestRunner : AndroidJUnitRunner() {
    override fun newApplication(cl: ClassLoader?, name: String?, context: Context?): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

이제 이 테스트 실행기를 실제로 테스트에 사용하도록 프로젝트에 알려야한다.

그래들에서 `testInstrumentationRunner` 속성에 테스트 실행기를 명시한다.

```groovy
android {

    defaultConfig {

        testInstrumentationRunner "com.peter.hilt_example.CustomTestRunner"
    }
}
```

이제 UI테스트에서 Hilt를 사용할 준비가 되었다.

## **Hilt를 사용하는 테스트 실행**

테스트 클래스에서 hilt를 사용하기 위해 다음을 행해야한다.

- 각 테스트의 hilt 구성요소 생성을 담당하는 `@HiltAndroidTest` 주석을 추가한다.
- 구성요소의 상태를 관리하고 테스트에 삽입을 실행하는 데 사용되는 `HiltAndroidRule`을 사용한다.

AppTest파일에 해당내용을 추가해준다.

- AppTest

```kotlin
@RunWith(AndroidJUnit4::class)
@HiltAndroidTest
class AppTest {

    @get:Rule
    var hiltRule = HiltAndroidRule(this)

    @Test
    fun happyPath() {
        ActivityScenario.launch(MainActivity::class.java)

        // Check Buttons fragment screen is displayed
        Espresso.onView(withId(R.id.textView))
            .check(ViewAssertions.matches(ViewMatchers.isDisplayed()))

        // Tap on Button 1
        Espresso.onView(withId(R.id.button1)).perform(ViewActions.click())

        // Navigate to Logs screen
        Espresso.onView(withId(R.id.all_logs)).perform(ViewActions.click())

        // Check Logs fragment screen is displayed
        Espresso.onView(ViewMatchers.withText(Matchers.containsString("Interaction with 'Button 1'")))
            .check(ViewAssertions.matches(ViewMatchers.isDisplayed()))
    }
}
```

`@RunWith`로 AndroidJUnit4 라이브러리를 사용하게 하고 `@Test`로 하나의 단위테스트를 만들었다.

테스트를 작성하다보면 여러 테스트 클래스에서 테스트 사전 작업, 직후 작업이 동일할 때가 있다. 이런 작업을 하나의 Rule로 만들어 놓으면 @Before, @After마다 자동으로 수행되어 보일러 플레이트 코드를 줄일 수 있다. 이럴때 `@get:Rule` 를 사용한다.

`HiltAndroidRule`은 구성요소의 상태를 관리하고, 테스트에서 삽입을 실행하는 데 사용된다.

`ActivityScenario` 는 [AndroidX 테스트](https://developer.android.com/training/testing?hl=ko) 라이브러리에있는 내용이다. `ActivityScenario.launch` 로 지정한 액티비티를 시작하고 액티비티 시나리오를 구성한다.

Espresso도 마찬가지로 AndroidX 테스트 제공하는 테스트 프레임워크이다. 사용자 상호작용을 시뮬레이션하는 UI테스트를 작성하기 위한 API를 제공한다.

코드를 보면 알 수 있듯이`onView`를 통해 뷰를 설정해주어 액션을 설정할 수 있다.

이제 테스트 버튼을 누르면 테스트가 진행된다.

![hilt_di4_image2.JPG](/assets/images/hilt_di4_image2.jpg?raw=true)

그리고 테스트가 완료되었다고 표시된다.

![hilt_di4_image3.JPG](/assets/images/hilt_di4_image3.jpg?raw=true)

이러면 테스트가 완료된것이다. 기기를 연결하고 테스트를하게되면 실제고 앱이 동작함으로 테스트가 진행되게 된다.

전체 코드는 아래 링크에서 볼 수 있다. 전 예제 링크와 동일하다.

[https://github.com/cellodove/Hilt_Example](https://github.com/cellodove/Hilt_Example)

## Hilt 테스트 추가 내용

당연히 테스트를 좀더 복잡하게 할 수 있다. 테스트를 위한 다양한 기능들이 있다.

### 유형 삽입

타입을 테스트에 삽입하려면 힐트 의존성주입할때와 마찬가지로 `@Inject` 를 사용한다. Hilt에 `@Inject`필드를 채우도록 알리려면 `hiltRule.inject()`를 호출한다.

```kotlin
@HiltAndroidTest
class SettingsActivityTest {

  @get:Rule
  var hiltRule = HiltAndroidRule(this)

  @Inject
  lateinit var analyticsAdapter: AnalyticsAdapter

  @Before
  fun init() {
    hiltRule.inject()
  }

  @Test
  fun `happy path`() {
    // Can already use analyticsAdapter here.
  }
}
```

여기서 처음보이는 주석들을 설명하자면 Junit의 어노테이션들이다.

- @BeforeClass : 테스트 클래스 테스트 시작시 1번만호출
- @Before : 테스트 케이스 시작전 각각 호출
- @After : 테스트 케이스 완료시 각각 호출
- @AfterClass : 테스트 클래스 모든 테스트 완료시 1번 호출

![junit_anotation.png](/assets/images/junit_anotation.png?raw=true)

### 결합 대체

실제 사용중인 종속 항목 대신, 테스트용 모의 종속 항목을 넣어야한다면 모의 항목을 사용하도록 힐트에 알려야한다.

예를 들어 프로덕션 코드에서 다음과 같이 `AnalyticsService`의 결합을 선언한다고 가정해 보겠다.

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
abstract class AnalyticsModule {

  @Singleton
  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}
```

먼저 다음과 같이 테스트 클래스에서 `@UninstallModules`주석을 사용하여 프로덕션 모듈을 무시하도록 Hilt에 알린다.

```kotlin
@UninstallModules(AnalyticsModule::class)
@HiltAndroidTest
class SettingsActivityTest {}
```

그 다음 결합을 대체해야 한다. 다음과 같이 테스트 클래스 내에 테스트 결합을 정의하는 새 모듈을 만든다.

```kotlin
@UninstallModules(AnalyticsModule::class)
@HiltAndroidTest
class SettingsActivityTest {

  @Module
  @InstallIn(ApplicationComponent::class)
  abstract class TestModule {

    @Singleton
    @Binds
    abstract fun bindAnalyticsService(
      analyticsServiceImpl: AnalyticsServiceImpl
    ): AnalyticsService
  }
}
```

이는 단일 테스트 클래스의 결합만 대체한다. 모든 테스트 클래스의 결합을 대체하려면 Robolectric 테스트의 `test`모듈 또는 계측 테스트의 `androidTest`모듈에 테스트 결합을 삽입한다.

⚠️ 참고

`@InstallIn`주석이 지정되지 않은 모듈은 제거할 수 없다. 제거하려고 하면 컴파일 오류가 발생한다.

### 새 값 결합

`@BindValue`주석을 사용하면 테스트의 필드를 Hilt 종속 항목 그래프에 쉽게 결합할 수 있다. 필드에 `@BindValue`로 주석을 지정하면 이 필드에 대해 존재하는 한정자와 함께 선언된 필드 유형 아래에서 결합된다.

다음과 같이 `@BindValue`를 사용하여 `AnalyticsService`를 가짜로 대체할 수 있다.

```kotlin
@UninstallModules(AnalyticsModule::class)
@HiltAndroidTest
class SettingsActivityTest {

  @BindValue analyticsService: AnalyticsService = FakeAnalyticsService()

}
```

이렇게 하면 동시에 두 작업을 모두 실행할 수 있으므로 간단히 결합을 대체하고 테스트에서 결합을 참조할 수 있다.

## 특수한 사례

Hilt는 비표준 사용 사례를 지원하는 기능도 제공한다.

### 테스트용 맞춤 애플리케이션

테스트 애플리케이션이 다른 애플리케이션을 확장해야 하므로 `HiltTestApplication`을 사용할 수 없다면 새 클래스 또는 인터페이스에 `@CustomTestApplication`으로 주석을 지정한다. 생성된 Hilt 애플리케이션이, 확장할 기본 클래스 값을 전달하여 애플리케이션을 테스트할 수 있게 해준다.

`@CustomTestApplication`은 매개변수로 전달한 애플리케이션을 확장하는 Hilt로 테스트할 준비가 된 애플리케이션 클래스를 생성한다.

```kotlin
@CustomTestApplication(BaseApplication::class)
interface HiltTestApplication
```

### 계측 테스트의 여러 **TestRule 객체**

테스트에 다른 `TestRule`객체가 있다면 모든 규칙이 함께 작동하도록 하는 몇 가지 방법이 있다.

다음과 같이 규칙을 함께 래핑할 수 있다.

```kotlin
@HiltAndroidTest
class SettingsActivityTest {

  @get:Rule
  var rule = RuleChain.outerRule(HiltEmulatorTestRule(this)).
        around(SettingsActivityTestRule(...))

  // UI tests here.
}
```

또는 `HiltEmulatorTestRule`이 먼저 실행되는 동일한 수준에서 두 규칙을 모두 사용할 수 있다. `@Rule`주석의 `order`속성을 사용하여 실행 순서를 지정한다.(JUnit 버전 4.13 이상에서만 작동한다.)

```kotlin
@HiltAndroidTest
class SettingsActivityTest {

  @get:Rule(order = 0)
  var hiltEmulatorTestRule = HiltEmulatorTestRule()

  @get:Rule(order = 1)
  var settingsActivityTestRule = SettingsActivityTestRule()

  // UI tests here.
}
```

## 참조

> [https://developer.android.com/training/dependency-injection/hilt-testing?hl=ko#ui-test](https://developer.android.com/training/dependency-injection/hilt-testing?hl=ko#ui-test)
>
> [https://developer.android.com/training/testing/ui-testing?hl=ko](https://developer.android.com/training/testing/ui-testing?hl=ko)
>
> [https://developer.android.com/codelabs/android-hilt?hl=ko#9](https://developer.android.com/codelabs/android-hilt?hl=ko#9)
>
> [https://developer.android.com/training/testing/ui-testing/espresso-testing?hl=ko#webviews](https://developer.android.com/training/testing/ui-testing/espresso-testing?hl=ko#webviews)
>
> [https://developer.android.com/training/testing?hl=ko](https://developer.android.com/training/testing?hl=ko)
>
> [https://beomseok95.tistory.com/302](https://beomseok95.tistory.com/302)
>