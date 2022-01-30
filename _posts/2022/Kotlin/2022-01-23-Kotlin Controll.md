---
layout: post
title: "Kotlin controll"
date: 2022-01-23
categories: [Kotlin]
---

## when구문

주어진 인자에 대해 다양한 조건을 만들거나 인자 없이 여러개의 조건을 구성할 수 있다. 다른 언어의 switch 구문보다 유연하게 사용 가능

```kotlin
fun main() {
    print("enter score: ")
    val score = readLine()!!.toDouble()
    var grade: Char = 'f'

    when(score){
        in 90.0..100.0 -> grade = 'A'
        in 80.0..89.9 -> grade = 'B'
        in 70.0..79.9 -> grade = 'C'
        else -> { // 블록 구문 사용 가능
        print("과락")
        }
    }

}
```

화살표 오른쪽에 사용한 수행 문장에서는 한줄인 경우에는 중괄호가 필요하지 않으며 또 switch~case에서 사용하던 break문을 사용하지 않아도 된다.

### 인자가 없는 when

인자가 주어지지 않으면 else if 처럼 조건을 사용하여 선언 할 수 있다.

```kotlin
when {
  조건[혹은 표현식] -> 실행문
  ...
}
```

## for 구문

자바와 달리 ;나 변수를 선언하여 증감 시키지 않는다. `for (요소 변수 in 컬렉션 혹은 범위) { 반복할 본문 }`의 형태이다.

`for (i in 5 downTo 1) print(i)` downTo는 5에서 1까지 하나씩 감소한다. (하행반복)

`for (i in 1..5 step 2) print(i)` step 은 증가하는 단계수 설정

`for (i in 0 until 5) print(i)` until은 0부터 4까지 이며, 마지막 범위의 -1을 의미 한다.

### break와 continue 라벨 사용

라벨을 사용하여 중단되는 위치 다시 조건문을 타는 위치를 제어할 수 있다.

```kotlin
fun labelBreak() {
    println("labelBreak")
    first@ for(i in 1..5) {
        second@ for (j in 1..5) {
            if (j == 3) break@first //second 반복문을 빠져나가 first 조건식으로 바로 빠져나간다.
            println("i:$i, j:$j")
        }
        println("after for j")
    }
    println("after for i")
}
```
