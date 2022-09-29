---
title : "[Coroutine&Flow] 13장 플로우 결합"
excerpt: "플로우 결합"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-09-28
---

![coroutine13_image1.jpg](/assets/images/coroutine13_image1.jpg?raw=true)

## **zip으로 묶기**

`zip`은 양쪽의 데이터를 한꺼번에 묶어 새로운 데이터를 만들어 낸다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow()
    val strs = flowOf("일", "이", "삼") 
    nums.zip(strs) { a, b -> "${a}은(는) $b" }
        .collect { println(it) }
}
```

```kotlin
1은(는) 일
2은(는) 이
3은(는) 삼
```

## **combine으로 묶기**

`combine`은 양쪽의 데이터를 같은 시점에 묶지 않고 한 쪽이 갱신되면 새로 묶어 데이터를 만든다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> { 
    val nums = (1..3).asFlow().onEach { delay(100L) }
    val strs = flowOf("일", "이", "삼").onEach { delay(200L) }
    nums.combine(strs) { a, b -> "${a}은(는) $b" }
        .collect { println(it) }
}
```

```kotlin
2은(는) 일
3은(는) 일
3은(는) 이
3은(는) 삼
```

데이터가 짝을 이룰 필요없이 최신의 데이터를 이용해 가공해야 하는 경우에 사용할 수 있다.
