---
title : "[Algorithm] 12장 기본 정렬 - 삽입 정렬"
excerpt: "삽입 정렬"

categories :
  - Algorithm_Class

toc: true
toc_sticky: true
last_modified_at: 2023-01-09
---

![algorithms12_image1.jpg](/assets/images/algorithms12_image1.jpg?raw=true)

## 삽입 정렬

삽입 정렬은 두 번째 원소부터 시작해서 그 앞의 원소들과 비교하여 삽입할 위치를 지정한 후, 원소를 뒤로 옮기고 지정된 자리에 자료를 삽입하여 정렬하는 알고리즘이다.

![algorithms12_image2.gif](/assets/images/algorithms12_image2.gif?raw=true)

## 구현

```java
public class InsertionSort {

    public ArrayList<Integer> sort(ArrayList<Integer> arrayList){

        for (int index = 0; index < arrayList.size(); index++){
            //index+1을 할필요는 없으나 하지않으면 무조건 시작의 한번 조건은 버려지기때문에 +1을 하여 하나의 조건도 버리지 않는다.
            for (int cursor = index + 1; cursor > 0; cursor--){
                //해당 위치값이 앞의 값보다 작다면 서로 교환한다.
                if (arrayList.get(cursor) < arrayList.get(cursor-1)){
                    Collections.swap(arrayList,cursor,cursor-1);
                }else {
                    //이미 앞에서 작다면 그 뒤의 데이터들도 지금 값보다 작기때문에 비교할 필요가 없다.
                    break;
                }
            }
        }

        return arrayList;
    }

    public static void main(String[] args) {
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            arrayList.add((int)(Math.random() * 100));
        }

        InsertionSort insertionSort = new InsertionSort();
        System.out.println(insertionSort.sort(arrayList));
    }
}
```

## 복잡도

최선의 경우 교환없이 1번의 비교만 이루어져 O(n)이다.

평균 복잡도는 O(n^2)이다.

### 정렬 알고리즘 시간 복잡도 비교

![algorithms12_image3.png](/assets/images/algorithms12_image3.png?raw=true)
