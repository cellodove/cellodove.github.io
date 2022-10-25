---
title : "[Coroutine&Flow] 20장 팬 아웃, 팬 인"
excerpt: "채널 팬 아웃, 팬 인"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-24
---

![coroutine20_image1.jpg.jpg](/assets/images/coroutine20_image1.jpg.jpg?raw=true)

## **팬 아웃**

여러 코루틴이 동시에 채널을 구독할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
        delay(100L)
    }
}

fun CoroutineScope.processNumber(id: Int, channel: ReceiveChannel<Int>) = launch {
    channel.consumeEach {
        println("${id}가 ${it}을 받았습니다.")
    }
}

fun main() = runBlocking<Unit> {
    val producer = produceNumbers()
    repeat (5) {
        processNumber(it, producer)
    }
    delay(1000L)
    producer.cancel()
}
```

```kotlin
0가 1을 받았습니다.
0가 2을 받았습니다.
1가 3을 받았습니다.
2가 4을 받았습니다.
3가 5을 받았습니다.
4가 6을 받았습니다.
0가 7을 받았습니다.
1가 8을 받았습니다.
2가 9을 받았습니다.
3가 10을 받았습니다.
```

## **팬 인**

팬 인은 반대로 생산자가 많은 것이다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

suspend fun produceNumbers(channel: SendChannel<Int>, from: Int, interval: Long) {
    var x = from
    while (true) {
        channel.send(x)
        x += 2
        delay(interval)
    }
}

fun CoroutineScope.processNumber(channel: ReceiveChannel<Int>) = launch {
    channel.consumeEach {
        println("${it}을 받았습니다.")
    }
}

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        produceNumbers(channel, 1, 100L)
    }
    launch {
        produceNumbers(channel, 2, 150L)
    }
    processNumber(channel)
    delay(1000L)
    coroutineContext.cancelChildren()
}
```

```kotlin
1을 받았습니다.
2을 받았습니다.
3을 받았습니다.
4을 받았습니다.
5을 받았습니다.
6을 받았습니다.
7을 받았습니다.
9을 받았습니다.
8을 받았습니다.
11을 받았습니다.
10을 받았습니다.
13을 받았습니다.
15을 받았습니다.
12을 받았습니다.
17을 받았습니다.
14을 받았습니다.
19을 받았습니다.
```

## **공정한 채널**

두 개의 코루틴에서 채널을 서로 사용할 때 공정하게 기회를 준다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

suspend fun someone(channel: Channel<String>, name: String) {
    for (comment in channel) {
        println("${name}: ${comment}")
        channel.send(comment.drop(1) + comment.first())
        delay(100L)
    }
}

fun main() = runBlocking<Unit> {
    val channel = Channel<String>()
    launch {
        someone(channel, "진수")
    }
    launch {
        someone(channel, "하나")
    }
    channel.send("안녕 하세요")
    delay(1000L)
    coroutineContext.cancelChildren()
}
```

```kotlin
진수: 안녕 하세요
하나: 녕 하세요안
진수:  하세요안녕
하나: 하세요안녕 
진수: 세요안녕 하
하나: 요안녕 하세
진수: 안녕 하세요
하나: 녕 하세요안
진수:  하세요안녕
하나: 하세요안녕 
진수: 세요안녕 하
```

## **select**

먼저 끝나는 요청을 처리하는 것이 중요할 수 있다. 이 경우에 `select`를 사용한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.selects.*

fun CoroutineScope.sayFast() = produce<String> {
    while (true) {
        delay(100L)
        send("패스트")
    }
}

fun CoroutineScope.sayCampus() = produce<String> {
    while (true) {
        delay(150L)
        send("캠퍼스")
    }
}

fun main() = runBlocking<Unit> {
    val fasts = sayFast()
    val campuses = sayCampus()
    repeat (5) {
        select<Unit> {
            fasts.onReceive {
                println("fast: $it")
            }
            campuses.onReceive {
                println("campus: $it")
            }
        }
    }
    coroutineContext.cancelChildren()
}
```

```kotlin
fast: 패스트
campus: 캠퍼스
fast: 패스트
fast: 패스트
campus: 캠퍼스
```

채널에 대해 `onReceive`를 사용하는 것 이외에도 아래의 상황에서 사용할 수 있다.

- `Job` - `onJoin`
- `Deferred` - `onAwait`
- `SendChannel` - `onSend`
- `ReceiveChannel` - `onReceive`, `onReceiveCatching`
- `delay` - `onTimeout`
