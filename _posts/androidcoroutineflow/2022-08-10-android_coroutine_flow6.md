---
title : "[Coroutine&Flow] 06장 코루틴 컨텍스트와 디스패처"
excerpt: "코루틴 컨텍스트와 디스패처"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-08-10
---

![coroutine6_image1.jpg](/assets/images/coroutine6_image1.jpg?raw=true)

## 코루틴 디스패처

코루틴은 여러 디스패처를 가지고있다. 코루틴을 만든 다음 해당 코루틴을 Dispatcher에 전송하면 Dispatcher은 자신이 관리하는 스레드풀 내의 스레드의 부하 상황에 맞춰 코루틴을 배분한다.

디스패처 에는 `Default`, `IO`, `Unconfined`, `newSingleThreadContext` 가 있다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch {
        println("부모의 콘텍스트 / ${Thread.currentThread().name}")
    }

    launch(Dispatchers.Default) {
        println("Default / ${Thread.currentThread().name}")
    }

    launch(Dispatchers.IO) {
        println("IO / ${Thread.currentThread().name}")
    }

    launch(Dispatchers.Unconfined) {
        println("Unconfined / ${Thread.currentThread().name}")
    }

    launch(newSingleThreadContext("Fast Campus")) {
        println("newSingleThreadContext / ${Thread.currentThread().name}")
    }
}
```

```kotlin
Unconfined / main @coroutine#5
IO / DefaultDispatcher-worker-2 @coroutine#4
Default / DefaultDispatcher-worker-1 @coroutine#3
부모의 콘텍스트 / main @coroutine#2
newSingleThreadContext / Fast Campus @coroutine#6
```

1. `Default`는 코어 수에 비례하는 스레드 풀에서 수행한다.
2. `IO`는 CPU를 덜 소모하기 때문에 코어 수 보다 훨씬 많은 스레드를 가지는 스레드 풀이다. 파일입출력이나 서버통신에 보통 사용한다.
3. `Unconfined`는 어디에도 속하지 않는다. 수행되는 스레드가 그때마다 달라진다. 지금 시점에는 부모의 스레드에서 수행될 것이다.
4. `newSingleThreadContext`는 항상 새로운 스레드를 만든다.

`Default` 와 `IO` 는 같은 워커를 사용한다 정책이 다른것이다.

## **async에서 코루틴 디스패처 사용**

`launch`외에 `async`, `withContext` 등의 코루틴 빌더에도 디스패처를 사용할 수 있다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    async {
        println("부모의 콘텍스트 / ${Thread.currentThread().name}")
    }

    async(Dispatchers.Default) {
        println("Default / ${Thread.currentThread().name}")
    }

    async(Dispatchers.IO) {
        println("IO / ${Thread.currentThread().name}")
    }

    async(Dispatchers.Unconfined) {
        println("Unconfined / ${Thread.currentThread().name}")
    }

    async(newSingleThreadContext("Fast Campus")) {
        println("newSingleThreadContext / ${Thread.currentThread().name}")
    }
}
```

```kotlin
Unconfined / main @coroutine#5
Default / DefaultDispatcher-worker-1 @coroutine#3
IO / DefaultDispatcher-worker-2 @coroutine#4
newSingleThreadContext / Fast Campus @coroutine#6
부모의 콘텍스트 / main @coroutine#2
```

## **Confined 디스패처 테스트**

`Confined`는 처음에는 부모의 스레드에서 수행된다. 하지만 한번 중단점(suspension point)에 오면 바뀌게 된다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    async(Dispatchers.Unconfined) {
        println("Unconfined / ${Thread.currentThread().name}")
        delay(1000L)
        println("Unconfined / ${Thread.currentThread().name}")
    }
}
```

```kotlin
Unconfined / main @coroutine#2
Unconfined / kotlinx.coroutines.DefaultExecutor @coroutine#2
```

`Confined`는 중단점 이후 어느 디스패처에서 수행될지 예측하기 어렵다. 가능하면 확실한 디스패처를 사용하는게 좋을것같다.

## **부모가 있는 Job과 없는 Job**

코루틴 스코프, 코루틴 컨텍스트는 구조화되어 있고 부모에게 계층적으로 되어 있다. 코루틴 컨텍스트의 `Job`역시 부모에게 의존적이다. 부모를 캔슬했을 때의 영향을 확인해보자.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {//증부모
    val job = launch {//부모
        //자식이였던...
        launch(Job()) {
            println(coroutineContext[Job])
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }

        launch {//자식
            println(coroutineContext[Job])
            println("launch2: ${Thread.currentThread().name}")
            delay(1000L)
            println("1!")
        }
    }

    delay(500L)
    job.cancelAndJoin()
    delay(1000L)
}
```

```kotlin
"coroutine#3":StandaloneCoroutine{Active}@b684286
launch1: main @coroutine#3
"coroutine#4":StandaloneCoroutine{Active}@36d64342
launch2: main @coroutine#4
3!
```

Job을 가지면 자식이 아니다. 그렇기에 부모를 취소해도 별개로 동작한다.

## 부모의 마음

구조화되어 계층화된 코루틴은 자식들의 실행을 지켜볼것인가?

```kotlin
import kotlin.system.*
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    val elapsed = measureTimeMillis {
        val job = launch { // 부모
            launch { // 자식 1
                println("launch1: ${Thread.currentThread().name}")
                delay(5000L)
            }

            launch { // 자식 2
                println("launch2: ${Thread.currentThread().name}")
                delay(10L)
            }
        }
        job.join()
    }
    println(elapsed)
}
```

```kotlin
launch1: main @coroutine#3
launch2: main @coroutine#4
5022
```

부모를 `join`해서 기다려 보면 부모는 두 자식이 모두 끝날 때까지 기다린다는 것을 알 수 있다.

첫번째`launch` 를 `launch(Job())` 으로만들면 더이상 자식이 아니기때문에 기다리지않아 빨리 결과가 나온다.

## **코루틴 엘리먼트 결합**

여러 코루틴 엘리먼트를 한번에 사용할 수 있다. `+`연산으로 엘리먼트를 합치면 된다. 합쳐진 엘리먼트들은 `coroutineContext[XXX]`로 조회할 수 있다.

```kotlin
import kotlin.system.*
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    launch {
        launch(Dispatchers.IO + CoroutineName("launch1")) {
            println("launch1: ${Thread.currentThread().name}")
            println(coroutineContext[CoroutineDispatcher])
            println(coroutineContext[CoroutineName])
            delay(5000L)
        }

        launch(Dispatchers.Default + CoroutineName("launch1")) {
            println("launch2: ${Thread.currentThread().name}")
            println(coroutineContext[CoroutineDispatcher])
            println(coroutineContext[CoroutineName])
            delay(10L)
        }
    }
}
```

`launch(Dispatchers.IO + CoroutineName("launch1"))` 디스패처를 지정하고 코루틴에 이름를 지정할 수 있다. 저 두개를 합처 코루틴 컨텍스트라 부른다.
