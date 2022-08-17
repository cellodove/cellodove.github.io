---
title : "[Coroutine&Flow] 09장 플로우 기초"
excerpt: "플로우 기초"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-08-16
---

![coroutine9_image1.jpg](/assets/images/coroutine9_image1.jpg?raw=true)

## **처음만나보는 플로우**

Flow는 코틀린에서 쓸 수 있는 비동기 스트림이다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(Random.nextInt(0, 500))
        delay(10L)
    }
}

fun main() = runBlocking {
    flowSomething().collect { value ->
        println(value)
    }
}
```

`flow`플로우 빌더 함수를 이용해서 코드블록을 구성하고 `emit`을 호출해서 스트림에 데이터를 흘려 보낸다.

플로우는 콜드 스트림이기 때문에 요청 측에서 `collect`를 호출해야 값을 발생하기 시작한다.

- 콜드 스트림 - 요청이 있는 경우에 보통 1:1로 값을 전달하기 시작.
- 핫 스트림 - 0개 이상의 상대를 향해 지속적으로 값을 전달.

## **플로우 취소**

코루틴 시간에 배웠던 `withTimeoutOrNull`을 이용해 간단히 취소할 수 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun flowSomething(): Flow<Int> = flow {
    repeat(10) {
        emit(Random.nextInt(0, 500))
        delay(100L)
    }
}

fun main() = runBlocking<Unit> {
    val result = withTimeoutOrNull(500L) {
        flowSomething().collect { value ->
            println(value)
        }
        true
    } ?: false
    if (!result) {
        println("취소되었습니다.")
    }
}
```

## **플로우 빌더 flowOf**

`flow` 이외에도 몇가지 `flowOf`, `asFlow`등의 플로우 빌더가 있다. 먼저 `flowOf`를 살펴보자.

`flowOf`는 여러 값을 인자로 전달해 플로우를 만든다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    flowOf(1, 2, 3, 4, 5).collect { value ->
        println(value)
    }
}
```

```kotlin
1
2
3
4
5
```

## **플로우 빌더 asFlow**

`asFlow`는 컬렉션이나 시퀀스를 전달해 플로우를 만들 수 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    listOf(1, 2, 3, 4, 5).asFlow().collect { value ->
        println(value)
    }
    (6..10).asFlow().collect {
        println(it)
    }
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
```
