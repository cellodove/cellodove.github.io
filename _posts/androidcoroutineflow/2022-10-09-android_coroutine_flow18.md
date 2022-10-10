---
title : "[Coroutine&Flow] 18장 채널 기초"
excerpt: "채널 기초"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-09
---

![coroutine18_image1.jpg](/assets/images/coroutine18_image1.jpg?raw=true)

## 채널

채널은 일종의 파이프이다. 송신측에서 채널에 `send`로 데이터를 전달하고 수신 측에서 채널을 통해 `receive` 받는다. (`trySend`와 `tryReceive`도 있다. 과거에는 `null`을 반환하는 `offer`와 `poll`가 있었다.)

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..10) {
            channel.send(x)
        }
    }

    repeat(10) {
            println(channel.receive())
    }
    println("완료")
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
완료
```

## **같은 코루틴에서 채널을 읽고 쓸경우**

`send`나 `receive`가 suspension point이고 서로에게 의존적이기 때문에 같은 코루틴에서 사용하는 것은 위험할 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..10) {
            channel.send(x)
        }

        repeat(10) {
            println(channel.receive())
        }
        println("완료")
    }
}
```

```kotlin
Evaluation stopped while it's taking too long️
```

무기한 대기하는 것을 볼 수 있다.

## **채널 close**

채널에서 더 이상 보낼 자료가 없으면 `close` 메서드를 이용해 채널을 닫을 수 있다. 채널은 for in 을 이용해서 반복적으로 `receive`할 수 있고 `close`되면 for in은 자동으로 종료된다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..10) {
            channel.send(x)
        }
        channel.close()
    }

    for (x in channel) {
        println(x)
    }
    println("완료")
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
완료
```

## **채널 프로듀서**

생산자(producer)와 소비자(consumer)는 굉장히 일반적인 패턴이다. 채널을 이용해서 한 쪽에서 데이터를 만들고 다른 쪽에서 받는 것을 도와주는 확장 함수들이 있다.

1. `produce`: 코루틴을 만들고 채널을 재공한다.
2. `consumeEach`: 채널에서 반복해서 데이터를 제공한다.

`ProducerScope`는 `CoroutineScope` 인터페이스와 `SendChannel` 인터페이스를 함께 상속받는다. 그래서 코루틴 컨텍스트와 몇가지 채널 인터페이스를 같이 사용할 수 있는 특이한 스코프이다.

## 참고

우리가 흔히 쓰는 `runBlocking`은 `BlockingCoroutine`을 쓰는데 이는 `AbstractCoroutine`를 상속받고 있다.

결국 코루틴 빌더는 코루틴을 만드는데 이들이 코루틴 스코프이기도 하다.

`AbstractCoroutine`은 `JobSupport`, `Job`(인터페이스), `Continuation`(인터페이스), `CoroutineScope`(인터페이스)을 상속받고 있다.

`Continuation`은 다음에 무엇을 할지, `Job`은 제어를 위한 정보와 제어, `CoroutineScope`는 컨텍스트 제공의 역할을 한다. `JobSupport`는 `Job`의 실무(?)를 한다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val oneToTen = produce {
        for (x in 1..10) {
            channel.send(x)
        }
    }

    oneToTen.consumeEach {
        println(it)
    }
    println("완료")
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
완료
```
