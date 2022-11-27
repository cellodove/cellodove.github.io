---
title : "[Algorithm] 2장 자료구조 - 배열 (Array)"
excerpt: "자료구조 - 배열 (Array)"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2022-11-26
---

![algorithms2_image1.jpg](/assets/images/algorithms2_image1.jpg?raw=true))

## 배열**(Array)**

데이터를 나열하고, 각 데이터를 인덱스에 대응하도록 구성한 데이터 구조이다.

### 배열이 필요한 이유

하나의 변수에 여러 데이터를 관리할 수 있다.

관련성 있는 데이터를 함께 변수에 저장하므로 데이터를 찾는데 용이하다.

### 장점

빠른 검색 성능 - 인덱스를 이용한 무작위 접근이 가능하므로 빠른 검색이 가능하다.

순차 접근의 경우에도 배열은 데이터를 하나의 연속된 메모리 공간에 할당하므로 Linked list보다 빠른 성능을 보여준다.

### 단점

데이터 추가/삭제 어려움 - 항목의 모든 요소를 이동시켜야한다.

크기를 바꿀 수 없음 - 너무 크게 잡으면 메모리 낭비, 작게잡으면 저장 불가

메모리 재사용 불가 - 배열은 초기 사이즈만큼의 메모리를 할당 받기때문에 데이터 존재 유무와 상관 없이 일정한 크기의 메모리를 점유하고있다.

![algorithms2_image2.png](/assets/images/algorithms2_image2.png?raw=true)

## 자바에서의 배열

자바에서 배열을 쉽게 다루기위해 ArrayList를 제공한다.

ArrayList 클래스는 가변 길이의 배열을 다룰 수 있는 기능을 제공한다.

### ArrayLisy와 List의 차이점

```java
List<Integer> list1 = new ArrayList<Integer>();
ArrayList<Integer> list1 = new ArrayList<Integer>();
```

List 는 인터페이스이고, ArrayList 는 클래스이다.

ArrayList 가 아니라, List 로 선언된 변수는 필요에 따라 다른 리스트 클래스를 쓸 수 있는 **구현상의 유연성** 을 제공한다.

```java
List<Integer> list1 = new ArrayList<Integer>();
list1 = new LinkedList<Integer>();
```

### 다차원 배열

자바에서는 기본적으로 다차원 배열을 작성할 수 있다.

- 2차원 배열

```java
Integer data_list[][] = { {1, 2, 3}, {4, 5, 6} };

System.out.println( data_list[0][1] ); //2
System.out.println( data_list[1][1] ); //5
```

- 3차원 배열

```java
Integer[][][] data_list = { 
        {
            {1, 2, 3}, 
            {4, 5, 6} 
        },
        {
            {7, 8, 9}, 
            {10, 11, 12} 
        }
};

System.out.println( data_list[0][1][1] ); //5
System.out.println( data_list[1][1][2] ); //12
```

## 예제

1. 3차원 배열에서 8, 10, 2 를 순서대로 각각의 라인에 출력해보자

```java
Integer[][][] data_list = { 
        {
            {1, 2, 3}, 
            {4, 5, 6} 
        },
        {
            {7, 8, 9}, 
            {10, 11, 12} 
        }
};
```

- 답

```java
System.out.println( data_list[1][0][1] );
System.out.println( data_list[1][1][0] );
System.out.println( data_list[0][0][1] );
```

1. 1차원 배열에서, 문자 M 을 가지고 있는 아이템의 수를 출력해보자

```java
String dataset[] = {
    "Braund, Mr. Owen Harris",
    "Cumings, Mrs. John Bradley (Florence Briggs Thayer)",
    "Heikkinen, Miss. Laina",
    "Futrelle, Mrs. Jacques Heath (Lily May Peel)",
    "Allen, Mr. William Henry",
    "Moran, Mr. James",
    "McCarthy, Mr. Timothy J",
    "Palsson, Master. Gosta Leonard",
    "Johnson, Mrs. Oscar W (Elisabeth Vilhelmina Berg)",
    "Nasser, Mrs. Nicholas (Adele Achem)",
    "Sandstrom, Miss. Marguerite Rut",
    "Bonnell, Miss. Elizabeth",
    "Saundercock, Mr. William Henry",
    "Andersson, Mr. Anders Johan",
    "Vestrom, Miss. Hulda Amanda Adolfina",
    "Hewlett, Mrs. (Mary D Kingcome) ",
    "Rice, Master. Eugene",
    "Williams, Mr. Charles Eugene",
    "Vander Planke, Mrs. Julius (Emelia Maria Vandemoortele)",
    "Masselmani, Mrs. Fatima",
    "Fynney, Mr. Joseph J",
    "Beesley, Mr. Lawrence",
    "McGowan, Miss. Anna",
    "Sloper, Mr. William Thompson",
    "Palsson, Miss. Torborg Danira",
    "Asplund, Mrs. Carl Oscar (Selma Augusta Emilia Johansson)",
    "Emir, Mr. Farred Chehab",
    "Fortune, Mr. Charles Alexander",
    "Dwyer, Miss. Ellen",
    "Todoroff, Mr. Lalio"
};
```

- 답

```java
public static void main(String[] args) {
    int count = 0;
    for(int i = 0; i<dataset.length; i++){
        if (dataset[i].contains("M")){
            count ++;
        }
    }
    System.out.println( count );
}
```
