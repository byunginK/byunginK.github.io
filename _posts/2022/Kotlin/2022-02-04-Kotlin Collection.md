---
layout: post
title: "Kotlin Collection"
date: 2022-02-04
categories: [Kotlin]
---

## 컬렉션

코틀린의 컬렉션은 자바 컬렉션의 구조를 확장 구현한 것입니다. 컬렉션의 종류로는 List, Set, Map 등이 있으며 자바와는 다르게 불변형(immutable)과 가변형(mutable)으로 나뉘어 컬렉션을 다룰 수 있습니다. 가변형 컬렉션 타입은 객체에 데이터의 추가/변경이 가능하고, 불변형 컬렉션은 한번 할당하면 읽기 전용이 됩니다. 자바에서는 오로지 가변형 컬렉션만 취급되므로 자바와 상호작용하는 코드에서는 주의

불변형 : listOf, setOf, mapOf
가변형 : mutableListOf, arrayListOf, mutableSetOf, hashSetOf, linkedSetOf, sortedSetOf,
mutableMapOf, hashMapOf, linkedMapOf, sortedMapOf

### List

```kotlin
//불변형
    var numbers: List<Int> = listOf(1,2,3,4,5)
    var names = listOf("one","two","Three")
    var mixed: List<Any> = listOf("one",1,1.5,'c')

    println("numbers : $numbers")
    println("names : $names")
    println("mixed : $mixed")

    println(numbers.size)
    println(numbers.indexOf(3))
    println(numbers.get(3))
    println(numbers[3])
    println(names.contains("one"))

/* 결과
numbers : [1, 2, 3, 4, 5]
names : [one, two, Three]
mixed : [one, 1, 1.5, c]
5
2
4
4
true
*/
```

## Set

setOf()는 읽기전용인 불변형 Set<T> 자료형을 반환

```kotlin
fun main() {
    val mixedTypesSet = setOf("Hello", 5, "world", 3.14, 'c') // 자료형 혼합 초기화
    var intSet: Set<Int> = setOf<Int>(1, 5, 5)  // 정수형만 초기화

    println(mixedTypesSet)
    println(intSet)
}
```

자료형을 혼합하거나 특정 자료형을 지정해 사용할 수 있습니다. 중복 요소를 허용하지 않으므로 intSet에서는 중복된 요소인 5가 결과에서 하나만 나타난다.

- 가변형 mutableSetOf()

mutableSetOf()함수로 추가 및 삭제가 가능한 집합을 만들 수 있습니다. mutableSetOf()는 MutableSet 인터페이스 자료형을 반환하는데, 내부적으로 자바의 LinkedHashSet을 만들어낸다.

_hasSetOf()_ : hashSetOf()헬퍼 함수를 통해 해시 테이블에 요소를 저장할 수 있는 자바의 HashSet컬렉션 (검색속도가 좋다.)

## Map

- 불변형 : mapOf()
- 가변형 : mutableMapOf()

```kotlin
fun main() {
    // 불변형 Map의 선언 및 초기화
    val langMap: Map<Int, String> = mapOf(11 to "Java", 22 to "Kotlin", 33 to "C++")
    for ((key, value) in langMap) { // 키와 값의 쌍을 출력
        println("key=$key, value=$value")
    }
    println("langMap[22] = ${langMap[22]}") // 키 22에 대한 요소 출력
    println("langMap.get(22) = ${langMap.get(22)}") // 위와 동일한 표현
    println("langMap.keys = ${langMap.keys}") // 맵의 모든 키 출력
}
```

## 확장 함수

### 연산자

```kotlin
    val list1 = listOf("one", "two", "three")
    val list2: List<Int> = listOf(1, 2, 3)

    println(list1 + "four") //list에 요소가 추가된 것이 아니다.
    println(list2 + "hello")// list2에서 추가되는것이 아니라 새로운 list로 만들어서 출력
    println(list1 - "one")

    println(list1 + listOf("abc","def")) //[one, two, three, abc, def]
```

### 집계

- forEach : 람다식 처리후 컬렉션 반환 x
- onEach : 람다식 처리후 컬렉션 반환

```kotlin
  list.forEach { print("$it ") }
```

- fold : 초기값과 정해진 식에 따라 처음 요소부터 끝 요소에 적용하여 값 반환
- reduce : fold와 동일, but 초기값 사용하지 않음

### map()

주어진 컬렉션의 요소를 일괄적으로 모든 요소에 식을 적용해 새로운 컬렉션을 만듦

```kotlin
val list = listOf(1,2,3,4,5,6)
println(list.map{ it * 2}) // [2, 4, 6, 8, 10, 12]
```

### groupBy()

주이진 식에 따라 요소를 그룹화 하고 이것을 다시 Map으로 반환

```kotlin
val grpMap = list.groupBy{ if(it % 2 ==0) "even" else "odd"}
println(grpMap) // {odd=[1, 3, 5], even = [2, 4, 6]}
```
