---
title : "[Android] Hilt를 사용한 DI(의존성 주입)2"
excerpt: "hilt설정 및 어노테이션 설명"

categories :
  - Elements

toc: true
toc_sticky: true
last_modified_at: 2022-04-16
---

![hilt_title_image2.jpg](/assets/images/hilt_title_image2.jpg?raw=true)

이제 Hilt를 사용해 DI적용된 예제 앱을 만들어 보겠다.

예제는 안드로이드 [코드랩](https://developer.android.com/codelabs/android-hilt?hl=ko#0)을 기반으로 만들었다.

사용자가 버튼을 누르면 누른 로그를 Room라이브러리를 사용하여 로컬 데이터베이스에 데이터를 저장하고 필요할때 데이터를 가지고와 표출하는 예제이다.

예제를 진행하는동안 중간중간 힐트에대해서 설명을 할 예정이다.

## 라이브러리 추가

먼저 프로젝트 그래들에 다음과 같이 추가해준다.

```groovy
buildscript {
    ext.kotlin_version = '1.5.31'
    ext.hilt_version = '2.40'
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.0.4'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

그다음 모듈 그래들에 다음과 같이 추가해준다.

```groovy
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-parcelize'
    id 'kotlin-kapt'
    id 'dagger.hilt.android.plugin'
}

dependencies {
    // Room
    implementation "androidx.room:room-runtime:2.4.2"
    kapt "androidx.room:room-compiler:2.4.2"

    // Testing dependencies
    androidTestImplementation "junit:junit:4.13.2"
    androidTestImplementation "androidx.test:core-ktx:1.4.0"
    androidTestImplementation "androidx.test.ext:junit-ktx:1.1.3"
    androidTestImplementation "androidx.test:rules:1.4.0"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.4.0"

    // Hilt dependencies
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-android-compiler:$hilt_version"

    // Hilt testing dependencies
    androidTestImplementation "com.google.dagger:hilt-android-testing:$hilt_version"
    kaptAndroidTest "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```

## UI 추가 및 Hilt설명

UI를 추가하면서 DI도 같이 하겠다.

먼저 어플레케이션 객체에 DI를 연결한다.

**LogApplication**  

- LogApplication

```kotlin
@HiltAndroidApp
class LogApplication : Application() {}
```

앱의 수명 주기에 연결된 컨테이너를 추가하려면 `@HiltAndroidApp` 을 `Application`클래스에 달아야 한다. 이를 위해 LogApplication 라는 객체를 만들어주고  어노테이션을 붙여준다.

### @HiltAndroidApp

애플리케이션 수준 종속 항목 컨테이너 역할을 하는 애플리케이션의 기본 클래스를 비롯하여 Hilt의 코드 생성을 트리거한다.

`Application`객체의 수명 주기에 연결되며 이와 관련한 종속 항목을 제공한다. 또한 앱의 상위 구성요소이므로 다른 구성요소는 이 상위 구성요소에서 제공하는 종속 항목에 액세스할 수 있다.

💡컨테이너(as hilt-component)

힐트의 컨테이너는 각 의존성(들)을 가지고(의존성을 어떻게 만드는지 알고 있다는것) **또 필요한 곳에 생명주기를 고려하여 의존성 주입을 하는 기능을 가진다.

- 의존성(클래스) 관리
- 각 의존성들이 생성되고 필요한 곳에 주입을 시켜준다.

임의로 생성한 어플리케이션 객체를 메니페스트에 연결해준다.

`android:name=".LogApplication"`

- manifests

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.peter.hilt_example">

    <application
        android:name=".LogApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Hilt_Example">
        <activity
            android:name=".ui.MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

이제 앱에서 Hilt를 사용할 준비가 되었다.

**MainActivity**

- MainActivity

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</FrameLayout>
```

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var navigator: AppNavigator

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        if (savedInstanceState == null) {
            navigator.navigateTo(Screens.BUTTONS)
        }
    }

    override fun onBackPressed() {
        super.onBackPressed()

        if (supportFragmentManager.backStackEntryCount == 0) {
            finish()
        }
    }
}
```

MainActivity 에서 힐트를 사용할려면`@AndroidEntryPoint` 을 달아야한다.

`Application`클래스에 Hilt를 설정하고 애플리케이션 수준 구성요소를 사용할 수 있게 되면 Hilt는 `@AndroidEntryPoint`주석이 있는 다른 Android 클래스에 종속 항목을 제공할 수 있다.

### @AndroidEntryPoint

- 안드로이드 생명주기에 맞는 의존성들의 컨테이너를 만든다.

이 어노터에션이 붙은 안드로이드 클래스는 해당 클래스의 생명주기를 따르는 의존성 컨테이너를 만든다. 즉 MainActivity의 수명 주기에 연결된 종속 항목 컨테이너를 생성하고 MainActivity에 인스턴스를 삽입할 수 있다.

@AndroidEntryPoint는 각 안드로이드 클래스와 짝이 맞는 hilt-component (ApplicationComponent, ActivityComponent…)를 생성한다.

컴포넌트는 각 컴포넌트 안에 있는 의존성(들)을 목적 주소지(안드로이드 클래스)로 보낼 수 있는(주입할 수 있는) 집합 장소라고 볼 수 있다.

💡 Hilt는 현재 다음 Android 클래스를 지원한다.

- Application
- Activity
- Fragment
- View
- Service
- BroadcastReceiver

⚠️ 유의 사항

- fragment에서 사용할 경우 이 fragemnt를 포함하는 activity에도 @AndroidEntryPoint를 사용해 주어야 한다.
- ComponentActivity(AppCompatActivity같이)를 상속하는 액티비티만 지원한다.
- 프래그먼트중 androidx의 프래그먼트를 상속한 애들만 지원한다.

### @Inject

- 변수(필드) 또는 생성자 파라미터에 의존성을 주입한다.

`@Inject`주석을 사용하여 Hilt에서 삽입하려는 다른 유형의 인스턴스(예: logger**,** dateFormatter)를 필드에 삽입하도록 할 수 있다. 이를 필드 삽입이라고 한다.

Inject에는 두가지가있다.

- Field injection
- Constructor injection

**Field injection**  

```kotlin
@Inject lateinit var navigator: AppNavigator
```

hilt가 어떻게 주입해야 하는지 알 경우navigator에 의존성을 주입시켜준다.

private접근자를 사용하면 hilt는 주입을 하지 못한다.그러니 hilt-field-injection시 private을 사용하면 안된다.

Hilt에서 언제 inject을 해주는지는 이[사이트](https://developer.android.com/training/dependency-injection/hilt-android#component-lifetimes)에서 확인할 수 있다.

**Constructor injection**  
의존성을 주입시키려면 힐트가 어떻게 주입 해주는지 알아야한다.

의존성 주입 방식에는 2가지가 있다.

- constructor inject 할 수 있는 클래스
  - 내가 구현한 클래스
- constructor inject 할 수 없는 클래스
  - 인터페이스, 추상 클래스 구현체(@Module 사용)
  - 내가 구현할 수 없는 클래스(3rd party library)(@Module 사용)

**contructor inject 할 수 있는 경우**  
해당 구현 클래스에 @Inject constructor(…)를 넣어 주기만 하면 된다.

```kotlin
class AppNavigatorImpl @Inject constructor(private val activity: FragmentActivity) : AppNavigator {}
```

**contructor inject 할 수 없는 경우**  
이경우는 모듈(@Module)이 필요하다.

Hilt 모듈은 `@Module`와 `@InstallIn` 를 사용한 클래스다.

### @Module

- constructor-injection 할 수 없는 경우에 사용

Hilt 모듈임을 가리킨다.(hilt가 알 수 있게)

### @IntallIn

- 모듈을 사용할 때 어느 안드로이드 컴포넌트에 맞춰 사용할 건지 지정

어느 안드로이드 클래스(activity, fragemnt etc)를 사용할건지 가리킨다. (hilt가 알 수 있게)

안드로이드 클래스의 생명주기(scope)에 맞게해당 컴포넌트(안드로이드 클래스와 대응하는 컴포넌트)를 지정하는 어노테이션이다.

모듈 생성에서 abstract, object(static) 두 경우가 있다.

### @Binds

- 모듈이 abstract 클래스인 메서드에서 사용
  - 구현체(리턴값)가 되는 파라미터를 하나만 가질 수 있다.
  - • function body가 없다.

인터페이스는 constructor를 삽입할 수 없다. 그대신 `@Binds`로 지정된 추상 함수를 생성하여 Hilt에 결합 정보를 제공한다.

```kotlin
//abstract, @Binds주목
@InstallIn(ActivityComponent::class)
@Module
abstract class LoggingInMemoryModule{

    @Binds
    abstract fun bindInMemoryLogger(impl:LoggerInMemoryDataSource):LoggerDataSource

}
```

### @Provides

- 모듈이 object 인 메서드에서 사용
  - 구현에 필요한 파라미터를 여러개 가질 수 있다.(없어도 된다.)
  - 해당 의존성을 사용할 때 매번 호출을 한다.
  - function body 존재한다.

object도 마찬가지로 constructor를 삽입할 수 없다.

클래스가 외부 라이브러리에서 제공되므로 클래스를 소유하지 않은 경우 또는 빌더 패턴
으로 인스턴스를 생성해야 하는 경우에 `@Provides` 를 지정하여 Hilt에 결합 정보를 제공한다.

```kotlin
//object,@Provides 주목 
@InstallIn(ApplicationComponent::class)
@Module
object DatabaseModule {

    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext appContext: Context): AppDatabase {
        return Room.databaseBuilder(
            appContext,
            AppDatabase::class.java, "logging.db"
        ).build()
    }

    @Provides
    fun provideLogDao(database: AppDatabase): LogDao {
        return database.logDao()
    }
}
```

@Binds나 @Provides에서 Hilt가 필요한 건 리턴 타입과 파라미터들 뿐이다.

코드를 설명하기위해선 먼저 힐트에대해 알아야할 필요가있었다. 다음 장에서는 본격적으로 예제를 나가겠다.

## 참조

> [https://developer.android.com/training/dependency-injection?hl=ko](https://developer.android.com/training/dependency-injection?hl=ko)
>
> [https://developer.android.com/codelabs/android-hilt?hl=ko#4](https://developer.android.com/codelabs/android-hilt?hl=ko#4)
>
> [https://f2janyway.github.io/android/hilt/](https://f2janyway.github.io/android/hilt/)
>