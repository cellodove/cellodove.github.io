---
title : "[Coroutine&Flow] 14장 플로우 플래트"
excerpt: "플로우 플래트"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-09-29
---

![coroutine14_image1.jpg](/assets/images/coroutine14_image1.jpg?raw=true)

## **flatMapConcat**

플로우에서는 3가지 유형의 `flatmap`을 자원하고 있다. `flatMapConcat`, `flatMapMerge`, `flatMapLatest`이다.

`flatMapConcat` 은 첫번째 요소에 대해서 플레트닝을 하고 두번째 요소를 합한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapConcat {
            requestFlow(it)
        }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

```kotlin
1: First at 135 ms from start
1: Second at 637 ms from start
2: First at 738 ms from start
2: Second at 1238 ms from start
3: First at 1339 ms from start
3: Second at 1839 ms from start
```

## **flatMapMerge**

`flatMapMerge`는 첫 요소의 플레트닝을 시작하며 이어 다음 요소의 플레트닝을 시작한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapMerge {
             requestFlow(it) 
        }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

```kotlin
1: First at 170 ms from start
2: First at 266 ms from start
3: First at 367 ms from start
1: Second at 671 ms from start
2: Second at 766 ms from start
3: Second at 869 ms from start
```

## **flatMapLatest**

`flatMapLatest`는 다음 요소의 플레트닝을 시작하며 이전에 진행 중이던 플레트닝을 취소한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapLatest {
            requestFlow(it) 
        }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
}
```

```kotlin
1: First at 145 ms from start
2: First at 281 ms from start
3: First at 382 ms from start
3: Second at 883 ms from start
```
