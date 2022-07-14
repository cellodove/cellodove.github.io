---
title : "[Coroutine&Flow] 03장 스코프빌더2"
excerpt: "스코프빌더2"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-07-13
---

![coroutine3_image1.jpg](/assets/images/coroutine3_image1.jpg?raw=true)

## suspend 함수에서 코루틴 빌더 호출

코루틴 빌더를 `suspend`함수 안에서 호출하면 어떻게 될까?

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() {
    launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```kotlin
Unresolved reference: launch
Suspension functions can be called only within coroutine body
Unresolved reference: launch
Unresolved reference: launch
Suspension functions can be called only within coroutine body
```

다음과 같이 에러가 난다.

코루틴 빌더는 코루틴 스코프 내에서만 호출해야 한다.

## 코루틴 스코프

코루틴 스코프를 만드는 다른 방법은 스코프 빌더를 이용하는 것이다. `coroutineScope` 를 사용해보자.

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```kotlin
4!
launch1: main @coroutine#2
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
2!
3!
runBlocking: main @coroutine#1
5!
```

코루틴 스코프는 `runBlocking`을 썼을 때와 모양이 비슷하다.

하지만 큰 차이가 하나있는데 `runBlocking` 은 현재 쓰레드를 멈추게 만들고, 기다리지만 `coroutineScope` 는 현재 쓰레드를 멈추게 하지 않는다. 호출한 쪽이 `suspend`되고 시간이 되면 다시 활동하게 된다.

## Job을 이용한 제어

코루틴 빌더 `launch`는 `Job`객체를 반환하며 이를 통해 종료될 때까지 기다릴 수 있다.

`Job`객체를 사용하여 기다리거나 취소할 수 있다.

`join()`메소드는 기다리게하는 것이다.

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    job.join() // suspension point

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```kotlin
launch1: main @coroutine#2
3!
4!
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
2!
runBlocking: main @coroutine#1
5!
```

job.join()가있는 줄이 suspension point가 되어 기다리게 된다.

언제까지 기다리냐면 첫번째 런치 블록이끝날때 까지 기다린다. 첫번째 런치 블록이 끝난뒤에 다음코드들이 동작하게 된다. 다음에있는 런치들은 기다리고있기때문에 코루틴스코프에있는 println("4!")가 먼저 출력된다.

## 가벼운 코루틴

코루틴은 협력적으로 동작하기 때문에 여러 코루틴을 만드는데 큰 비용이 들지 않는다. 10만개의 간단한 일을 하는 코루틴도 큰 부담이 아니다.

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }
    job.join()

    launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    repeat(1000) {
        launch {
            println("launch3: ${Thread.currentThread().name}")
            delay(500L)
            println("2!")  
        }
    }
    println("4!")
}

fun main() = runBlocking {
    doOneTwoThree()
    println("runBlocking: ${Thread.currentThread().name}")
    println("5!")
}
```

```kotlin
launch1: main @coroutine#2
3!
4!
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
launch3: main @coroutine#5
launch3: main @coroutine#6
launch3: main @coroutine#7
launch3: main @coroutine#8
launch3: main @coroutine#9
launch3: main @coroutine#10
.
.
.
```