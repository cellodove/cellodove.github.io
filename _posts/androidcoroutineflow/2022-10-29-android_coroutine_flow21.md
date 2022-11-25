---
title : "[Coroutine&Flow] 21장 채널 버퍼링"
excerpt: "채널 버퍼링"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-29
---

![coroutine21_image1.jpg.jpg](/assets/images/coroutine21_image1.jpg.jpg?raw=true)

## **버퍼**

이전에 만들었던 예제를 확장하여 버퍼를 지정해보자. `Channel` 생성자는 인자로 버퍼의 사이즈를 지정 받는다.

지정하지 않으면 버퍼를 생성하지 않는다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(10)
    launch {
        for (x in 1..20) {
            println("${x} 전송중")
            channel.send(x)
        }
        channel.close()
    }

    for (x in channel) {
        println("${x} 수신")
        delay(100L)
    }
    println("완료")
}
```

```kotlin
1 전송중
2 전송중
3 전송중
4 전송중
5 전송중
6 전송중
7 전송중
8 전송중
9 전송중
10 전송중
11 전송중
12 전송중
1 수신
2 수신
13 전송중
3 수신
14 전송중
4 수신
15 전송중
5 수신
16 전송중
6 수신
17 전송중
7 수신
18 전송중
8 수신
19 전송중
9 수신
20 전송중
10 수신
11 수신
12 수신
13 수신
14 수신
15 수신
16 수신
17 수신
18 수신
19 수신
20 수신
완료
```

채널에 인자로 `10`을 지정했다. 10개까지는 수신자가 받지 않아도 계속 전송한다.

## **랑데뷰**

버퍼 사이즈를 랑데뷰(Channel.RENDEZVOUS)로 지정해보자.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(Channel.RENDEZVOUS)
    launch {
        for (x in 1..20) {
            println("${x} 전송중")
            channel.send(x)
        }
        channel.close()
    }

    for (x in channel) {
        println("${x} 수신")
        delay(100L)
    }
    println("완료")
}
```

```kotlin
1 전송중
2 전송중
1 수신
2 수신
3 전송중
3 수신
4 전송중
4 수신
5 전송중
5 수신
6 전송중
6 수신
7 전송중
7 수신
8 전송중
8 수신
9 전송중
9 수신
10 전송중
10 수신
11 전송중
11 수신
12 전송중
12 수신
13 전송중
13 수신
14 전송중
14 수신
15 전송중
15 수신
16 전송중
16 수신
17 전송중
17 수신
18 전송중
18 수신
19 전송중
19 수신
20 전송중
20 수신
완료
```

랑데뷰는 버퍼 사이즈를 `0`으로 지정하는 것이다. 생성자에 사이즈를 전달하지 않으면 랑데뷰가 디폴트이다.

이외에도 사이즈 대신 사용할 수 있는 다른 설정 값이 있다.

- `UNLIMITED` - 무제한으로 설정
- `CONFLATED` - 오래된 값이 지워짐.
- `BUFFERED` - 64개의 버퍼. 오버플로우엔 suspend

## **버퍼 오버플로우**

버퍼의 오버플로우 정책에 따라 다른 결과가 나올 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val channel = Channel<Int>(2, BufferOverflow.DROP_OLDEST)
    launch {
        for (x in 1..50) {
            channel.send(x)
        }
        channel.close()
    }

    delay(500L)

    for (x in channel) {
        println("${x} 수신")
        delay(100L)
    }
    println("완료")
}
```

```kotlin
49 수신
50 수신
완료
```

- `SUSPEND` - 잠이 들었다 깨어남
- `DROP_OLDEST` - 예전 데이터를 지움
- `DROP_LATEST` - 새 데이터를 지움
