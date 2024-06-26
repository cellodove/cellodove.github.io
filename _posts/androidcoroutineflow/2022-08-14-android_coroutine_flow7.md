---
title : "[Coroutine&Flow] 07장 CEH와 슈퍼 바이저 잡"
excerpt: "CEH와 슈퍼 바이저 잡"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-08-14
---

![coroutine7_image1.jpg](/assets/images/coroutine7_image1.jpg?raw=true)

## **GlobalScope**

어디에도 속하지 않지만 원래부터 존재하는 전역 `GlobalScope`가 있다. 이 전역 스코프를 이용하면 코루틴을 쉽게 수행할 수 있다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun printRandom() {
    delay(500L)
    println(Random.nextInt(0, 500))
}

fun main() {
    val job = GlobalScope.launch(Dispatchers.IO) {
        launch { printRandom() }
    }
    Thread.sleep(1000L)
}
```

`GlobalScope`는 어떤 계층에도 속하지 않고 영원히 동작하게 된다는 문제가 있다. 프로그래밍에서 전역 객체를 잘 사용하지 않는 것 처럼 `GlobalScope`도 잘 사용하지 않는다.

## **CoroutineScope**

`GlobalScope`보다 권장되는 형식은 `CoroutineScope`를 사용하는 것이다. `CoroutineScope`는 인자로 `CoroutineContext`를 받는데 코루틴 엘리먼트를 하나만 넣어도 좋고 이전에 배웠듯 엘리먼트를 합쳐 코루틴 컨텍스트를 만들어도 된다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun printRandom() {
    delay(500L)
    println(Random.nextInt(0, 500))
}

fun main() {
    val scope = CoroutineScope(Dispatchers.Default)
    val job = scope.launch(Dispatchers.IO) {
        launch { printRandom() }
    }
    Thread.sleep(1000L)
}
```

하나의 코루틴 엘리먼트, 디스패처 `Dispatchers.Default`만 넣어도 코루틴 컨텍스트가 만들어지기 때문에 이렇게 사용할 수 있다.

이제부터 `scope`로 계층적으로 형성된 코루틴을 관리할 수 있다. 우리의 필요에 따라 코루틴 스코프를 관리할 수 있다.

## **CEH (코루틴 익셉션 핸들러)**

예외를 가장 체계적으로 다루는 방법은 CEH (Coroutine Exception Handler, 코루틴 익셉션 핸들러)를 사용하는 것이다.

`CoroutineExceptionHandler`를 이용해서 우리만의 CEH를 만든 다음 상위 코루틴 빌더의 컨텍스트에 등록한다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun printRandom1() {
    delay(1000L)
    println(Random.nextInt(0, 500))
}

suspend fun printRandom2() {
    delay(500L)
    throw ArithmeticException()
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val scope = CoroutineScope(Dispatchers.IO)
    val job = scope.launch (ceh) {
        launch { printRandom1() }
        launch { printRandom2() }
    }
    job.join()
}
```

`CoroutineExceptionHandler`에 등록하는 람다에서 첫 인자는 `CoroutineContext` 두 번째 인자는 `Exception`이다. 대부분의 경우에는 `Exception`만 사용하고 나머지는 `_`로 남겨둔다.

## **runBlocking과 CEH**

`runBlocking`에서는 CEH를 사용할 수 없습니다. `runBlocking`은 자식이 예외로 종료되면 항상 종료되고 CEH를 호출하지 않습니다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun getRandom1(): Int {
    delay(1000L)
    return Random.nextInt(0, 500)
}

suspend fun getRandom2(): Int {
    delay(500L)
    throw ArithmeticException()
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val job = launch (ceh) {
        val a = async { getRandom1() }
        val b = async { getRandom2() }
        println(a.await())
        println(b.await())
    }
    job.join()
}
```

```kotlin
Exception in thread "main" java.lang.ArithmeticException
 at FileKt.getRandom2 (File.kt:12) 
 at FileKt$getRandom2$1.invokeSuspend (File.kt:-1) 
 at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith (ContinuationImpl.kt:33)
```

## **SupervisorJob**

슈퍼 바이저 잡은 예외에 의한 취소를 아래쪽으로 내려가게 한다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun printRandom1() {
    delay(1000L)
    println(Random.nextInt(0, 500))
}

suspend fun printRandom2() {
    delay(500L)
    throw ArithmeticException()
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val scope = CoroutineScope(Dispatchers.IO + SupervisorJob() + ceh)
    val job1 = scope.launch { printRandom1() }
    val job2 = scope.launch { printRandom2() }
    joinAll(job1, job2)
}
```

```kotlin
Something happend: java.lang.ArithmeticException
169
```

`printRandom2`가 실패했지만 `printRandom1`은 제대로 수행된다.

`joinAll`은 복수개의 `Job`에 대해 `join`를 수행하여 완전히 종료될 때까지 기다린다.

## **SupervisorScope**

코루틴 스코프와 슈퍼바이저 잡을 합친듯 한 SupervisorScope가 있다.

```kotlin
import kotlin.random.Random
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun printRandom1() {
    delay(1000L)
    println(Random.nextInt(0, 500))
}

suspend fun printRandom2() {
    delay(500L)
    throw ArithmeticException()
}

suspend fun supervisoredFunc() = supervisorScope {
    launch { printRandom1() }
    launch(ceh) { printRandom2() }
}

val ceh = CoroutineExceptionHandler { _, exception ->
    println("Something happend: $exception")
}

fun main() = runBlocking<Unit> {
    val scope = CoroutineScope(Dispatchers.IO)
    val job = scope.launch {
        supervisoredFunc()
    }
    job.join()
}
```

```kotlin
Something happend: java.lang.ArithmeticException
374
```

슈퍼바이저 스코프를 사용할 때 주의점은 무조건 자식 수준에서 예외를 핸들링 해야한다. 자식의 실패가 부모에게 전달되지 않기 때문에 자식 수준에서 예외를 처리해야한다.
