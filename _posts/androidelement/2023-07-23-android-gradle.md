---
published: true
title : "[Android] Gradle의 buildSrc, precompiled에 대하여"
excerpt: "Gradle precompiled"

categories :
  - Elements
  - 
toc: true
toc_sticky: true
last_modified_at: 2022-07-23
---

![android_gradle_precompile_image1.jpg](/assets/images/android_gradle_precompile_image1.jpg?raw=true)

## Gradle 스크립트

요즘 안드로이드 프로젝트들은 모듈별로 나누어서 개발을 하기때문에 많은 모듈들이 만들어진다. 모듈이 늘어날수록 빌드시간이 길어지고 그래들 스크립트에 모듈별 그레들 세팅들도 늘어나게된다. 

안드로이드 모듈을 만들면 이런식으로 중복되는 세팅을 했다. 모듈별로 유지, 관리하기가 매우 까다롭다.

- /module_a/build.gradle

```kotlin
plugins {
    id("com.android.library")
    id("kotlin-android")
    id("kotlin-android-extensions")
}

android {
    compileSdkVersion 29

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        consumerProguardFiles "consumer-rules.pro"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro"
        }
    }  

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.core:core-ktx:1.3.2"
    implementation "androidx.appcompat:appcompat:1.2.0"

    testImplementation "junit:junit:4.13.1"
    androidTestImplementation "androidx.test.ext:junit:1.1.2"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
}
```

경우에따라 각각의 모듈마다 추가 플러그인을 적용하거나 다른 종속성을 추가할 수 있지만 대부분의 구성이 중복된다. 모든 종류의 코드를 복제하는 것은 코드베이스를 유지, 관리하기 어렵게 만들고 복사본이 생길때 실수가 발생하기 쉽게된다.

이러한 문제를 방지하기 위한 방법중 하나가 `buildSrc`를 사용하는 것이다.

## buildSrc

buildSrc는 프로젝트의 다른 부분보다 먼저 gradle에 의해 자동으로 인식되고 컴파일된다. 사용자 지정 작업 또는 플러그인과 같은 명령형 빌드를 정의하고 유지 관리하기에 좋은 위치이다.

컴파일된 코드는 buildSrc 루트 빌드 스크립트의 클래스 경로에 배치되어 프로젝트의 다른 모든 모듈에서 사용할 수 있다. 예를들어 간단한 config클래스를 정의하고 프로젝트를 동기화한 후 gradle 스크립트에서 사용할 수있다.

- /buildSrc/src/main/java/Config.kt

```kotlin
object Config {
    const val COMPILE_SDK = 29
    const val TARGET_SDK = 29
    const val MIN_SDK = 21
}
```

- /module_a/build.gradle

```kotlin
android {
    compileSdkVersion Config.COMPILE_SDK

    defaultConfig {
        minSdkVersion Config.MIN_SDK
        targetSdkVersion Config.TARGET_SDK
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    ...
}
...
```

### buildSrc 구조

buildSrc는 모듈로 인식되고 다른 모듈과 동일한 구조를 따른다.

- 메인 소스 코드는 다음 위치에있다. `/buildSrc/src/main/<language>/`
- 테스트 소스 코드는 다음 위치에 있다. `/buildSrc/src/test/<language>/`
- Gradle 스크립트는 다음 위치에 있다. `/buildSrc/build.gradle`

![android_gradle_precompile_image2.png](/assets/images/android_gradle_precompile_image2.png?raw=true)

### 커스텀 task

static constants를 정의하는 데만 국한되지 않는다. Gradle 작업에서 일부 동작을 캡슐화 해본다.

단순화를 위해 현재 JVM 메모리 통계를 stdout으로 출력하는 코드를 만들어보자.

```kotlin
open class ReportJvmMemoryTask : DefaultTask() {
    @TaskAction
    fun run() {
        val runtime = Runtime.getRuntime()
        println("Free JVM memory: ${runtime.freeMemory() shr 20}MB.")
        println("Total JVM memory: ${runtime.totalMemory() shr 20}MB.")
        println("Max JVM memory: ${runtime.maxMemory() shr 20}MB.")
    }
}
```

buildSrc에 코드를 만들었지만 그 자체로는 효과가 없다. 모듈의 Gradle 스크립트에 등록해야 한다.

```kotlin
task reportJvmMemory(type: ReportJvmMemoryTask)
build.finalizedBy(reportJvmMemory)
```

### 커스텀 플러그인

커스텀 task는 유용할 수 있지만 모듈에서 모든 중복 구성을 제거하는데 도움이 되지 않는다. 한단계 더 나아가 보자.

우선 Android(ex: BaseExtension) 및 Kotlin(ex: KotlinJvmOptions) 플러그인에서 제공하는 API를 사용할 예정이므로 buildSrc 모듈에 적절한 종속성을 추가해야 한다.

- /buildSrc/build.gradle

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm") version "1.4.10"
}

repositories {
    jcenter()
    google()
}

dependencies {
    implementation("com.android.tools.build:gradle:4.1.0")
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin")
}
```

이제 커스텀 플러그인을 정의한다.

- /buildSrc/src/main/kotlin/BaseAndroidLibrary.kt

```kotlin
class BaseAndroidLibrary : Plugin<Project> {

    override fun apply(target: Project) {
        target.plugins.apply("com.android.library")
        target.plugins.apply("kotlin-android")
        target.plugins.apply("kotlin-android-extensions")

        target.extensions.configure(BaseExtension::class.java) { android ->
            android.compileSdkVersion(Config.COMPILE_SDK)

            android.defaultConfig {
                it.minSdkVersion(Config.MIN_SDK)
                it.targetSdkVersion(Config.TARGET_SDK)
                it.versionCode = 1
                it.versionName = "1.0"

                it.multiDexEnabled = true
                it.testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
                it.consumerProguardFiles("consumer-rules.pro")
            }

            android.compileOptions {
                it.sourceCompatibility = JavaVersion.VERSION_1_8
                it.targetCompatibility = JavaVersion.VERSION_1_8
            }

            android.buildTypes {
                it.named("release") { release ->
                    release.isMinifyEnabled = false
                    release.proguardFiles(android.getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
                }
            }
        }
    }
}
```

이전과 마찬가지로 여전히 모듈 `Config`내에 정의된 클래스에 액세스한다. `buildSrc`플러그인이 Kotlin으로 작성되었으므로 extension, sealed 클래스 objects 등 모든 언어 기능을 원하는 대로 활용할 수 있다.

이제 android 공통 구성이 플러그인에 캡슐화되었다. 모든 모듈에 해당 플러그인을 적용하고 코드 블록을 모두 제거할 수 있다.

```kotlin
apply plugin: BaseAndroidLibrary

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.core:core-ktx:1.3.2"
    implementation "androidx.appcompat:appcompat:1.2.0"

    testImplementation "junit:junit:4.13.1"
    androidTestImplementation "androidx.test.ext:junit:1.1.2"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
}
```

## kotlin-dsl 및 Precompile

그레들 스크입트에서 캡슐화된 플러그인으로 로직을 뺌으로 약간의 가독성을 희생했고 원래 스크립트와 동일한 결과를 얻기 위해 코틀린 또는 자바로 코드구성을 만들어야 했다.

코틀린 익스텐션 함수를 사용해 좀더 간결하게 만들 수 있다.

```kotlin
fun Project.android(configure: LibraryExtension.() -> Unit) {
    extensions.configure(LibraryExtension::class.java, configure)
}

fun BaseExtension.kotlinOptions(configure: KotlinJvmOptions.() -> Unit) {
    (this as ExtensionAware).extensions.configure(KotlinJvmOptions::class.java, configure)
}
```

하지만 코드를 재구성 해야한다는 불편함은 여전하다.

**`kotlin-dsl` 플러그인이 Kotlin의 유형 안전성과 Groovy의 역동적인 세계 사이의 간극을 메워준다.**

이 기능을 사용하기 위해 `kotlin-ds**l` 와** `kotlin-dsl-precompiled-script-plugins` 를 추가한다.

- /buildSrc/build.gradle.kts

```kotlin
plugins {
    `kotlin-dsl`
    `kotlin-dsl-precompiled-script-plugins`
}

repositories {
    google()
    mavenCentral()
    gradlePluginPortal()
}

object PluginVersion {
    const val ANDROID_GRADLE = "7.3.1"
    const val KOTLIN = "1.8.0"
    const val HILT = "2.44.2"
}

dependencies {
    implementation("com.android.tools.build:gradle:${PluginVersion.ANDROID_GRADLE}")
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin:${PluginVersion.KOTLIN}")
    implementation("com.google.dagger:hilt-android-gradle-plugin:${PluginVersion.HILT}")
}
```

그다음 처음에 만들었던 `BaseAndroidLibrary.kt` 코틀린 코드를 kotlin gradle 스크립트로 바꿔준다.

관리를 위해 패키지와 파일이름을 변경하였다.

- /buildSrc/main/java/precompiled/precompiled.gradle.kts

```kotlin
import dependencies.AndroidEnvironmentValue
import dependencies.Dependency
import dependencies.TestDependency
import org.gradle.kotlin.dsl.dependencies

plugins {
    id("com.android.library")
    id("kotlin-android")
    id("kotlin-kapt")
    id("dagger.hilt.android.plugin")
}

android {
    compileSdk = AndroidEnvironmentValue.COMPILE_SDK

    defaultConfig {
        minSdk = AndroidEnvironmentValue.MIN_SDK
        targetSdk = AndroidEnvironmentValue.TARGET_SDK
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    buildFeatures {
        viewBinding = true
        dataBinding = true
    }

    kotlinOptions {
        jvmTarget = "11"
    }
    hilt {
        enableExperimentalClasspathAggregation = true
    }
}

dependencies {

    implementation(Dependency.KTX.CORE)
    implementation(Dependency.KTX.ACTIVITY)
    implementation(Dependency.KTX.FRAGMENT)
    implementation(Dependency.KTX.LIFECYCLE_RUNTIME_KTX)
    implementation(Dependency.KTX.LIFECYCLE_EXTENSIONS)
    implementation(Dependency.KTX.LIFECYCLE_VIEW_MODEL)
    implementation(Dependency.KTX.NAVIGATION_FRAGMENT)
    implementation(Dependency.KTX.NAVIGATION_RUNTIME)
    implementation(Dependency.KTX.NAVIGATION_UI)

    implementation(Dependency.AndroidX.PAGING3)
    implementation(Dependency.AndroidX.APP_COMPAT)
    implementation(Dependency.AndroidX.CONSTRAINT_LAYOUT)
    implementation(Dependency.AndroidX.PAGING3)
    implementation(Dependency.AndroidX.SWIPE_REFRESH)

    implementation(Dependency.Google.GSON)

    implementation(Dependency.Kotlin.REFLECTION)
    implementation(Dependency.Kotlin.SDK)
    implementation(Dependency.Kotlin.COROUTINE_CORE)
    implementation(Dependency.Kotlin.COROUTINE_ANDROID)

    implementation(Dependency.Google.MATERIAL3)

    kapt(Dependency.Hilt.APT)
    implementation(Dependency.Hilt.CORE)

    testImplementation(TestDependency.JUNIT)
    testImplementation(TestDependency.Kotlin.TEST)
    testImplementation(TestDependency.Kotlin.JUNIT_TEST)
    testImplementation(TestDependency.Kotlin.COROUTINE_TEST)
    testImplementation(TestDependency.AndroidX.TEST_CORE)
    testImplementation(TestDependency.MockK.CORE)
}
```

프로젝트를 동기화할 때 플러그인은 블록에 적용된 모든 플러그인을 검사하고 해당 플러그인이 제공하는 모든 구성에 대한 접근자를 생성한다. (해당 코드는 자동으로 생성된다.)

- /buildSrc/main/java/PrecompiledPlugin

```kotlin
/**
 * Precompiled [precompiled.gradle.kts][Precompiled_gradle] script plugin.
 *
 * @see Precompiled_gradle
 */
class PrecompiledPlugin : org.gradle.api.Plugin<org.gradle.api.Project> {
    override fun apply(target: org.gradle.api.Project) {
        try {
            Class
                .forName("Precompiled_gradle")
                .getDeclaredConstructor(org.gradle.api.Project::class.java, org.gradle.api.Project::class.java)
                .newInstance(target, target)
        } catch (e: java.lang.reflect.InvocationTargetException) {
            throw e.targetException
        }
    }
}
```

이제 이 생성된 플러그인을 적용해본다. 

프리 컴파일 스크립트는 파일 이름과 동일한 ID가 할당된다.

/feature/build.gradle

```kotlin
plugins{
    id("precompiled")
}
```

이 3줄이면 사실상 끝이다 그리고 해당 피쳐에 필요한 종속성이 있을경우 추가해주면된다.

```kotlin
plugins{
    id("precompiled")
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
    implementation "androidx.core:core-ktx:1.3.2"
    implementation "androidx.appcompat:appcompat:1.2.0"

    testImplementation "junit:junit:4.13.1"
    androidTestImplementation "androidx.test.ext:junit:1.1.2"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.3.0"
}
```

Groovy에서도 프리 컴파일 플러그인 스크립트를 설정할 수도 있지만 설정이 달라 권장되지 않는다.

## 결론

모듈에서 공통 빌드 구성을 추출하고 커스텀 플러그인에 캡슐화하는 것은 모든 모듈이 동일한 방식으로 구성되도록하는 좋은 방법이다.

`precompiled-scripts-plugin` 을 함께 사용하면 추출된 형태, 구성을 그대로 사용할 수 있으며 Gradle이 플러그인을 생성한다.

Kotlin DSL 플러그인을 사용하면 Kotlin으로 빌드 스크립트를 작성할 수 있으므로 사용자 지정 작업 및 플러그인을 작성하거나 다른 플러그인이 광범위한 구성을 필요로 할 때 더 나은 IDE 통합을 제공할 수 있다.

그리고 빌드가 실행될 때마다 빌드 스크립트를 해석하는 대신 미리 컴파일된 스크립트를 바이트코드로 한 번 컴파일한 다음 후속 빌드에서 바이트코드로 실행하므로 빌드 속도를 높일 수 있다.

해당 테스트 코드는 이곳에서 확인할 수 있다.

[https://github.com/cellodove/Precompile-Example](https://github.com/cellodove/Precompile-Example)

## 참조

[https://docs.gradle.org/current/userguide/custom_plugins.html#sec:precompiled_plugins](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:precompiled_plugins)

[https://antimonit.github.io/2020/12/06/to-buildsrc-and-back.html](https://antimonit.github.io/2020/12/06/to-buildsrc-and-back.html)