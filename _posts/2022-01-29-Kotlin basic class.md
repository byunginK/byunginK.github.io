---
layout: post
title: "Kotlin basic class"
date: 2022-01-29
categories: [Kotlin]
---

## 생성자

생성자는 주 생성자(Primary Constructor)와 부 생성자(Secondary Constructor)로 나뉘며 필요에 따라 주 생성자 혹은 부 생성자를 사용할 수 있다. 부 생성자는 필요하면 매개변수를 다르게 여러 번 정의할 수 있다.

선언부를 간략화 하기 위해 매개변수 선언부에 val혹은 var을 사용해 프로퍼티로 선언할 수 있다.

```kotlin
class Bird(var name: String, val wing: Int, var beak: String){ //주 생성자에 프로퍼티 바로 선언도 가능
    //프로퍼티 선언 방식
    /*
    var name: String = _name
    val wing: Int = _wing
    var beak: String = _beak
    */

    /*
    constructor(name: String, wing: Int, beak: String){ // 부생성자 아래와 같지만 this를 통해 위에 선언된 변수선언
        this.name = name
        this.wing = wing
        this.beak = beak
    }
    */
    /*
    constructor(_name: String, _wing: Int, _beak: String){ // 부생성자
        name = _name
        wing = _wing
        beak = _beak
    }
    */
    init { //객체를 생성하면 자동적으로 실행됨
        println("---------- init start ----------")
        name = name.capitalize()
        println("name : $name, wing: $wing, beak: $beak")
        println("---------- init end ------------")
    }

    //메서드
    fun fly(){
        println("Fly")
    }
}

fun main() {
    val coco = Bird("coco", 2, "long")

    coco.fly()
    println(coco.name)
    println(coco.wing)
    println(coco.beak)
}
```

## 상속

상속 가능한 클래스를 선언할 때 앞에 open 키워드를 사용해야 파생 클래스에서 상속할 수 있다.

## 다형성

동일한 것처럼 보이지만 매개변수가 서로 다른 형태를 취하거나 실행 결과를 다르게 가질 수 있는 것을 다형성(polymorphism) 이라고 한다.

상위와 하위 클래스에서 메서드나 프로퍼티의 이름은 같지만 기존의 동작이 다른 동작으로 재정의하는 것을 오버라이딩(overriding)이라고 부른다.

동작은 동일하지만 인자의 형식만 달라지는 경우로 오버로딩(overloading) 이라고 부른다.

```kotlin
open class Bird (var name: String, var wing: Int, var beak: String){ //open은 상속을 받게 해줌
    open fun fly(){ //override를 하게끔 해줌
        println("Fly")
    }
}

//부모 클래스 생성자 프로퍼티를 가져와서 Lark 주생성자에 받아줌
open class Lark(name: String, wing: Int, beak: String) : Bird(name, wing, beak) {

    final override fun fly(){ //파생 클래스에서 오버라이딩 금지 final
        println("Quick Fly")
    }

    fun singHitone(){
        println("sing hition")

    }
}

class Parrot : Bird {
    var language: String
    //부모 생성자를 상속 받고 Parrot 클래스만의 프로퍼티 this로 받음
    constructor(name: String, wing: Int, beak: String, language: String) : super(name, wing, beak){
        this.language = language
    }

    override fun fly(){
        println("Slow Fly")
    }
    fun speak(){
        println("speak: $language")
    }
}

fun main() {
    val lark = Lark("mylark", 2, "short")
    val parrot = Parrot("myparrot",2,"logn","English")

    println("lark - name : ${lark.name}")
    println("parrot -name : ${parrot.name} , lang: ${parrot.language}")

    lark.singHitone()
    lark.fly()

    parrot.speak()
    parrot.fly()
}
```

## super, this

super를 사용하면 상위 클래스의 프로퍼티나 메서드, 생성자를 사용할 수 있다.

this를 이용해 프로퍼티, 메서드, 생성자 등을 참조할 수 있다.

```kotlin
open class Person {
    constructor(firstName: String) {
        println("[Person] firstName: $firstName")
    }
    constructor(firstName: String, age: Int) { // ③
        println("[Person] firstName: $firstName, $age")
    }
}

class Developer: Person {

    constructor(firstName: String): this(firstName, 10) { // ①
        println("[Developer] $firstName")
    }
    constructor(firstName: String, age: Int): super(firstName, age) { // ②
        println("[Developer] $firstName, $age")
    }
}

fun main() {
    val sean = Developer("Sean") // 시작
}
```

## 캡슐화

메서드, 프로퍼티의 접근 범위 가시성을 아래와 같이 지정할 수 있다.

- private: 이 지시자가 붙은 요소는 외부에서 접근할 수 없다.
- public: 이 요소는 어디서든 접근이 가능하다. (기본값)
- protected: 외부에서 접근할 수 없으나 하위 상속 요소에서는 가능하다.
- internal: 같은 정의의 모듈 내부에서는 접근이 가능하다.

```kotlin
private class PrivateText {
    private var i = 1
    private fun privatFunc(){
        i += 1
        println(i)
    }
    fun access(){ //public
        privatFunc()
    }
}

class OtherClass {
    // val pc = PrivateText() 클래스에서 공개 생성 불가
    fun test() {
        val pc = PrivateText() // 분리된 함수에서는 생성 가능
        pc.access()
    }
}

fun main() {
    val pc = PrivateText()
    // pc.i = 3 접근 불가
    // pc.privatFunc() 접근 불가
    pc.access()
}
```

- 아래는 protect에 대한 예시

```kotlin
open class Base{
    protected var i = 1
    protected fun protectedFunc(){
        i += 1
        println(i)
    }
    fun access() {
        protectedFunc()
    }
}

class Derived : Base() {
    var j = 1 + i
    fun derivedFunc(): Int{
        protectedFunc() // Base 클래스의 메서드 접근 가능
        return i // Base 클래스의 프로퍼티 접근 가능
    }
}

class Other {
    fun other() {
        val base = Base()
        // base.i = 3 접근 불가가
    }
}

fun main() {
    val base = Base()
    base.access()

    val derived = Derived()
    derived.j = 3
    derived.derivedFunc()
}
```
