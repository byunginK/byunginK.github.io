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

```kotlin
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

## 클로저

람다식 함수를 이용하다 보면 내부 함수에서 외부 변수를 사용하고 싶을 때가 있습니다. 클로저란 람다식으로 표현된 내부 함수에서 외부 범위에 선언된 변수에 접근할 수 있는 개념입니다. 이 때 람다식 안에 있는 외부 변수는 값을 유지하기 위해 람다가 포획(capture)한 변수라고 부릅니다.

실행 시점에서 람다식의 모든 참조가 포함된 닫힌(closed) 객체를 람다 코드와 함께 저장합니다. 이때 이러한 데이터 구조를 클로저(closure)라고 부르는 것

기본적으로 함수 안에 정의된 변수는 로컬 변수로 스택에 저장되어 있다가 함수가 끝나면 같이 사라지게 됩니다. 하지만 클로저 개념에서는 포획한 변수는 참조가 유지되어 종료되어도 사라지지 않고 접근하거나 수정할 수 있게된다.

```kotlin
fun main() {

    val calc = Calc()
    var result = 0 // 외부의 변수
    calc.addNum(2,3) { x, y -> result = x + y }  // 클로저
    println(result) // 값을 유지하여 5가 출력
}

class Calc {
    fun addNum(a: Int, b: Int, add: (Int, Int) -> Unit) { // 람다식 add에는 반환값이 없음
        add(a, b)
    }
}
```

위 코드와 같이 result는 람다식 내부에서 재할당 되어 사용되는데 이때 할당된 값은 유지되 출력문에서 사용할 수 있게 됩니다. 클로저에 의해 독립된 복사본을 가지고 사용
