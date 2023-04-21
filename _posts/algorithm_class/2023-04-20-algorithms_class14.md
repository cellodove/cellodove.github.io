---
title : "[Algorithm] 14장 동적 계획법과 분할 정복"
excerpt: "동적 계획법과 분할 정복"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2023-04-20
---

![algorithms14_image1.jpg](/assets/images/algorithms14_image1.jpg?raw=true)

## 정의

- 동적계획법(DP)

입력 크기가 작은 부분 문제들을 해결한 후, 해당 부분 문제의 해를 활용해서, 보다 큰 크기의 부분 문제를 해결, 최종적으로 전체 문제를 해결하는 알고리즘이다.

상향식 접근법으로, 가장 최하위 해답을 구한 후, 이를 저장하고, 해당 결과값을 이용해서 상위 문제를 풀어가는 방식이다.

Memoization 기법을 사용한다.

메모이제이션이란 프로그램 실행 시 이전에 계산한 값을 저장하여, 다시 계산하지 않도록 하여 전체 실행 속도를 빠르게 하는 기술이다.

- 분활 정복

문제를 나눌 수 없을 때까지 나누어서 각각을 풀면서 다시 합병하여 문제의 답을 얻는 알고리즘이다.

하양식 접근법으로, 상위의 해답을 구하기 위해, 아래로 내려가면서 하위의 해답을 구하는 방식이다. 일반적으로 재귀함수로 구현한다.

문제를 잘게 쪼갤 때, 부분 문제는 서로 중복되지 않는다. 예)병합, 퀵 정렬

## 공통점, 차이점

- 공통점

문제를 잘게 쪼개서, 가장 작은 단위로 분할한다.

- 차이점
    - 동적 계획법
        - 부분 문제는 중복되어, 상위 문제 해결 시 재활용된다.
        - 메모이제이션 기법사용
    - 분할 정복
        - 부분 문제는 서로 중복되지 않음
        - 메모이제이션 기법 사용하지 않는다.

## 예제를 통한 이해

피보나치 수열을 재귀용법과 동적계획법으로 구현하여 이해를 해본다.

### 재귀용법

```java
public class Factorial {
    public int factorialFunc(int data) {
        if (data <= 1) {
            return data;
        }
        return this.factorialFunc(data - 1) + this.factorialFunc(data - 2);
    }
}
```

```java
Factorial fObject = new Factorial();
fObject.factorialFunc(10);
```

### 동적 계획법

```java
public class Dynamic {
    public int dynamicFunc(int data) {
        Integer[] cache = new Integer[data + 1];
        cache[0] = 0;
        cache[1] = 1;
        for (int index = 2; index < data + 1; index++) {
            cache[index] = cache[index - 1] + cache[index - 2];
        }
        return cache[data];
    }   
}
```

```java
Dynamic dObject = new Dynamic();
dObject.dynamicFunc(10);
```