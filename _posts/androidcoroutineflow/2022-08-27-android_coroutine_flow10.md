---
title : "[Coroutine&Flow] 10장 플로우 연산"
excerpt: "플로우 연산"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-08-27
---

![coroutine10_image1.jpg](/assets/images/coroutine10_image1.jpg?raw=true)

## **플로우와 map**

플로우에서 `map`연산을 통해 데이터를 가공할 수 있다.

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
    flowSomething().map {
        "$it $it"
    }.collect { value ->
        println(value)
    }
}
```

```kotlin
469 469
414 414
347 347
420 420
33 33
28 28
398 398
425 425
404 404
393 393
```

## **플로우와 filter**

`filter`기능을 사용할 수 있다. 이 기능을 이용해 짝수만 남겨보자.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    (1..20).asFlow().filter {
        (it % 2) == 0
    }.collect {
        println(it)
    }
}
```

```kotlin
2
4
6
8
10
12
14
16
18
20
```

## **filterNot**

만약 홀수만 남기고 싶을 때 술어(predicate)를 수정할 수 도 있다. 하지만 술어를 그대로 두고 `filterNot`을 사용할 수도 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    (1..20).asFlow().filterNot {
        (it % 2) == 0
    }.collect {
        println(it)
    }
}
```

```kotlin
1
3
5
7
9
11
13
15
17
19
```

## **transform 연산자**

`transform` 연산자를 이용해 조금 더 유연하게 스트림을 변형할 수 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.collect {
        println(it)
    }
}
```

```kotlin
1
2
2
4
3
6
4
8
5
10
6
12
7
14
8
16
9
18
10
20
11
22
12
24
...
```

중간 연산자(map)은 요소마다 1개의 변환밖에 하지 못하지만

변환 연산자(transform)은 예시처럼 emit()을 추가하여 요소마다 여러개의 변환이 가능하게 해준다.

## **take 연산자**

`take` 연산자는 몇개의 수행 결과만 얻는다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.take(5)
    .collect {
        println(it)
    }
}
```

```kotlin
1
2
2
4
3
```

## **takeWhile 연산자**

`takeWhile`을 이용해 조건을 만족하는 동안만 값을 가져오게 할 수 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.takeWhile {
        it < 15
    }.collect {
        println(it)
    }
}
```

```kotlin
1
2
2
4
3
6
4
8
5
10
6
12
7
14
8
```

## **drop 연산자**

`drop`연산자는 처음 몇개의 결과를 버린다. `take`가 `takeWhile`을 가지듯 `dropWhile`도 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    (1..20).asFlow().transform {
        emit(it)
        emit(someCalc(it))
    }.drop(5)
    .collect {
        println(it)
    }
}
```

```kotlin
6
4
8
5
10
6
12
7
14
8
16
9
18
10
20
11
22
12
24
13
...
```

## **reduce 연산자**

`collect`, `reduce`, `fold`, `toList`, `toSet`과 같은 연산자는 플로우를 끝내는 함수라 종단 연산자(terminal operator)라고 한다.

`reduce`는 흔히 `map`과 `reduce`로 함께 소개되는 함수형 언어의 오래된 메커니즘이다. 첫번째 값을 결과에 넣은 후 각 값을 가져와 누진적으로 계산한다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    val value = (1..10)
        .asFlow()
        .reduce { a, b ->
            a + b
        }
    println(value)
}
```

```kotlin
55
```

## **fold 연산자**

`fold` 연산자는 `reduce`와 매우 유사하다. 초기값이 있다는 차이만 있다.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun someCalc(i: Int): Int {
    delay(10L)
    return i * 2
}

fun main() = runBlocking<Unit> {
    val value = (1..10)
        .asFlow()
        .fold(10) { a, b ->
            a + b
        }
    println(value)
}
```

```kotlin
65
```

## **count 연산자**

`count`의 연산자는 술어를 만족하는 자료의 갯수를 센다. 짝수의 갯수를 세어보자.

```kotlin
import kotlin.random.Random
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
    val counter = (1..10)
        .asFlow()
        .count {
            (it % 2) == 0
        }
    println(counter)
}
```

```kotlin
5
```
