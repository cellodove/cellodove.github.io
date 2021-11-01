---
title : "안드로이드 Handler"
excerpt: "안드로이드 Handler에 대하여"

categories :
  - Android

toc: true
toc_sticky: true
last_modified_at: 2021-10-30 
---



핸들러에 대해 알아보자 먼저 핸들러는 안드로이드에서 main Thread가 아닌 sub Thread를 사용하기위해 사용된다. 그럼왜 main Thread가 아닌 sub Thread를 사용해야할까?

먼저 ANR과 Thread에 대해 알아야한다

## ANR(Application Not Responding)

즉 어플리케이션이 응답하지 않는다는 에러이다.

**이 에러의 원인은 main Thread (UI Thread)가 일정 시간 어떤 Task에 잡혀 있으면 발생하게 된다.**

어기서 어떤 Task란 시간이 오래걸리는 작업들을 의미한다. 예를들어 I/O명령과 관련된 느린 작업이나, 긴 계산을 하고있는 경우이다.

그렇다면 ANR을 예방하기 위해서는 어찌해야할까

Main Thread에서 실행되는 임의의 method는 최소한의 일을 해야한다. 특히 onCreate나 onResume

같은 안드로이드의 핵심 생명주기에 속해있는 method에서는 가능한 적은 일을 수행해야한다.

아니면 시간 소모가 많은 작업은 Sub Thread를 통해 처리하거나 사용자에게 프로그래스바 등을

이용해 작업의 진행 과정을 안내해 기다리도록 한다.

## Thread

**Thread는 무엇일까? 동시작업을 할려면 필요한 하나의 작업단위이다.**

그렇다면 안드로이드에서 다른 Sub Thread가 필요한 이유는 무엇일까?

만약 sub Thread 없이 Main Thread만으로 구성하게된다면 어떤 Task가 끝날때까지 사용자는

멈춰있는 화면을 보아야한다. 이는곧 ANR에러를 보게된다. 그리하여 오래걸리는 작업들은 외부 Thread를 사용 백그라운드 처리를 해주는 것이 좋다.

그리고 Thread처리는 UI변경이 필요한 경우와 그렇지 않은 경우로 나눌 수 있는데 sub Thread 단독으로는 UI변경을 할 수 없다. 왜그럴까??

## Handler, Looper

안드로이드의 UI는 기본적으로 메인스레드를 주축으로하는 싱글 스레드 모델로 동작한다.

메인스레드에서는 긴 작업을 피해야한다. 긴작업은 여분의 다른 스레드에서 실행하고 UI를 바꿀때는

UI스레드로 접근하도록 스레드간 통신 방법을 사용해야한다. 이때 사용하는 것이 Message나

Runnable객체를 받아와 다른곳으로 전달해주는 Handler 클래스다.

Handler와 Looper는 왜 필요할까?

![Thread.png](/assets/images/Thread.png?raw=true)

병렬 처리로 돌아가고 있는 Main과 Sub Thread에서 같은 textview를 set할 때 어떤 Thread의 것을

따라야할까? 이러한 동기화 문제를 처리하기 위해 안드로이드에서는 메인 스레드에서만 UI작업이

가능하도록 제한했다. Handler와 Looper를 이용해 Sub Thread에서 Main Thread로 UI처리 작업을

전달하던지, MainThread 내에서 자체적으로 처리하던지 해야하는것이다.

![Thread2.png](/assets/images/Thread2.png?raw=true)

### Handler와 Looper의 동작과정

![Thread3.png](/assets/images/Thread3.png?raw=true)

Handler는 단어 의미 그대로 무언가를 처리하는 것이다. 이 친구는 Message와 Runnable객체를 처리한다. Runnable메시지는 run()메서드를 호출해 처리하고, Message는 handleMessage()메서드를 이용해 처리한다.

*Runnable메시지

Thread(Runnable runnable) 생성자로 만들어서 Runnable 인터페이스를 구현한 개체를 생성하여 상속받은 스레드는 추상 메서드 run()을 반드시 구현해야하는 것

1. 메시지는 다른 스레드에 속한 Message Queue에서 전달된다.
2. Message Queue에 메시지를 넣을 땐 Handler - sendMessage()를 이용한다.
3. Looper는 MessageQueue에서 Loop()를 통해 반복적으로 처리할 메시지를 Handler에 전달

4. Handler는 handleMessage를 통해 메시지를 처리한다.

Handler는 의존적이다. Handler혼자서는 아무것도 못한다. MessageQueue가 있어야하고,또 그 메시지를 전달해줄 Looper가 없으면 Handler의 handleMessage()는 아무것도 못한다.

즉 Handler는 Thread와 Looper, MessageQueue가 꼭 팔요하다.

### Looper의 역할

Looper는 하나의 스레드만을 담당할 수 있고 하나의 스레드도 오직 하나의 Looper만을 가질 수 있다.

Looper는 MessageQueue가 비어있는 동안은 아무 행동도 안하고 메시지가 들어오면 해당 메시지가 들어오면 해당 메시지를 꺼내 적절한 Handler로 전달한다. 계속 반복적으로 수행하는 동작 때문에 Looper라는 이름이 붙여졌다고 한다.

## 참조

> [https://itmining.tistory.com/3?category=640759](https://itmining.tistory.com/3?category=640759)
>
> [https://itmining.tistory.com/5?category=640759](https://itmining.tistory.com/5?category=640759)
>