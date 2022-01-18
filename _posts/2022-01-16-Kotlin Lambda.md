---
layout: post
title: "Kotlin Lambda"
date: 2022-01-17
categories: [Kotlin]
---

## 람다식과 고차함수

기본형 변수로 할당된 값은 스택에 있고 다른 함수에 인자로 전달하는 경우에는 해당 값이 복사되어 전달된다.
참조형 변수로 할당된 객체는 참조 주소가 스택에 있고 객체는 힙에 있습니다. 참조형 객체는 함수에 전달할 때는 참조된 주소가 복사되어 전달

## 값에 의한 호출

코틀린에서 값에 의한 호출은 함수가 인자로 전달될 경우 람다식 함수는 값으로 처리되어 그 즉시 함수가 수행된 후 값을 전달

```kotlin
fun main() {
    val result = callByValue(lambda()) // 1. 람다식 함수를 호출
    println(result) // true
}

fun callByValue(b: Boolean): Boolean { // 4. 일반 변수 자료형으로 선언된 매개변수
    println("callByValue function")
    return b
}

val lambda: () -> Boolean = {  // 2. 람다 표현식이 두 줄이다
    println("lambda function")
    true 		    // 3. 마지막 표현식 문장의 결과가 반환
}
```

## 이름에 의한 호출

람다식 함수의 이름을 인자로 넣어 사용하는 경우에는 람다식 자체가 매개변수에 복사되고 해당 함수가 호출되 사용되기 전까지는 람다식 함수가 실행되지 않습니다.함수가 호출되 사용될 때 비로소 람다식 함수 내용이 실행됩니다. 이것을 잘 이용하면 상황에 맞춰 즉시 실행할 필요가 없는 코드를 작성하는 경우 이름에 의한 호출 방법을 통해 필요할 때만 람다식 함수가 작동하도록 만들 수 있습니다.

```kotlin
fun main() {
    val result = callByName(otherLambda) // 1. 람다식 이름으로 호출
    println(result)
}

fun callByName(b: () -> Boolean): Boolean { // 2. 람다식 함수 자료형으로 선언된 매개변수
    println("callByName function")
    return b()
}

val otherLambda: () -> Boolean = { // 3. 람다식 실행
    println("otherLambda function")
    true
}
```

## 다른 함수의 참조에 의한 호출

람다식이 아닌 일반 함수를 또 다른 함수의 인자에서 호출하는 고차 함수의 경우를 생각해 봅시다. 두 개의 콜론기호(::)를 함수 이름 앞에 사용해 소괄호와 인자를 생략하고 사용하는 경우 일반 함수를 참조에 의한 호출로 사용할 수 있게 됩니다.

```Kotlin
fun main() {
    // 1. 인자와 반환값이 있는 함수
    val res1 = funcParam(3, 2, ::sum)
    println(res1)

    // 2. 인자가 없는 함수
    hello(::text) // 반환값이 없음

    // 3. 일반 변수에 값처럼 할당
    val likeLambda = ::sum
    println(likeLambda(6,6))
}

fun sum(a: Int, b: Int) = a + b

fun text(a: String, b: String) = "Hi! $a $b"

fun funcParam(a: Int, b: Int, c: (Int, Int) -> Int): Int {
    return c(a, b)
}

fun hello(body: (String, String) -> String): Unit {
    println(body("Hello", "World"))
}
```
