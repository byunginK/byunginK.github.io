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

## 인라인 함수

인라인(inline) 함수는 이 함수가 호출되는 곳에 내용을 모두 복사해 넣어 함수의 분기 없이 처리되기 때문에 코드의 성능을 높일 수 있다. 인라인 함수는 코드가 복사되어 들어가기 때문에 대개 내용은 짧게 작성하고, 람다식 매개변수를 가지고 있는 함수 형태를 권장한다.

```kotlin
inline fun shortFunc(a: Int, out: (Int) -> Unit){
    println("Before calling out()")
    out(a)
    println("After calling out()")
}

fun main() {
    shortFunc(3){println("a: $it")}
}
    /* 결과
    Before calling out()
    a: 3
    After calling out()
    */
```

### 1. 인라인 함수 제한

`noinline` 키워드를 람다식 함수 매개변수 정의에서 사용하면 기본적으로 람다식 함수를 매개변수로 가진 해당 함수가 inline으로 정의되어 있다고 하더라도 사용할 때 람다식 함수를 inline 시키지 않는다.

```kotlin
inline fun sub(out1: () -> Unit, noinline out2: () -> Unit) { ...
```

### 2. 인라인 함수와 비지역 반환

코틀린에서는 익명 함수를 종료하기 위해서 return을 사용할 수 있다. 이때 특정 반환값 없이 return만 사용. 람다식 함수를 인자로 사용하는 함수는 의도하지 않게 람다식 함수 바깥에 있는 함수가 같이 반환되 버리는데 이것을 비지역 반환이라고 한다.

```kotlin
fun main() {
    shortFunc2(3) {
        println("First call: $it")
        return // ①
    }
}
/* 의도치 않게 shortFunc2 처리가 전부 되기 전에 return 되어 버림
결과
Before calling out()
First call: 3
 */

inline fun shortFunc2(a: Int, out: (Int) -> Unit) {
    println("Before calling out()")
    out(a)
    println("After calling out()") // ②
}
```

이때 이런 비지역 반환을 금지하려면 crossinline이라는 키워드를 람다식 함수 앞에 사용해 함수의 본문 블록에서 return이 사용되는 것을 금지할 수 있다.

```kotlin
fun main() {
    shortFunc2(3) {
        println("First call: $it")
        return // 아래 람다식에 corssinline을 선언하여 return은 에러로 컴파일되지 않는다.
    }
}

inline fun shortFunc2(a: Int, crossinline out: (Int) -> Unit) {
    println("Before calling out()")
    out(a)
    println("After calling out()")
}
```

만약 return을 사용 하려면 라벨을 사용하게 된다. 아래는 기존 비지역 반환 코드

```kotlin
fun main() {
    retFunc() //결과 : Start of Func
}

inline fun inLineLambda(a: Int,b:Int, out: (Int, Int) -> Unit){
    out(a,b)
}

fun retFunc(){
    println("Start of Func")
    inLineLambda(12,3){ a, b ->
        val result = a + b
        if(result > 10) return // 비지역 반환이 되기 때문에 함수 자체를 벗어난다.
        println("result : $result")
    }
    println("End of Func")
}
```

라벨을 사용하여 return

```kotlin
fun main() {
    retFunc()
//결과
//Start of Func
//End of Func
}

inline fun inLineLambda(a: Int,b:Int, out: (Int, Int) -> Unit){
    out(a,b)
}

fun retFunc(){
    println("Start of Func")
    inLineLambda(12,3)lit@{ a, b ->
        val result = a + b
        if(result > 10) return@lit
        println("result : $result")
    }
    println("End of Func")
}
```

암묵적인 라벨도 가능하다. 람다식 표현식 블록에 직접 라벨을 쓰는 것이 아닌 람다식 함수의 명칭을 그대로 라벨처럼 사용할 수 있는데 이것을 암묵적 라벨이라고 한다.

```kotlin
fun retFunc() {
    println("start of retFunc")
    inlineLambda(13, 3) { a, b ->
        val result = a + b
        if(result > 10) return@inlineLambda
        println("result: $result")
    }
    println("end of retFunc")
}
```

## 익명 함수

물론 람다식 함수 표현식 대신에 익명 함수를 넣을 수도 있습니다. 익명 함수는 앞서 배운 것 처럼 fun (...) {...} 형태로 이름 없이 특정 함수의 인자로 넣을 수 있다. 이때는 익명함수 내부에서 라벨을 사용하지 않고 단순히 return만 사용하더라도 비지역 반환이 일어나지 않습니다. 따라서 일반 함수의 반환처럼 편하게 사용할 수 있다.

```kotlin
fun retFunc(){
    println("Start of Func")
    inLineLambda(12,3, fun (a, b){
        val result = a + b
        if(result > 10) return //비지역 반환이 일어나지 않는다.
        println("result: $result")
    }) //inlineLambda()함수의 끝
    println("End of Func")
}
```

## 확장 함수

클래스에는 다양한 함수가 정의되어 있는데 해당 함수를 멤버 메서드라고 한다. 확장 함수는 기존 멤버 베서드는 아니지만 내가 원하는 함수를 하나 더 포함 하고 싶을때 만들 수 있는 방법이다.

```kotlin
fun main() {
    val source = "Hello World!"
    val target = "Kotlin"
    println(source.getLongString(target))
}

// String을 확장해 getLongString 추가 (this는 String.getLongString에서 String을 의미 "hello world"를 가르킴)
fun String.getLongString(target: String): String =
        if (this.length > target.length) this  else target
```

## 중위 함수

중위 표현법(infix notation)이란 클래스의 멤버 호출 시 사용하는 점(.)을 생략하고 함수 이름 뒤에 소괄호를 붙이지 않아 직관적인 이름을 사용할 수 있는 표현법. 즉, 중위 함수란 일종의 연산자를 구현할 수 있는 함수이며, 중위 함수는 특히 비트 연산자에서 사용한다.

```kotlin
...
    // 중위 표현법
    val multi = 3 multiply 10
...

// Int를 확장해서 multiply() 함수가 하나 더 추가되었음
infix fun Int.multiply(x: Int): Int {  // infix로 선언되므로 중위 함수
    return this * x
}
```

## 꼬리 재귀 함수

코틀린에서는 꼬리 재귀 함수(tail recursive function)를 통해 스택 오버플로 현상을 해결할 수 있다. (tailrec 키워드를 사용)
꼬리 재귀를 사용하면 팩토리얼의 값을 그때 그때 계산하므로 스택 공간을 낭비하지 않아도 되므로 일반 재귀함수보다 훨씬 안전한 코드가 된다.

```kotlin

fun main() {
    val number = 5
    println("Factorial: $number -> ${factorial(number)}")
}

tailrec fun factorial(n: Int, run: Int = 1): Long {
    return if (n == 1) run.toLong() else factorial(n-1, run*n)
}

```
