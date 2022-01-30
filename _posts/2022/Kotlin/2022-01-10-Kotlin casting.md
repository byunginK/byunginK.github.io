---
layout: post
title: "Kotlin casting"
date: 2022-01-10
categories: [Kotlin]
---

## 검사와 자료형 변환

코틀린의 자료형은 모두 참조형으로 선언합니다. 하지만 컴파일을 거쳐서 최적화 될 때는 Int, Long, Short와 같은 참조형 자료형은 기본형 자료형으로 변환됩니다. 참조형과 기본형의 저장방식은 서로 다르다

### 자료형 변환 메소드

- toByte: Byte

- toLong: Long

- toShort: Short

- toFloat: Float

- toInt: Int

- toDouble: Double

- toChar: Char

### 기본형과 참조형 비교

단순히 값만 비교할 때는 이중 등호(==)를 사용하고 참조 주소를 비교하려면 삼중 등호(===)를 사용

```kotlin
fun main() {
    val a: Int = 128
    //Int 자료형으로 자동 캐스팅
    val b = a

    println(a==b)//true
    println(a===b) // true

    val c: Int? = a
    val d: Int? = a
    val e: Int? = c

    println(c==d) //true
    println(c === d) //false
    println( c === e) //true
}
```

### 스마트 캐스트

Number형을 사용하면 숫자를 저장하기 위한 특수한 자료형 객체를 만든다. Number형으로 정의된 변수에는 저장되는 값에 따라 정수, 실수 등으로 자료형이 변환된다. 하위 자료형을 모두 포함하므로 하위에 속하는 어떤 자료형으로도 변환 할 수 있다. 다만 String은 Number에 속하지 않으므로 변환할 수 없다.

### 검사

코틀린에서 is는 해당 자료형이 무엇인지 확인할 수 있는 메소드를 제공한다.

```kotlin
fun main() {
    val a: Number = 2000
    val b = 3333

    if(a is Int){
        println("a is Int")
    }else if(a !is Int){
        println("Not a Int")
    }
}
```

### 묵시적 변환

Any형은 자료형이 특별히 정해지지 않은 경우에 사용한다. 코틀린의 Any형은 모든 클래스의 뿌리입니다. 우리가 자주 사용한 Int나 String 그리고 사용자가 직접 만든 클래스까지 모두 Any형의 자식 클래스이다.

즉,코틀린의 모든 클래스는 바로 이 Any 형이라는 슈퍼 클래스(Superclass)를 가집니다. 따라서 Any를 사용해 변수를 선언하면 해당 변수는 어떤 형으로도 변환할 수 있게 된다.
