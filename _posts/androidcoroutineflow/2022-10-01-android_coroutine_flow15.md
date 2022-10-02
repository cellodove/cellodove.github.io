---
title : "[Coroutine&Flow] 15장 플로우 예외처리"
excerpt: "플로우 예외처리"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-01
---

![coroutine15_image1.jpg](/assets/images/coroutine15_image1.jpg?raw=true)

## **수집기 측에서 예외처리**

예외는 `collect`을 하는 수집기 측에서도 try-catch 식을 이용해 할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->         
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}
```

```kotlin
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

## **모든 예외는 처리가능**

어느 곳에서 발생한 예외라도 처리가 가능하다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}
```

```kotlin
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

## **예외 투명성**

빌더 코드 블록 내에서 예외를 처리하는 것은 예외 투명성을 어기는 것이다. 플로우에서는 `catch` 연산자를 이용하는 것을 권장한다.

`catch`블록에서 예외를 새로운 데이터로 만들어 `emit`을 하거나, 다시 예외를 던지거나, 로그를 남길 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> emit("Caught $e") } // emit on exception
        .collect { value -> println(value) }
}
```

```kotlin
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

## **catch 투명성**

`catch`연산자는 업스트림(catch 연산자를 쓰기 전의 코드)에만 영향을 미치고 다운스트림에는 영향을 미치지 않는다. 이를 catch 투명성이라고 한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}
```

```kotlin
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
 at FileKt$main$1$2.emit (File.kt:15) 
 at FileKt$main$1$2.emit (File.kt:14) 
 at kotlinx.coroutines.flow.FlowKt__ErrorsKt$catchImpl$2.emit (Errors.kt:158)
```
