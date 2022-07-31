---
title : "[Coroutine&Flow] 04장 취소와 타임아웃"
excerpt: "취소와 타임아웃"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-07-30
---

![coroutine4_image1.jpg.jpg](/assets/images/coroutine4_image1.jpg.jpg?raw=true)

## Job에 대해 취소

명시적인 `Job`에 대해 `cancel`메서드를 호출하여 취소할 수 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job1 = launch {
        println("launch1: ${Thread.currentThread().name}")
        delay(1000L)
        println("3!")
    }

    val job2 = launch {
        println("launch2: ${Thread.currentThread().name}")
        println("1!")
    }

    val job3 = launch {
        println("launch3: ${Thread.currentThread().name}")
        delay(500L)
        println("2!")  
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
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
launch2: main @coroutine#3
1!
launch3: main @coroutine#4
2!
4!
runBlocking: main @coroutine#1
5!
```

## 취소 불가능한 Job

`launch(Dispatchers.Default)`는 그 다음 코드 블록을 다른 스레드에서 수행을 시킨다.

```kotlin
import kotlinx.coroutines.*

suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancel()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}
```

```kotlin
1
2
doCount Done!
3
4
5
6
7
8
9
10
```

결과를 보면 몇가지 문제가 있다.

분명 취소 메소드를 사용했으나 취소가 되지않았다. 그리고 카운트가 마친후에 “doCount Done!”가 출력되길 원했지만 중간에 출력되었다. 이 두가지를 해결해 보자.

## cancel과 join

```kotlin
import kotlinx.coroutines.*

suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancel()
    job1.join()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}
```

```kotlin
1
2
3
4
5
6
7
8
9
10
doCount Done!
```

`join`을 넣어서 해당 코루틴이 끝날때까지 기다린 후 “doCount Done!”이 출력되게 할 수 있다.

## cancelAndJoin

`cancel`을 하고 `join`을 하는 일은 자주 일어나는 일기다. 그래서 한번에 처리할 수 있는 `cancelAndJoin`메소드가 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancelAndJoin()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}
```

```kotlin
1
2
3
4
5
6
7
8
9
10
doCount Done!
```

## cancel가능한 코루틴

`isActive`를 호출하면 해당 코루틴이 여전히 활성화된지 확인할 수 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10 && isActive) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
    
    delay(200L)
    job1.cancelAndJoin()
    println("doCount Done!")
}

fun main() = runBlocking {
    doCount()
}
```

```kotlin
1
2
doCount Done!
```

while문에서 isActive 조건을 추가해 코루틴이 종료되면 while문도 종료되게 한다.

## finally를 같이 사용

`launch`에 자원을 할당한 경우에는 어떻게 해야할까? 예를 들어 파일 입출력이나, 소켓등이 있다.

`suspend`함수들은 `JobCancellationException`이 발생하기 때문에 표준 try catch finally로 대응할 수 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job1 = launch {
        try {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        } finally {
            println("job1 is finishing!")
        }
    }

    val job2 = launch {
        try {
            println("launch2: ${Thread.currentThread().name}")
            delay(1000L)
            println("1!")
        } finally {
            println("job2 is finishing!")
        }
    }

    val job3 = launch {
        try {
            println("launch3: ${Thread.currentThread().name}")
            delay(1000L)
            println("2!")
        } finally {
            println("job3 is finishing!")
        }
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
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
launch2: main @coroutine#3
launch3: main @coroutine#4
4!
job1 is finishing!
job2 is finishing!
job3 is finishing!
runBlocking: main @coroutine#1
5!
```

## 취소 불가능한 블록

어떤 코드는 취소가 불가능해야 한다.

`withContext(NonCancellable)`을 이용하면 취소 불가능한 블록을 만들 수 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doOneTwoThree() = coroutineScope {
    val job1 = launch {
        withContext(NonCancellable) {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("3!")
        }
        delay(1000L)
        print("job1: end")
    }

    val job2 = launch {
        withContext(NonCancellable) {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("1!")
        }
        delay(1000L)
        print("job2: end")
    }

    val job3 = launch {
        withContext(NonCancellable) {
            println("launch1: ${Thread.currentThread().name}")
            delay(1000L)
            println("2!")
        }
        delay(1000L)
        print("job3: end")
    }

    delay(800L)
    job1.cancel()
    job2.cancel()
    job3.cancel()
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
launch1: main @coroutine#3
launch1: main @coroutine#4
4!
3!
1!
2!
runBlocking: main @coroutine#1
5!
```

취소 불가능한 코드를 `finally`절에 사용할 수도 있다.

cancel이 동작하여 finally절에 들어갈텐데 그안에 들어가서도 cancel이 이루어질 수 있다. 그렇기에 무조건 수행이 되어야한다면`withContext(NonCancellable)` 를 사용하면 된다.

## 타임 아웃

일정 시간이 끝난 후에 종료하고 싶다면 `withTimeout`을 이용할 수 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10 && isActive) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
}

fun main() = runBlocking {
    withTimeout(500L) {
        doCount()
    }
}
```

```kotlin
1
2
3
4
Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 500 ms
 at (Coroutine boundary. (:-1) 
 at FileKt$main$1$1.invokeSuspend (File.kt:-1) 
 at FileKt$main$1.invokeSuspend (File.kt:-1) 
Caused by: kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 500 ms
at kotlinx.coroutines.TimeoutKt .TimeoutCancellationException(Timeout.kt:184)
at kotlinx.coroutines.TimeoutCoroutine .run(Timeout.kt:154)
at kotlinx.coroutines.EventLoopImplBase$DelayedRunnableTask .run(EventLoop.common.kt:508)
```

취소가 되면 `TimeoutCancellationException`예외가 발생한다.

## **withTimeoutOrNull**

예외를 핸들하는 것은 귀찮은 일이다. `withTimeoutOrNull`을 이용해 타임 아웃할 때 `null`을 반환하게 할 수 있다.

```kotlin
import kotlinx.coroutines.*

suspend fun doCount() = coroutineScope {
    val job1 = launch(Dispatchers.Default) {
        var i = 1
        var nextTime = System.currentTimeMillis() + 100L

        while (i <= 10 && isActive) {
            val currentTime = System.currentTimeMillis()
            if (currentTime >= nextTime) {
                println(i)
                nextTime = currentTime + 100L
                i++
            }
        }
    }
}

fun main() = runBlocking {
    val result = withTimeoutOrNull(500L) {
        doCount()
        true
    } ?: false
    println(result)
}
```

```kotlin
1
2
3
4
false
```

성공할 경우 `whithTimeoutOrNull`의 마지막에서 `true`를 리턴하게 하고 실패했을 경우 `null`을 반환할테니 엘비스 연산자(`?:`)를 이용해 `false`를 리턴하게 했다. 엘비스 연산자는 `null`값인 경우에 다른 값으로 치환한다.