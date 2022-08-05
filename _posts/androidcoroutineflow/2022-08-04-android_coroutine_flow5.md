---
title : "[Coroutine&Flow] 05장 서스펜딩 함수"
excerpt: "서스펜딩 함수"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-08-04
---

![coroutine5_image1.jpg](/assets/images/coroutine5_image1.jpg?raw=true)

## **suspend 함수들의 순차적인 수행**

순차적으로 `suspend`함수를 수행시켜보자.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

fun main() = runBlocking {
    val elapsedTime = measureTimeMillis {
        val value1 = getRandom1()
        val value2 = getRandom2()
        println("${value1} + ${value2} = ${value1 + value2}")
    }
    println(elapsedTime)
}
```

```kotlin
98 + 47 = 145
2027
```

대략 2000ms 이상 수행된다는 것을 볼 수 있다.

순차적으로 수행되었기 때문에 `getRandom1`이 1000ms 정도를 소비하고 `getRandom2`가 1000ms 정도 소비하는 것이다.

## **async를 이용해 동시 수행하기**

`aync`키워드를 이용하면 동시에 다른 블록을 수행할 수 있다. `launch`와 비슷하게 보이지만 수행 결과를 `await`키워드를 통해 받을 수 있다는 차이가 있다.

결과를 받아야 한다면 `async`, 결과를 받지 않아도 된다면 `launch`를 선택하면 된다.

`await`키워드를 만나면 `async`블록이 수행이 끝났는지 확인하고 아직 끝나지 않았다면 `suspend`
되었다 나중에 다시 깨어나고 반환값을 받아온다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

fun main() = runBlocking {
    val elapsedTime = measureTimeMillis {
        val value1 = async { getRandom1() }
        val value2 = async { getRandom2() }
        println("${value1.await()} + ${value2.await()} = ${value1.await() + value2.await()}")
    }
    println(elapsedTime)
}
```

```kotlin
438 + 356 = 794
1060
```

수행 결과를 보면 `getRandom1`과 `getRandom2`를 같이 수행해서 경과시간이 거의 반으로 줄어들었다.

많은 다른 언어들이 `async`, `await` 키워드를 가지고 있는데 그것과는 비슷하게 보이지만 조금 다르다. 코틀린은 `suspend` 함수를 호출하기 위해 어떤 키워드도 필요하지 않다. 코틀린의 `suspend`가 다른 언어에서 `async`와 같다고 보면 된다.

## **`aync` 게으르게 사용하기**

`async`키워드를 사용하는 순간 코드 블록이 수행을 준비하는데, `async(start = CoroutineStart.LAZY)`로 인자를 전달하면 원하는 순간 수행을 준비하게 할 수 있다. 이후 `start`메서드를 이용해 수행을 준비하게 할 수 있다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

fun main() = runBlocking {
    val elapsedTime = measureTimeMillis {
        val value1 = async(start = CoroutineStart.LAZY) { getRandom1() }
        val value2 = async(start = CoroutineStart.LAZY) { getRandom2() }

        value1.start()
        value2.start()

        println("${value1.await()} + ${value2.await()} = ${value1.await() + value2.await()}")
    }
    println(elapsedTime)
}
```

```kotlin
337 + 46 = 383
1132
```

## **async를 사용한 구조적인 동시성**

코드를 수행하다 보면 예외가 발생할 수 있다. 예외가 발생하면 위쪽의 코루틴 스코프와 아래쪽의 코루틴 스코프가 취소된다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun getRandom1(): Int {
    try {
        delay(1000L)
        return Random.nextInt(0, 500)
    } finally {
        println("getRandom1 is cancelled.")
    }
}

suspend fun getRandom2(): Int {
    delay(500L)
    throw IllegalStateException()
}

suspend fun doSomething() = coroutineScope {
    val value1 = async { getRandom1() }
    val value2 = async { getRandom2() }
    try {
        println("${value1.await()} + ${value2.await()} = ${value1.await() + value2.await()}")
    } finally {
        println("doSomething is cancelled.")
    }
}

fun main() = runBlocking {
    try {
        doSomething()
    } catch (e: IllegalStateException) {
        println("doSomething failed: $e")
    }
}
```

```kotlin
getRandom1 is cancelled.
doSomething is cancelled.
doSomething failed: java.lang.IllegalStateException
```

`getRandom2`가 오류가 나서 `getRandom1`와 `doSomething`은 취소된다. (`JobCancellationException`발생) 문제가 된 `IllegalStateException`도 외부에서 잡아줘야 한다.
