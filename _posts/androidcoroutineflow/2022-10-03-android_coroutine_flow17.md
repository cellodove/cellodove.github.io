---
title : "[Coroutine&Flow] 17장 플로우 런칭"
excerpt: "플로우 런칭"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-03
---

![coroutine17_image1.jpg](/assets/images/coroutine17_image1.jpg?raw=true)

## **이벤트를 Flow로 처리하기**

`addEventListener` 대신 플로우의 `onEach`를 사용할 수 있다. 이벤트마다 `onEach`가 대응한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect()
    println("Done")
}
```

```kotlin
Event: 1
Event: 2
Event: 3
Done
```

하지만 `collect`가 플로가 끝날 때 까지 기다리는 것이 문제이다.

## **launchIn을 사용하여 런칭하기**

`launchIn`을 이용하면 별도의 코루틴에서 플로우를 런칭할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this)
    println("Done")
}
```

```kotlin
Done
Event: 1
Event: 2
Event: 3
```
