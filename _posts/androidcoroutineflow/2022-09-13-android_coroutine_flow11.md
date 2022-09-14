---
title : "[Coroutine&Flow] 11장 플로우 컨텍스트"
excerpt: "플로우 컨텍스트"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-09-13
---

![coroutine11_image1.jpg](/assets/images/coroutine11_image1.jpg?raw=true)

## **플로우는 코루틴 컨텍스트에서**

플로우는 현재 코루틴 컨텍스트에서 호출된다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    log("flow를 시작합니다.")
    for (i in 1..10) {
        emit(i)
    }
}  

fun main() = runBlocking<Unit> {
    launch(Dispatchers.IO) {
        simple()
            .collect { value -> log("${value}를 받음.") } 
    }
}
```

```kotlin
[DefaultDispatcher-worker-1 @coroutine#2] flow를 시작합니다.
[DefaultDispatcher-worker-1 @coroutine#2] 1를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 2를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 3를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 4를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 5를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 6를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 7를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 8를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 9를 받음.
[DefaultDispatcher-worker-1 @coroutine#2] 10를 받음.
```

## **다른 컨텍스트로 옮겨갈 수 없는 플로우**

플로우는 기본적으로 다른 컨텍스트로 옮길 수 없다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    withContext(Dispatchers.Default) {
        for (i in 1..10) {
            delay(100L)
            emit(i)
        }
    }
}  

fun main() = runBlocking<Unit> {
    launch(Dispatchers.IO) {
        simple()
            .collect { value -> log("${value}를 받음.") } 
    }
}
```

```kotlin
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
Flow was collected in [CoroutineId(2), "coroutine#2":StandaloneCoroutine{Active}@3cbec098, Dispatchers.IO],
but emission happened in [CoroutineId(2), "coroutine#2":DispatchedCoroutine{Active}@7b108e6a, Dispatchers.Default].
Please refer to 'flow' documentation or use 'flowOn' instead
 at kotlinx.coroutines.flow.internal.SafeCollector_commonKt.checkContext (SafeCollector.common.kt:85) 
 at kotlinx.coroutines.flow.internal.SafeCollector.checkContext (SafeCollector.kt:106) 
 at kotlinx.coroutines.flow.internal.SafeCollector.emit (SafeCollector.kt:83)
```

## **flowOn 연산자**

`flowOn` 연산자를 통해 컨텍스트를 올바르게 바꿀 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")
           
fun simple(): Flow<Int> = flow {
    for (i in 1..10) {
        delay(100L)
        log("값 ${i}를 emit합니다.")
        emit(i)
    }
}.flowOn(Dispatchers.Default)

fun main() = runBlocking<Unit> {
    simple().collect { value -> 
        log("${value}를 받음.")
    } 
}
```

```kotlin
[DefaultDispatcher-worker-2 @coroutine#2] 값 1를 emit합니다.
[main @coroutine#1] 1를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 2를 emit합니다.
[main @coroutine#1] 2를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 3를 emit합니다.
[main @coroutine#1] 3를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 4를 emit합니다.
[main @coroutine#1] 4를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 5를 emit합니다.
[main @coroutine#1] 5를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 6를 emit합니다.
[main @coroutine#1] 6를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 7를 emit합니다.
[main @coroutine#1] 7를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 8를 emit합니다.
[main @coroutine#1] 8를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 9를 emit합니다.
[main @coroutine#1] 9를 받음.
[DefaultDispatcher-worker-2 @coroutine#2] 값 10를 emit합니다.
[main @coroutine#1] 10를 받음.
```
