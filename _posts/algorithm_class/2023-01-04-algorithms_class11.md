---
title : "[Algorithm] 11장 기본 정렬 - 선택 정렬"
excerpt: "선택 정렬"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2023-01-04
---

![algorithms11_image1.jpg](/assets/images/algorithms11_image1.jpg?raw=true)

## 선택 정렬

버블 정렬과 유사한 알고리즘으로 해당 순서에 원소를 넣을 위치는 이미 정해져 있고, 어떤 원소를 넣을지 선택하는 알고리즘이다.

1. 주어진 배열에서 최소 값을 찾는다.
2. 그 값을 맨 앞에 위치한 값과 교체한다.
3. 맨 처음 위치를 뺀 나머지 배열을 같은 방법으로 교체한다.

![algorithms11_image2.gif](/assets/images/algorithms11_image2.gif?raw=true)

## 구현

```java
public class SelectionSort {

    public ArrayList<Integer> sort(ArrayList<Integer> arrayList){
        int lowest;

        for (int index = 0; index < arrayList.size(); index++){
            //가장 작은 값을 저장할 변수
            lowest = index;

						//한 칸씩 정렬이되므로 한 칸씩 앞으로 나아간다 
            for (int cursor = index; cursor < arrayList.size(); cursor++){
                //리스트를 순회하여 가장 작은 값을 찾는다.
                if (arrayList.get(lowest) > arrayList.get(cursor)){
                    lowest = cursor;
                }
            }
            //찾은 가장 작은값을 교환한다.
            Collections.swap(arrayList,index, lowest);
        }

        return arrayList;
    }

    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            arrayList.add((int)(Math.random() * 100));
        }

        SelectionSort selectionSort = new SelectionSort();
        System.out.println(selectionSort.sort(arrayList));
    }
}

```

## 복잡도

데이터의 개수가 n개라고 했을 때,

- 첫 번째 회전에서의 비교횟수 : 1 ~ (n-1) => n-1
- 두 번째 회전에서의 비교횟수 : 2 ~ (n-1) => n-2
- (n-1) + (n-2) + .... + 2 + 1 => n(n-1)/2

그래서 시간 복잡도는 O(n^2)이다.
