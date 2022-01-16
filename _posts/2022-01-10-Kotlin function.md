---
layout: post
title: "Kotlin function"
date: 2022-01-16
categories: [Kotlin]
---

## 매개변수의 기본값

매개변수의 기본값을 지정하면 인자를 전달하지 않고도 함수를 실행할 수 있다.

```kotlin
fun add(name: String, email: String = "default") {
  // name과 email을 회원 목록에 저장
}
...
add("Youngdeok") // email 인자를 생략하여 호출(name에만 "Youngdeok"이 전달됨)
```

## 가변인자 함수

가변 인자를 사용하면 함수는 하나만 정의해놓고 여러 개의 인자를 받을 수 있다.

```kotlin
// vararg는 인자를 여러개 받을 수 있다.
fun normalVarargs(vararg a: Int){
    for(num in a){
        print("$num ")
    }
}

fun main() {

    normalVarargs(1)
    println()
    normalVarargs(1,2,3,4,5,6) //1 2 3 4 5 6
}
```

## 순수함수

만일 어떤 함수가 같은 인자에 대하여 항상 같은 결과를 반환하면 부작용이 없는 함수라고 말한다. 그리고 부작용이 없는 함수가 함수 외부의 어떤 상태도 바꾸지 않는다면 순수 함수(pure function)라고 한다.

#### 순수 함수의 조건

- 같은 인자에 대하여 항상 같은 값을 반환합니다.
- 함수 외부의 어떤 상태도 바꾸지 않습니다.

```kotlin
// 순수 함수의 예
fun sum(a: Int, b: Int): Int {
    return a + b // 동일한 인자인 a, b를 입력 받아 항상 a + b를 출력(부작용이 없음)
}
```

## 일급 객체

함수형 프로그래밍에서는 함수를 일급 객체이다. 람다식 역시 일급 객체의 특징을 가지고 있다.

#### 일급 객체의 특징

- 일급 객체는 함수의 인자로 전달할 수 있다.
- 일급 객체는 함수의 반환값에 사용할 수 있다.
- 일급 객체는 변수에 담을 수 있다.

## 고차 함수

고차 함수(high-order function)란 다른 함수를 인자로 사용하거나 함수를 결괏값으로 반환하는 함수, 일급 객체 혹은 일급 함수를 서로 주고받을 수 있는 함수

```kotlin
fun main() {
    println(highFunc({ x, y -> x + y }, 10, 20)) // 람다식 함수를 인자로 넘김
}

fun highFunc(sum: (Int, Int) -> Int, a: Int, b: Int): Int = sum(a, b) // sum 매개변수는 함수
```

```kotlin
// 람다식을 마지막에 선언
fun highFunc(a: Int, b: Int, sum: (Int, Int) -> Int): Int{
    return sum(a, b)
}

// 위에서 람다식을 마지막에 선언하여 아래 main에서 람다식을 맨 마지막에 쓸 수 있고
// () 밖에 선언이 가능하여 식이 길경우 가독성 좋게 내려서 작성 가능
fun main() {
    val result = highFunc(1, 3) {
        x, y -> x + y
    }
    println(result)
}
```

## 함수형 프로그래밍의 정의와 특징

- 순수 함수를 사용해야 합니다.
- 람다식을 사용할 수 있습니다.
- 고차 함수를 사용할 수 있습니다.
