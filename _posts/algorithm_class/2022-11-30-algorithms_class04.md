---
title : "[Algorithm] 4장 자료구조 - 스택 (Stack)"
excerpt: "자료구조 - 스택 (Stack)"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2022-11-30
---

![algorithms4_image1.jpg](/assets/images/algorithms4_image1.jpg?raw=true)

## 스택 **(Stack)**

스택이란 단어는 ‘차곡 차곡 쌓여진 더미’를 의미한다. LIFO(Last In First Out, 후입선출) 구조이다.

![algorithms4_image2.png](/assets/images/algorithms4_image2.png?raw=true)

push(): 데이터를 스택에 넣기

pop(): 데이터를 스택에서 꺼내기

## 자바에서 사용해보기

```java
Stack<Integer> stackInt = new Stack<Integer>(); // Integer 형 스택 선언

stack_int.push(1); // Stack 에 1 추가
stack_int.push(2); // Stack 에 2 추가
stack_int.push(3); // Stack 에 3 추가

stack_int.pop(); // Stack 에서 데이터 추출 (마지막에 넣은 3이 출력)
stack_int.pop(); // (2이 출력)
stack_int.pop(); // (1이 출력)
```

## 사용 사례

- 재귀 알고리즘
- DFS(깊이 우선 탐색)
- 계산기

## 스택 직접 만들어보기

스택을 자바에서 제공하는 클래스가아닌 직접 ArrayList를 사용하여 구현해보자.

```java
import java.util.ArrayList;

public class MyStack<T> {
    private ArrayList<T> stack = new ArrayList<>();

    public void push(T item){
        stack.add(item);
    }

    public T pop(){
        if (isEmpty()){
            return null;
        }else {
            return stack.remove(stack.size()-1);
        }
    }

    public Boolean isEmpty(){
        return stack.isEmpty();
    }

    public static void main(String[] args) {
        MyStack<String> myStack = new MyStack();
        myStack.push("a");
        myStack.push("b");
        myStack.push("c");

        System.out.println(myStack.pop());
        System.out.println(myStack.pop());
        System.out.println(myStack.pop());
        System.out.println(myStack.pop());
    }

}
```

```java
//결과
c
b
a
null
```
