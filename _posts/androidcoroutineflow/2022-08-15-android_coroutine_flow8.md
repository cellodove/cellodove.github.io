---
title : "[Coroutine&Flow] 08장 공유 객체, Mutex, Actor"
excerpt: "공유 객체, Mutex, Actor"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-08-15
---

![coroutine8_image1.jpg](/assets/images/coroutine8_image1.jpg?raw=true)

## **공유 객체 문제**

```kotlin
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

```kotlin
31 ms동안 100000개의 액션을 수행했습니다.
Counter = 97405
```

`withContext`는 수행이 완료될 때 까지 기다리는 코루틴 빌더이다. 뒤의 `println("Counter = $counter")` 부분은 잠이 들었다. `withContext` 블록의 코드가 모두 수행되면 깨어나 호출된다.

위의 코드는 불행히도 항상 `10000`이 되는 것은 아니다. `Dispatchers.Default`에 의해 코루틴이 어떻게 할당되냐에 따라 값이 달라진다.

## **volatile을 적용하기**

손 쉽게 생각할 수 있는 방법은 `volatile`를 사용하는 것이다.

```kotlin
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis { 
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

@Volatile // 코틀린에서 어노테이션입니다.
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

```kotlin
54 ms동안 100000개의 액션을 수행했습니다.
Counter = 95338
```

`volatile`은 가시성 문제만을 해결할 뿐 동시에 읽고 수정해서 생기는 문제를 해결하지 못한다.

## **스레드 안전한 자료구조 사용하기**

`AtomicInteger`와 같은 스레드 안전한 자료구조를 사용하는 방법이 있다.

```kotlin
import java.util.concurrent.atomic.*
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

val counter = AtomicInteger()

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.incrementAndGet()
        }
    }
    println("Counter = $counter")
}
```

`AtomicInteger`가 이 문제에는 적합한데 항상 정답은 아니다.

## 스레드 한정

`newSingleThreadContext`를 이용해서 특정한 스레드를 만들고 해당 스레드를 사용할 수 있다.

```kotlin
import java.util.concurrent.atomic.*
import kotlin.system.*
import kotlinx.coroutines.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

var counter = 0
val counterContext = newSingleThreadContext("CounterContext")

fun main() = runBlocking {
    withContext(counterContext) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
```

## **뮤텍스**

뮤텍스는 상호배제(Mutual exclusion)의 줄임말이다.

공유 상태를 수정할 때 임계 영역(critical section)를 이용하게 하며, 임계 영역을 동시에 접근하는 것을 허용하지 않는다.

```kotlin
import kotlin.system.*
import kotlinx.coroutines.*
import kotlinx.coroutines.sync.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100 // 시작할 코루틴의 갯수
    val k = 1000 // 코루틴 내에서 반복할 횟수
    val elapsed = measureTimeMillis {
        coroutineScope { // scope for coroutines
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")
}

val mutex = Mutex()
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            mutex.withLock {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
```

## **액터**

액터는 1973년에 칼 휴이트가 만든 개념으로 액터가 독점적으로 자료를 가지며 그 자료를 다른 코루틴과 공유하지 않고 액터를 통해서만 접근하게 만든다.

먼저 실드 클래스를 만들어서 시작합니다.

```kotlin
sealed class CounterMsg
object IncCounter : CounterMsg()
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg()
```

1. 실드(sealed) 클래스는 외부에서 확장이 불가능한 클래스이다. `CounterMsg`는 `IncCounter`와 `GetCounter` 두 종류로 한정된다.
2. `IncCounter`는 싱글톤으로 인스턴스를 만들 수 없다. 액터에게 값을 증가시키기 위한 신호로 쓰인다.
3. `GetCounter`는 값을 가져올 때 쓰며 `CompletableDeferred<Int>`를 이용해 값을 받아온다.

```kotlin
fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0 // 액터 안에 상태를 캡슐화해두고 다른 코루틴이 접근하지 못하게 합니다.

    for (msg in channel) { // 외부에서 보내는 것은 채널을 통해서만 받을 수 있습니다.(recieve)
        when (msg) {
            is IncCounter -> counter++ // 증가시키는 신호.
            is GetCounter -> msg.response.complete(counter) // 현재 상태를 반환합니다.
        }
    }
}
```

채널은 송신 측에서 값을 `send`할 수 있고 수신 측에서 `receive`를 할 수 있는 도구이다.

전체 코드는 다음과 같다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100
    val k = 1000
    val elapsed = measureTimeMillis {
        coroutineScope {
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("$elapsed ms동안 ${n * k}개의 액션을 수행했습니다.")  
}

sealed class CounterMsg
object IncCounter : CounterMsg()
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg()

fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0
    for (msg in channel) {
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}

fun main() = runBlocking<Unit> {
    val counter = counterActor()
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.send(IncCounter)
        }
    }

    val response = CompletableDeferred<Int>()
    counter.send(GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close()
}
```
