---
title : "[Coroutine&Flow] 16장 플로우 완료처리"
excerpt: "플로우 완료처리"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-02
---

![coroutine16_image1.jpg](/assets/images/coroutine16_image1.jpg?raw=true)

## **명령형 finally 블록**

완료를 처리하는 방법 중의 하나는 명령형의 방식으로 `finally` 블록을 이용하는 것이다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}
```

```kotlin
1
2
3
Done
```

## **선언적으로 완료 처리하기**

`onCompletion`연산자를 선언해서 완료를 처리할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { println("Done") }
        .collect { value -> println(value) }
}
```

```kotlin
1
2
3
Done
```

## **onCompletion의 장점**

`onCompletion`은 종료 처리를 할 때 예외가 발생되었는지 여부를 알 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}
```

```kotlin
1
Flow completed exceptionally
Caught exception
```
