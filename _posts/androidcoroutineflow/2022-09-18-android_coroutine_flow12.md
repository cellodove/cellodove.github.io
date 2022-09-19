---
title : "[Coroutine&Flow] 12장 플로우 버퍼링"
excerpt: "플로우 버퍼링"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-09-18
---

![coroutine12_image1.jpg](/img/coroutine12_image1.jpg?raw=true)

## **버퍼가 없는 플로우**

보내는 쪽과 받는 쪽이 모두 바쁘다고 가정해보자

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300)
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}
```

```kotlin
1
2
3
Collected in 1245 ms
```

딜레이가 긴쪽을 기다려 동작하는것을 볼 수 있다.

## **buffer**

`buffer`로 버퍼를 추가해 보내는 측이 더 이상 기다리지 않게 할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().buffer()
            .collect { value -> 
                delay(300)
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}
```

```kotlin
1
2
3
Collected in 1178 ms 
```

## **conflate**

`conflate`를 이용하면 중간의 값을 융합(conflate)할 수 있다. 처리보다 빨리 발생한 데이터의 중간 값들을 누락합니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().conflate()
            .collect { value -> 
                delay(300)
                println(value) 
            } 
    }   
    println("Collected in $time ms")
}
```

```kotlin
1
3
Collected in 753 ms
```

## **마지막 값만 처리하기**

`conflate`와 같이 방출되는 값을 누락할 수도 있지만 수집 측이 느릴 경우 새로운 데이터가 있을 때 수집 측을 종료시키고 새로 시작하는 방법도 있습니다.

`collectLatest`를 사용한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collectLatest { value -> 
            println("값 ${value}를 처리하기 시작합니다.")
            delay(300)
            println(value) 
            println("처리를 완료하였습니다.")
        } 
    }   
    println("Collected in $time ms")
}
```

```kotlin
값 1를 처리하기 시작합니다.
값 2를 처리하기 시작합니다.
값 3를 처리하기 시작합니다.
3
처리를 완료하였습니다.
Collected in 702 ms
```
