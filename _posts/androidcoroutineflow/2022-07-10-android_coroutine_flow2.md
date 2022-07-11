---
title : "[Coroutine&Flow] 02장 스코프빌더1"
excerpt: "스코프빌더1"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-07-10
---

![coroutine2_image1.jpg](/assets/images/coroutine2_image1.jpg?raw=true)

## 간단한 코루틴

코루틴을 만드는 함수를 코루틴 빌더라고 한다.

코루틴을 만드는 가장 간단한 함수는 `runBlocking` 이다. `runBlocking` 은 코루틴을 만들고 코드 블록의 수행이 끝날 때까지 `runBlocking` 다음의 코드를 수행하지 못하게 막는다. 그래서 이름이 블로킹이다.

코드 실행은 [플레이그라운드](https://play.kotlinlang.org/)에서 실행시키면된다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    println(Thread.currentThread().name)
    println("Hello")
}
```

해당 코드를 실행시키면 다음과 같이 나온다.

```kotlin
main @coroutine#1
Hello
```

스레드 이름이 `main @coroutine1` 이다. 메인 스레드에서 수행되는데 뒤에 수식어 `@coroutine1`이 붙어 있다.

## 코루틴 빌더의 수신 객체

`runBlocking`안에서 `this`를 수행하면 코루틴이 수신 객체(Receiver)인 것을 알 수 있다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    println(this)
    println(Thread.currentThread().name)
    println("Hello")
}
```

```kotlin
"coroutine#1":BlockingCoroutine{Active}@75412c2f
main @coroutine#1
Hello
```

`"coroutine#1":BlockingCoroutine{Active}@3930015a`이런 형태의 결과가 나온다.

`BlockingCoroutine`은 `CoroutineScope`의 자식이다. 코틀린 코루틴을 쓰는 모든 곳에는 코루틴 스코프(`CoroutineScope`)가 있다고 생각하면 된다.

**코루틴의 시작은 코루틴 스코프이다.**

## 코루틴 컨텍스트

코루틴 스코프는 코루틴을 제대로 처리하기 위한 정보, 코루틴 컨텍스트(`CoroutineContext`)를 가지고 있다. 수신 객체의 `coroutineContext`를 호출해 내용을 확인해보자.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    println(coroutineContext)
    println(Thread.currentThread().name)
    println("Hello")
}
```

```kotlin
[CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@f5f2bb7, BlockingEventLoop@73035e27]
main @coroutine#1
Hello
```

## launch 코루틴 빌더

이제 코루틴 내에서 다른 코루틴을 수행해보겠다.

`launch` 라는 빌더를 사용해 코드를 수행항다. `launch` 는 코루틴 빌더이다. 새로운 코루틴을 만들기 때문에 새로운 코루틴 스코프를 만든다.

`launch` 는 할 수 있다면 다른 코루틴 코드를 같이 수행시키는 코루틴 빌더이다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    println("Hello")
}
```

```kotlin
runBlocking: main @coroutine#1
Hello
launch: main @coroutine#2
World!
```

`launch` 코루틴 빌더에 있는 코드가 `runBlocking` 이 있는 메인 흐름보다 늦게 수행되었다. 둘 다 메인 스레드를 사용하기 때문에 `runBlocking` 의 코드들이 메인 스레드를 다 사용할 때 까지 `launch` 의 코드 블록이 기다리는 것이다.

`runBlocking`은 Hello를 출력하고 나서 종료하지는 않고 `launch`코드블록의 내용이 다 끝날 때까지 기다린다.

## delay 함수

Hello를 조금 더 늦게 수행시키기 위해 `delay`함수를 호출해 보자. 인자로 밀리세컨드 단위의 시간을 지정할 수 있다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("Hello")
}
```

```kotlin
runBlocking: main @coroutine#1
launch: main @coroutine#2
World!
Hello
```

프린트 문이 호출된 이후 `delay`가 호출되는데 이때 `runBlocking`의 코루틴이 멈추게 되고 `launch`의 코드 블록이 먼저 수행된다.

## 코루틴 내에서 sleep

`Thread.sleep`을 호출하면 어떻게 될까

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch: ${Thread.currentThread().name}")
        println("World!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    Thread.sleep(500)
    println("Hello")
}
```

```kotlin
runBlocking: main @coroutine#1
Hello
launch: main @coroutine#2
World!
```

`Thread.sleep`을 하면 코루틴이 아무 일을 하지 않는 동안에도 스레드를 양보하지 않고 독점한다.

## 한번에 여러 launch

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("2!")
}
```

```kotlin
runBlocking: main @coroutine#1
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
2!
3!
```

딜레이 값을 바꿔 보면 `suspend`된 이후 깨어나는 순서에 따라 출력 결과가 달라진다.

## 상위 코루틴은 하위 코루틴을 끝까지 책임진다

`runBlocking`안에 두 `launch`가 속해 있는데 계층화되어 있어 구조적이다. `runBlocking`은 그 속에 포함된 `launch`가 다 끝나기 전까지 종료되지 않는다.

```kotlin
import kotlinx.coroutines.*

fun main() {
    runBlocking {
        launch {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }
        launch {
            println("launch2: ${Thread.currentThread().name}")
            println("1!")
        }
        println("runBlocking: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")
    }
    print("4!")
}
```

```kotlin
runBlocking: main @coroutine#1
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
2!
3!
4!
```

## suspend 함수

`delay`, `launch` 등 지금까지 봤던 함수들은 코루틴 내에서만 호출 할 수 있다. 그럼 이 함수들을 포함한 코드들을 어떻게 함수로 분리할 수 있을까?

코드의 일부를 함수로 분리할 때는 함수의 앞에 `suspend` 키워드를 붙이면 된다.

```kotlin
import kotlinx.coroutines.*

suspend fun doThree() {
    println("launch1: ${Thread.currentThread().name}")
    delay(1000L)
    println("3!")
}

suspend fun doOne() {
    println("launch1: ${Thread.currentThread().name}")
    println("1!")
}

suspend fun doTwo() {
    println("runBlocking: ${Thread.currentThread().name}")
    delay(500L)
    println("2!")
}

fun main() = runBlocking {
    launch {
        doThree()
    }
    launch {
        doOne()
    }
    doTwo()
}
```

```kotlin
runBlocking: main @coroutine#1
launch1: main @coroutine#2
launch1: main @coroutine#3
1!
2!
3!
```

`doOne()`은 `delay`와 같은 함수(`suspend`인 함수)를 호출하지 않았기 때문에 `suspend`를 붙이지 않은 일반 함수로 해도 된다.

만약 `suspend` 함수를 다른 함수에서 호출하려면 그 함수가 `suspend` 함수이거나 코루틴 빌더를 통해 코루틴을 만들어야 한다.