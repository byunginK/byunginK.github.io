---
layout: post
title: "Kotlin basic method"
date: 2022-01-25
categories: [Kotlin]
---

## 표준 라이브러리

람다식을 사용하는 코틀린의 표준 라이브러리에서 let(), apply(), with(), also(), run() 등 여러 가지 표준 함수를 활용해 코드의 효율을 높입니다. 이러한 함수를 통해 기존의 복잡한 코드를 단순화하고 효율적으로 만들 수 있다.

## let()

let()함수는 함수를 호출하는 객체 T를 이어지는 block의 인자로 넘기고 block의 결과값 R을 반환

** 람다식에서 it과 this를 인자로 받을 경우 값을 가져오는 방식이 다르다. it은 인자를 복제하여 후 처리를 진행하고, this는 인자의 참조값을 그대로 후 처리를 진행한다. **

-예제 코드

```kotlin
    val score: Int? = 32
...
    // let을 사용해 null 검사를 제거
    fun checkScoreLet() {
        score?.let { println("Score: $it") } // ① it은 32를 반환
        val str = score.let { it.toString() } // ② 32를 String으로 변환 후 반환
        println(str)
    }
    checkScoreLet()
```

### let 함수의 체이닝

여러 메서드를 연속적으로 호출한다.

```kotlin
var a = 1
var b = 2

a = a.let { it + 2 }.let { // it은 1을 복제
    val i = it + b // it은 위에서 처리된 결과 값 3을 복제한다.
    i  // 마지막 식 반환
}
println(a) //5
```

### let 함수의 중첩

```kotlin
var x = "Kotlin"
    x.let { outer ->
        outer.let { inner ->
            print("Inner is $inner and outer is $outer") // it 을 사용하지않고 명시적이름을 사용 inner & outer 둘다 "Kotlin"
        }
    }
```

### 반환값은 바깥쪽의 람다식에만 적용

```kotlin
fun main() {
    var x = "Kotlin"
    x = x.let { outer ->
        outer.let { inner ->
            println("Inner is $inner and outer is $outer")
            "Inner String" // 반환 되지 않음
        }
        "Outer String" // 해당 문자열이 x에 반환되어 할당
    }
    println(x) // Outer String
}
```

## 사용 예시

### 1. 안드로이드 커스텀 뷰 padding 값 지정

```kotlin
val padding = TypedValue.applyDimension(
    TypedValue.COMPLEX_UNIT_DIP, 16f, resources.displayMetrics).toInt()

setPadding(padding, 0, padding, 0)

```

위의 코드를 let을 사용하여 간략하게 만들 수 있다.

```kotlin
TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 16f,
    resources.displayMetrics).toInt().let{ padding ->
    setPadding(padding, 0, padding, 0) // 계산된 값을 padding 이라는 이름의 인자로 받음
    }
```

위와 같은 코드이며 it으로 사용하였을 경우

```kotlin
TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 16f,
    resources.displayMetrics).toInt().let{
    setPadding(it, 0, it, 0) // padding 이라는 이름 대신 it 사용
    }
```

### 2. null 검사

세이프 콜(?.)과 함께 사용하면 간략하게 검사 가능

```kotlin
var obj: String? // null 일 수 있는 변수
...
if(null != obj){
    Toast.makeText(applicationContext, obj, Toast.LENGTH_LONG).show()
}
```

위처럼 `if(null != obj)`대신 let 사용 가능

```kotlin
obj?.let{  // 세이프 콜 사용
    Toast.makeText(applicationContext, it, Toast.LENGTH_LONG).show()
}
```

만약 else문이 포함되었다면

```kotlin
val firstName: String?
var lastName: String
...
if (null != firstName){
    print("$firstName $lastName")
}else{
    print("$lastName")
}
```

위 코드를 한줄로 줄일 수 있다.

```kotlin
firstName?.let{print("$it $lastName")} ?: print("$lastName")
```

## also()

also()는 함수를 호출하는 객체 T를 이어지는 block에 전달하고 객체 T 자체를 반환

`public inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }`

예제

```kotlin
fun main() {
    data class Person(var name: String, var skill: String)
    val person = Person("Killdong", "swing")

    val a = person.also{
        it.skill = "Java"
        "Success" //마지막 식은 반환되지 않음 (무의미한 식)
    }
    println("a: $a") // a: Person(name=Killdong, skill=Java)
    println("person: $person") //person: Person(name=Killdong, skill=Java)
}
```

## apply()

apply()함수는 also()함수와 마찬가지로 호출하는 객체 T를 이어지는 block으로 전달하고 객체 자체인 this를 반환. 보통 객체 초기화 작업을 수행하는 경우 효과적으로 사용

```kotlin
fun main() {
    data class Person(var name: String, var skills : String)
    var person = Person("Kildong", "Kotlin")

    // 여기서 this는 person 객체를 가리킴
    person.apply { this.skills = "Swift" }
    println(person) // Person(name=Kildong, skills=Swift)

    val retrunObj = person.apply {
        name = "Sean" // ① this는 생략할 수 있음
        skills = "Java" // this 없이 객체의 멤버에 여러 번 접근
    }
    println(person) // Person(name=Sean, skills=Java)
    println(retrunObj) // Person(name=Sean, skills=Java)
}
```

## run()

run()함수는 인자가 없는 익명 함수처럼 동작하는 형태로 단독 사용하거나 확장 함수 형태로 호출하는 형태 두 가지로 사용 가능

### 1. 독립적 사용

독립적으로 사용할 때는 block에 처리할 내용을 넣어주며 마지막 식이 반환

```kotlin
val a = 10
skills = run {
    val level = "Kotlin Level:" + a
    level // 마지막 표현식이 반환됨
}
```

할당 없이 사용할 때는 체이닝을 사용해 특정 결과에 대한 메서드를 실행 가능

```kotlin
   run {
        if (firstTimeView) introView else normalView
    }.show()
```

### 2. 확장 함수로 사용

```kotlin
    val retrunObj = person.apply {
        name = "Sean" // ① this는 생략할 수 있음
        skills = "Java" // this 없이 객체의 멤버에 여러 번 접근
        "Success" // apply()는 반환값이 없음
    }
    println(person) // Person(name=Sean, skills=Java)
    println(retrunObj) // Person(name=Sean, skills=Java)

    val retrunObj2 = person.run {
        this.name = "Kim"
        this.skills = "Swift"
        "Success"
    }
    println(person) // Person(name=Kim, skills=Swift)
    println(retrunObj2) // Success
```

## use()

보통 특정 객체가 사용된 후 닫아야 하는 경우가 생기는데 이때 use()를 사용하면 객체를 사용한 후 close() 등을 자동적으로 호출해 닫아 줄 수 있다.

```kotlin
fun main() {

    PrintWriter(FileOutputStream("d:\\test\\output.txt")).use {
        it.println("hello")
    }
}
```

PrintWirter는 파일을 열거나 새롭게 생성해 파일에 출력할 수 있다. 이때 use를 사용하고 있는데 먼저 콘솔에 출력하듯 println을 통해 파일에 출력할 수 있게 된다.
이후 use는 열었던 파일을 닫아주는 작업을 내부에서 진행
