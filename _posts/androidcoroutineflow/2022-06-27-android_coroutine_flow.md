---
title : "1. 코루틴을 사용해야하는 이유"
excerpt: "코루틴을 사용해야하는 이유에 대하여"

categories :
  - CoroutineFlow

toc: true
toc_sticky: true
last_modified_at: 2022-06-27
---

![coroutine1_image1.jpg](/assets/images/coroutine1_image1.jpg?raw=true)

## 동시성이 없다면?

### 사용자가 불편하다

과거에는 동시성이라는 기능이 없었다. 나는 겪어본적이 없지만 이에따라 불편한점이 많이 있었다.

예를들면 인터넷에서 파일을 다운로드 받을때 파일이 전부 다운받아 지기전에는 다른 행동을 하지 못했다.

![coroutine1_image2.jpeg](/assets/images/coroutine1_image2.jpeg?raw=true)

### 하드웨어 리소스 낭비

라이젠 3950X는 749,070 밉스이다. 동시성이 없다면 초당 7천억번의 명령을 처리할 수 있는 기회를 날릴 수 있다. 하드웨어 성능이 높아짐에 따라 멀티 태스킹에 대한 요구가 등장하기 시작한다.

- 💡**MIPS (million instructions per second)**

    프로세서의 성능을 나타내는 단위로 초당 몇 백만개의 명령어를 처리할 수 있는지를 나타내는 수치이다. 예를 들어 50 MIPS 성능을 갖는 프로세서가 있다면 초당 5천만개의 명령어를 처리할 수 있음을 의미한다.

![coroutine1_image3.png](/assets/images/coroutine1_image3.png?raw=true)

## 동시성 해결방법

### 합의해서 나누어 쓴다

자원을 서로 나누어 쓰는 것이 비선점형 멀티태스킹이다.

누군가 자원을 독점하고 죽어버리는 경우가 많아 현실에서는 거의 성공하지 못했다.

![coroutine1_image5.png](/assets/images/coroutine1_image5.png?raw=true)

- **비선점형(Non-Preemptive)**

    운영체제가 하드웨어 자원을 선점하지 않고, 응용프로그램이 하드웨어 자원을 자발적으로 반환할 때까지 기다린다.

### 중재자를 둔다

Ring0의 권한을 가진 운영체제가 자원을 관리한다.

응용프로그램은 Ring3이다.

![coroutine1_image4.png](/assets/images/coroutine1_image4.png?raw=true)

- **선점형(Preemptive)**

    운영체제가 수행 중인 각 프로그램의 실행시간을 할당하는 방식이다.

    응답이 없는 응용 프로그램에게 강제적으로 자원을 빼앗아 오는 일이 가능하며, 실행중인 각 프로그램들은 서로에게 지연/방해 등의 영향을 주지 않는다.

Ring레벨이 높은 곳에서 낮은 곳으로만 접근 가능하다. 예를 들어 Ring 0 → Ring 3은 접근 가능하지만 Ring 3 → Ring 0 은 직접 접근이 불가능하다. 시스템 콜 등의 방법으로 간접 접근은 가능하다.

운영체제 커널이나 하드웨어 드라이버는 보통 Ring 0의 특권을 가진다. (시스템 종료, 인터럽트 활성화/비활성화, 하드웨어 입출력)

일반적으로 시분할 등의 방법으로 자원을 나누었다.

![coroutine1_image6.png](/assets/images/coroutine1_image6.png?raw=true)

- 프로세스

    운영체제는 1개 이상의 프로세스를 시분할에 의해 번갈아서 동작을 시킨다.

    코드, 데이터, 힙 스택등의 자료구조를 가진다.

- 스레드

    프로세스는 1개 이상의 스레드를 가지며 시분할에 의해 번갈아서 동작한다.

    일반적으로 스택을 구분하고 나머지는 공유하지만 운영체제 마다 다르다.

    리눅스에서는 프로세스와 스레드의 차이가 크지 않지만 윈도우에서는 프로세스의 생성이 느리기 때문에 스레드의 중요성이 커졌다.

## 병렬성 SMP (Symmetric Multiprocessor)와 가시성

여러 프로세스가 하나의 메모리를 쓰는 모델이다. 여러 프로세스가 물리적으로 여러 일을 동시에 사용하는 일이 가능해졌다.

![coroutine1_image7.jpeg](/assets/images/coroutine1_image7.jpeg?raw=true)

### 여러 CPU가 개별 캐쉬를 사용하게 되면서 발생한 문제

![coroutine1_image8.png](/assets/images/coroutine1_image8.png?raw=true)

CPU 1에서 A[0]을 1 증가하고 CPU 2에서 A[1]을 1 증가했다면
CPU 1의 캐쉬에 있는 A[0]만 0 -> 1이고 CPU 2의 캐쉬에 있는 A[1]만 0 -> 1이게 된다.
심지어 CPU 3에서 데이터 A[0]이나 A[1]을 접근해도 데이터는 0일 수 있다. 그래서 락이나 메모리 베리어가 필요할 수 있다.

### 데이터 갱신 문제

캐시는 캐시라인의 집합인데 64, 128 바이트 등 연속된 사이즈로 묶여있다. 데이터 하나만 교체할 수 없고 캐시라인 통채로 한번에 교체된다.

![coroutine1_image9.png](/assets/images/coroutine1_image9.png?raw=true)

한 캐시라인을 하나의 코어가 갱신하는 동안 다른 코어의 캐시는 무효화되고 접근할 수 없다.
코어 1이 A[0]을 접근하고 코어 2가 A[1]을 접근한다면 코어 2는 코어 1이 A[0]을 갱신하는동안 기다려야한다. 결국 두개의 코어를 쓰는 의미가 없어진다.

## 마치며

지금까지 알아본것처럼 비동기, 병렬성은 점점더 필요성은 늘어나고있지만 막상 프로그래밍하기에는 어려움이 많다. 그래서 이런문제를 해결하기위해 콜백이나 RxJava등이 등장했다. 하지만 과연 콜백과 Rx가 최선인지에는 의문이 있다.

- 콜백 코드

![coroutine1_image10.png](/assets/images/coroutine1_image10.png?raw=true)

- 코루틴 코드

![coroutine1_image11.png](/assets/images/coroutine1_image11.png?raw=true)

유지보수성이나 가시성을 생각해보았을때 콜백이나 rx보다 코루틴이 훨씬 좋다. 코루틴은 동시성이나 아니냐에따라 코드가 달라지지 않는다.

코루틴은 비동기와 병렬성을 순차적으로 짤수있는 코드방법을 제공한다. 플로우는 비동기와 병렬성을 스트림 형태로 보여준다. 이둘을 이용해 순차적으로 짤수있는곳에서는 코루틴을 사용하고 스트림이 맞는 부분에서는 플로우를 사용하겠다.

## 참조

> [https://yoda.wiki/wiki/Instructions_per_second](https://yoda.wiki/wiki/Instructions_per_second)
>
> [http://oojoo.egloos.com/m/1196903](http://oojoo.egloos.com/m/1196903)
>
> [https://www.fun-coding.org/syscall.html](https://www.fun-coding.org/syscall.html)
>
> [https://medium.com/@flqjsl/콜백이란-무엇인가-56c26e1f1bc3](https://medium.com/@flqjsl/%EC%BD%9C%EB%B0%B1%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-56c26e1f1bc3)
>