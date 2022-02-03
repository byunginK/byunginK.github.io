---
layout: post
title: "Kotlin Array"
date: 2022-02-03
categories: [Kotlin]
---

## 배열

기본적인 배열을 생성하기 위해서는 arrayOf()나 Array() 생성자를 사용해 배열을 만든다. 빈 상태의 배열을 지정하는 경우 arrayOfNulls()를 사용

```kotlin
fun main() {
    //자료형 제한하지 않으면 혼합하여 삽입가능
    //또한 객체를 삽입하여 바로 생성 가능
    val arr = arrayOf(1,2,3,"four", true)

    println(arr.get(2)) // 3
    println(arr[2]) // 3
    println(arr.size)

    for (item in arr){
        print(item)
    }
    //[1, 2, 3, four, true]
    println(Arrays.toString(arr))

    arr.set(1,8)
    arr[1] = 8 //위 set과 같음
    println(Arrays.toString(arr))
}
```

### 배열의 여러가지 메소드

first()나 last()를 이용하면 첫 번째나 마지막 요소를 확인하거나 특정 요소의 인덱스, 요소 평균 값, 개수 등을 확인할 수 있습니다. 그 밖에 요소의 순서를 완전히 뒤집는 reversedArray(), reverse(), 요소를 합산할 수 있는 sum(), 주어진 요소를 채우는 fill() 등 다양한 메서드가 존재하므로 필요에 따라 사용하면 좋다.

- 정수는 for루프가 빠름
- 컬렉션을 사용할 경우 순환 메서드가 빠름 또한 메서드 체이닝을 통해 가독성높은 메소드 적용 가능

### 배열 정렬

정렬 기능은 Array에서 확장된 함수들을 이용할 것입니다. 먼저 sortedArray()와 sortedArrayDescending()을 사용해 정렬된 배열을 반환할 수 있습니다. 원본은 그대로 두고 정렬된 배열을 새로 할당할 때에 사용합니다. 만일 원본 배열에 대한 정렬을 진행하려면 sort() 혹은 sortDescending()를 사용

```kotlin
fun main() {
    val arr = arrayOf(8,3,2,6,3,1,2)

    val sortedArr = arr.sortedArray()
    println(Arrays.toString(sortedArr))

    val sortedArrDesc = arr.sortedArrayDescending()
    println(Arrays.toString(sortedArrDesc))

    arr.sort(1,3)
    println(Arrays.toString(arr))
}
```

좀 더 복잡한 형태인 데이터 클래스의 멤버에 따라 정렬할 수 있습니다. 이때는 마찬가지로 Array에서 확장된 sortBy( ) 함수를 이용하면 해당 멤버 변수에 따라 정렬가능
