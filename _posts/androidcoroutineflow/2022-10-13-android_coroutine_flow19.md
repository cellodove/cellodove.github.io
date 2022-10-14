---
title : "[Coroutine&Flow] 19장 채널 파이프라인"
excerpt: "채널 파이프라인"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-10-13
---

![coroutine19_image1.jpg.jpg](/assets/images/coroutine19_image1.jpg.jpg?raw=true)

## **파이프라인**

파이프 라인은 일반적인 패턴이다. 하나의 스트림을 프로듀서가 만들고, 다른 코루틴에서 그 스트림을 읽어 새로운 스트림을 만드는 패턴.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.produceStringNumbers(numbers: ReceiveChannel<Int>): ReceiveChannel<String> = produce {
    for (i in numbers) {
        send("${i}!")
    }
}

fun main() = runBlocking<Unit> {
    val numbers = produceNumbers()
    val stringNumbers = produceStringNumbers(numbers)

    repeat(5) {
        println(stringNumbers.receive())
    }
    println("완료")
    coroutineContext.cancelChildren()
}
```

```kotlin
1!
2!
3!
4!
5!
완료
```

## **홀수 필터**

파이프라인을 응용해 홀수 필터를 만들어 보자

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.filterOdd(numbers: ReceiveChannel<Int>): ReceiveChannel<String> = produce {
    for (i in numbers) {
        if (i % 2 == 1) {
            send("${i}!")
        }
    }
}

fun main() = runBlocking<Unit> {
    val numbers = produceNumbers()
    val oddNumbers = filterOdd(numbers)

    repeat(10) {
        println(oddNumbers.receive())
    }
    println("완료")
    coroutineContext.cancelChildren()
}
```

```kotlin
1!
3!
5!
7!
9!
11!
13!
15!
17!
19!
완료
```

## **소수 필터**

파이프라인을 연속으로 타면서 원하는 결과를 얻을 수 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int): ReceiveChannel<Int> = produce {
    for (i in numbers) {
        if (i % prime != 0) {
            send(i)
        }
    }
}

fun main() = runBlocking<Unit> {
    var numbers = numbersFrom(2)

    repeat(10) {
        val prime = numbers.receive()
        println(prime)
        numbers = filter(numbers, prime)
    }
    println("완료")
    coroutineContext.cancelChildren()
}
```

```kotlin
2
3
5
7
11
13
17
19
23
29
완료
```

원한다면 디스패처를 이용해 CPU 자원을 효율적으로 이용하는 것이 가능하다.
