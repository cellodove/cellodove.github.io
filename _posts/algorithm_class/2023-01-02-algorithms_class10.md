---
title : "[Algorithm] 10장 기본 정렬 - 버블 정렬"
excerpt: "버블 정렬"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2023-01-02
---

![algorithms10_image1.jpg](/assets/images/algorithms10_image1.jpg?raw=true)

## 정렬(sorting) 이란

어떤 데이터들이 주어졌을 때 이를 정해진 순서대로 나열하는 것

정렬은 프로그램 작성시 빈번하게 필요로 한다.

다양한 알고리즘이 고안되었으며 알고리즘 학습의 필수이다.

## 버블 정렬

두 인접한 데이터를 비교해서, 앞에 있는 데이터가 뒤에 있는 데이터보다 크면, 자리를 바꾸는 알고리즘 이다.

![algorithms10_image2.gif](/assets/images/algorithms10_image2.gif?raw=true)

## 버블 정렬 구현

한단계 한단계 진행해 본다.

### 데이터가 두 개일 경우

```java
ArrayList<Integer> dataList = new ArrayList<Integer>();
dataList.add(4);
dataList.add(2);

if (dataList.get(0) > dataList.get(1)) {
    Collections.swap(dataList, 0, 1);
}
System.out.println(dataList);
```

앞의 데이터가 클경우 두번째 데이터와 교환한다.

### 데이터가 세 개일 경우

```java
ArrayList<Integer> dataList = new ArrayList<Integer>();
dataList.add(9);
dataList.add(2);
dataList.add(4);

for (int index = 0; index < dataList.size() - 1; index++) {
    if (dataList.get(index) > dataList.get(index + 1)) {
        Collections.swap(dataList, index, index + 1);
    }
}
System.out.println(dataList);
```

한 칸씩 옮기며 데이터를 비교한다.

## 데이터가 n 개일 경우

n개의 리스트가 있을 경우 최대 n-1번 동작한다.(한번 정렬한 데이터는 반복 하지 않는다.)

로직을 1번 적용할 때마다. 가장 큰 숫자가 뒤에서부터 1개씩 결정된다.

로직을 적용할 때 한 번도 데이터가 교환된 적이 없다면 이미 완전히 정렬 되었다고 판단할 수있다.

```java
public class BubbleSort {

    public ArrayList<Integer> sort(ArrayList<Integer> arrayList){
        //한번 반복할 때마 가장큰수는 배열의 가장 마지막 부분으로 밀린다.
        for (int index = 0; index < arrayList.size()-1; index++){
            boolean swap = false;
            //한번의 반복마다 가장 뒤의 인덱스는 비교하지 않아도 된다.
            for (int i = 0; i<arrayList.size() - 1 - index; i++){
                //만약 뒤의 인덱스가 더 크면 서로 위치를 바꾼다.
                if (arrayList.get(i)>arrayList.get(i+1)){
                    Collections.swap(arrayList, i,i+1);
                    swap = true;
                }
            }
            //만약 한번도 위치 교환이 이루어지지 않으면 정렬이 되었다 판단하고 반목문을 나온다.
            if (!swap){
                break;
            }
        }
        return arrayList;
    }

    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<Integer>();
        for (int i = 0; i < 100; i++) {
            arrayList.add((int)(Math.random() * 100));
        }

        BubbleSort bubbleSort = new BubbleSort();
        System.out.println(bubbleSort.sort(arrayList));
    }
}
```

## 복잡도

반복문이 두 개있기때문에 O(n^2)이다.

최악의 경우 n*(n-1)/2 이고 최선의 경우는 O(n)이다.
