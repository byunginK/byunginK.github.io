---
layout: post
title: "Kotlin kind of Class"
date: 2022-02-01
categories: [Kotlin]
---

## 데이터 클래스

보통 데이터 전달을 위한 객체를 DTO(Data Transfer Object)라고 부른다. (자바에서 POJO라고 불리기도 함) DTO는 구현 로직을 가지고 있지 않고 순수한 데이터 객체를 표현하기 때문에 보통 속성과 속성을 접근하고자 하는 게터/세터를 가진다.
여기에 추가적으로 toString(), equals() 등과 같은 데이터 표현하거나 비교하는 메서드를 가져야 합니다. 자바에서 이것들을 모두 정의하려면 소스 코드가 아주 길어지게 되지만 코틀린에서는 간략하게 표현할 수 있다.

데이터 클래스 선언시 `data` 키워드 사용

```kotlin
data class Customer(var name: String, var email: String)
```

data 클래스는 다음 아래와 같은 조건 따른다.

- 주 생성자는 최소한 하나의 매개변수를 가져야 한다.
- 주 생성자의 모든 매개변수는 val, var로 지정된 프로퍼티여야 한다.
- 데이터 클래스는 abstract, open, sealed, inner 키워드를 사용할 수 없다.

## 중첩 클래스

코틀린에서 중첩 클래스는 기본적으로 정적(static) 클래스처럼 다뤄진다. 중첩 클래스는 객체 생성 없이 접근 가능

```kotlin
class Outer {
    val ov = 5
    class Nested { //중첩 클래스 자바의 static과 같음
        val nv = 10
        fun greeting() = "[Nested] Hello ! $nv" // 외부의 ov에는 접근 불가
        fun accessOuter(){
            println(country) // 외부의 compaion object에는 접근 가능
            getSomthing()
        }

    }
    fun outside() {
        val msg = Nested().greeting() // 객체 생성 없이 중첩 클래스의 메서드 접근
        println("[Outer]: $msg, ${Nested().nv}") // 중첩 클래스의 프로퍼티 접근
    }

    companion object{
        const val country = "Korea" //상수화
        fun getSomthing(){
            return println("Get something..")
        }
    }
}

fun main() {
    // static 처럼 Outer의 객체 생성 없이 Nested객체를 생성 사용할 수 있음
    val output = Outer.Nested().greeting()
    println(output)
    Outer.Nested().accessOuter()
    // Outer.outside() // 에러! Outer 객체 생성 필요
    val outer = Outer()
    outer.outside() //Outer의 메소드는 객체 생성 필요
}
```

## 이너 클래스

내부에 작성된 중첩 클래스와는 좀 다른 역할을 하며, 이너 클래스는 바깥 클래스의 멤버들에 접근가능
`inner` 키워드를 사용하여 선언.

```kotlin
class Smartphone(val model: String) {

    private val cpu = "Exynos"

    inner class ExternalStorage(val size: Int) {
        fun getInfo() = "${model}: Installed on $cpu with ${size}Gb"
        // 바깥 클래스의 프로퍼티 접근
    }
}

fun main() {
    val mySdcard = Smartphone("S7").ExternalStorage(32)
    println(mySdcard.getInfo())
}
```

## 지역 클래스

특정 메서드의 블록이나 init 블록과 같이 블록 범위에서만 유효한 클래스입니다. 블록 범위를 벗어나면 더 이상 사용되지 않는 클래스.

```kotlin
class Smartphone(val model: String) {

    private val cpu = "Exynos"
...
    fun powerOn(): String {
        class Led(val color: String) {  // 지역 클래스 선언
            fun blink(): String = "Blinking $color on $model"  // 외부의 프로퍼티는 접근 가능
        }
        val powerStatus = Led("Red") // 여기에서 지역 클래스가 사용됨
        return powerStatus.blink()
    } // powerOn() 블록 끝
}

fun main() {
...
    val myphone = Smartphone("Note9")
    myphone.ExternalStorage(128)
    println(myphone.powerOn())
}
```

## 익명 객체

코틀린에서는 object 키워드를 사용하는 익명 객체로 같은 기능을 수행한다. (다중의 인터페이스를 구현가능)

```kotlin
interface Switcher { //인터페이스 선언
    fun on() : String
}

class Smartphone(val model: String) {

    private val cpu = "Exynos"

    inner class ExternalStorage(val size: Int) {
        fun getInfo() = "${model}: Installed on $cpu with ${size}Gb" // 바깥 클래스의 프로퍼티 접근
    }
    fun powerOn(): String {
        class Led(val color: String){ //내부에서 사용되는 임시 클래스 (지역 클래스)
            fun blink()  = "Blinking $color on $model"
        }
        val powerStatus = Led("Red")
        //return powerStatus.blink()
        val powerSwitch = object : Switcher { //익명 클래스 선언 (인터페이스 상속)
            override fun on(): String {
                return powerStatus.blink()
            }
        }
        return powerSwitch.on()
    }
}

fun main() {
    val mySdcard = Smartphone("S7").ExternalStorage(32)
    println(mySdcard.getInfo())
    println(Smartphone("S8").powerOn())
}
```

## 실드 클래스

실드 클래스를 선언하려면 `sealed` 키워드를 class와 함께 사용. 실드 클래스 그 자체로는 추상 클래스와 같기 때문에 객체를 만들 수는 없다. 또한 생성자도 기본적으로는 private이며 private이 아닌 생성자는 허용하지 않는다. 실드 클래스는 같은 파일 안에서는 상속이 가능하지만, 다른 파일에서는 상속이 불가능하게 제한. 블록 안에 선언되는 클래스는 상속이 필요한 경우 open 키워드로 선언될 수 있다.

```kotlin
//sealed 클래스는 주로 상태 관리 클래스로 사용
sealed class Result {
    open class Success(val message: String): Result()
    class Error(val code: Int, val message: String): Result()
}

fun main() {
    val result = Result.Error(10,"No disk")
    println(eval(result))
}

fun eval(result: Result): String = when(result){
    is Result.Success -> result.message
    is Result.Error -> result.message
    //모든 조건을 가지기 때문에 else 를 만들 필요 없음
}
```

## 열거형 클래스

열거형 클래스란 여러 개의 상수를 선언하고 열거된 값을 조건에 따라 선택할 수 있는 특수한 클래스.
실드 클래스처럼 다양한 자료형을 다루지 못한다.

```kotlin
interface Score {
    fun getScore(): Int
}

enum class MemberType(var prio: String): Score {
    NORMAL("Thrid") {
        override fun getScore(): Int = 100
    },
    SILVER("Second"){
        override fun getScore(): Int = 500
    },
    GOLD("First") {
        override fun getScore(): Int = 1500
    }
}

fun main() {
    println(MemberType.NORMAL.getScore())
    println(MemberType.GOLD)
    println(MemberType.valueOf("SILVER"))
    println(MemberType.SILVER.prio)
}
/*
결과
100
GOLD
SILVER
Second
*/
```
