---
layout: post
title: "Kotlin property"
date: 2022-01-30
categories: [Kotlin]
---

## 프로퍼티 (property)

자바의 필드 = 코틀린 프로퍼티

보통 변수에 접근하기 위해 사용하는 접근 메서드는 게터(getter)와 세터(setter)가 있다. 이전 자바에서는 게터, 세터를 모두 선언하여 코드가 길어졌다.
코틀린의 경우 내부에서 접근메서드가 내장되어 컴파일 되므로 별도 선언 없이 접근이 가능하다.

**코트상에서 직접 접근으로 보이지만 내부에서는 getter와 setter을 사용하여 접근 한다.**

```kotlin

//여기에 있는 것은 임시적인 매개변수 (관용적 표현으로 _를 붙인다)
class Person(_id: Int, _name: String, _age: Int){
    //프로퍼티들
    var id: Int = _id
    val name: String = _name
    val age: Int = _age
}

//위와 같은 선언이지만 이렇게 축약 가능
class Person (var id: Int, val name: String, val age: Int)

fun main() {
    val person = Person(1, "kildong", 30)

    person.id = 2 // setter
    println("person : ${person.id}")
```

#### 프로퍼티의 게터와 세터를 직접 작성 할 수도 있으며, val로 선언된 경우에는 getter만 가능하다. 직접 선언 할 경우 내부 getter와 setter에서 변수를 받는 변수가 지정되어있다.

- value: 세터의 매개변수로 외부로부터 값을 가져옴
- field: 프로퍼티를 참조하는 변수

field는 프로퍼티를 참조하는 변수로 보조 필드(backing field)라고도 합니다. get() = field는 결국 각 프로퍼티의 값을 읽는 특별한 식별자입니다. 만일 게터와 세터 안에서 field 대신에 get() = age와 같이 사용하면 프로퍼티의 get()이 다시 호출되는 것과 같으므로 무한 재귀 호출에 빠져 스택 오버플로 오류가 발생할 수 있다.

```kotlin
// 직접 구성한 기본 게터와 세터
class User(_id: Int, _name: String, _age: Int) {
    // 프로퍼티
    val id: Int = _id
        get() = field // field는 id를 참조한다.

    var name: String = _name
        get() = field
        set(value) {
            field = value
        }

    var age: Int = _age
        get() = field
        set(value) {
            field = value
        }
}

fun main() {
    val user1 = User(1, "Kildong", 30)
    // user1.id = 2  // val 프로퍼티는 값 변경 불가
    user1.age = 35 // 세터
    println("user1.age = ${user1.age}") // 게터
}
```

## 지연 초기화

## 프로퍼티는 선언되면 기본적으로는 모두 초기화 해야 한다. 하지만 객체의 정보가 나중에 나타나는 경우 객체 생성과 동시에 초기화 하기가 힘들 경우 지연 초기화 사용

### 1. lateinit

클래스를 선언할 때 프로퍼티 선언은 null을 허용하지 않습니다. 하지만 지연 초기화를 위한 `lateinit` 키워드를 사용하면 프로퍼티에 값이 바로 할당되지 않아도 컴파일러에서 허용한다.

_ \* 단 실행할 때까지 비어있는 상태면 에러를 유발할 수 있으니 주의_

_ \* var에서만 사용 가능_

```kotlin
class Person {
    lateinit var name: String // ① 늦은 초기화를 위한 선언

    fun test() {
        if(!::name.isInitialized) { // ② 프로퍼티의 초기화 여부 판단
            println("not initialized")
        } else {
            println("initialized")
        }
    }
}

fun main() {
    val person = Person()
    person.test()
    person.name = "Kildong" // ③ 이 시점에서 초기화됨(지연 초기화)
    person.test()
    println("name = ${person.name}")
}

/*
결과
not Initialized
Initialized
kildong
*/
```

## 2. by lazy

- 호출 시점에 by lazy {...} 정의에 의해 블록 부분의 초기화를 진행한다.
- 불변의 변수 선언인 val에서만 사용 가능하다.(읽기 전용)
- val이므로 값을 다시 변경할 수 없다.

```kotlin
class LazyTest {
    init {
        println("init block") // ②
    }

    private val subject by lazy {
        println("lazy initialized") // ⑥
        "Kotlin Programming" // ⑦ lazy 반환값
    }
    fun flow() {
        println("not initialized") //  ④
        println("subject one: $subject") // ⑤ 최초 초기화 시점!
        println("subject two: $subject") // ⑧ 이미 초기화된 값 사용
    }
}

fun main() {
    val test = LazyTest() // (1)
    test.flow() //(3)
}
```

객체단위로도 초기화 할 수 있다. (위임 변수 초기화 참고)

```kotlin
class Person(val name: String, val age: Int)

fun main() {
    var isPersonInstantiated: Boolean = false  // ① 초기화 확인 용도

    val person : Person by lazy { // ② lazy를 사용한 person 객체의 지연 초기화
        isPersonInstantiated = true
        Person("Kim", 23) // ③ 이 부분이 Lazy 객체로 반환 됨
    }
    val personDelegate = lazy { Person("Hong", 40) }  // ④ 위임 변수를 사용한 초기화

    println("person Init: $isPersonInstantiated")
    println("personDelegate Init: ${personDelegate.isInitialized()}")

    println("person.name = ${person.name}")  // ⑤ 이 시점에서 초기화
    println("personDelegate.value.name = ${personDelegate.value.name}")  // ⑥ 이 시점에서 초기화
    //${personDelegate.value.name}는 위에서 위임변수로 초기화 하였으므로 value를 통해 접근해야한다.

    println("person Init: $isPersonInstantiated")
    println("personDelegate Init: ${personDelegate.isInitialized()}")
}
```

## 3. by 란?

by를 사용하면 하나의 클래스가 다른 클래스에 위임하도록 선언하여 위임된 클래스가 가지는 멤버를 참조없이 호출할 수 있게 된다.

```kotlin
interface Car {
    fun go(): String
}

class VanImpl(val power: String): Car {
    override fun go() = "는 짐을 적재하며 $power 마력을 가진다."
}

class SportImpl(val power: String): Car {
    override fun go() = "는 경주용 이며 $power 마력을 가진다."
}

class CarModle(val model: String, Carimpl: Car): Car by Carimpl { // 만약 by 위임이 없으면 위에 go()를 오버라이딩 해야한다.
    fun carInfo() {
        println("$model ${go()}") // 참조 없이 각 인터페이스 구현 클래스의 go를 접근
    }
}
/*
class CarModle(val model: String, private val Carimpl: Car): Car{  //위임 하지 않을경우 상속된 인자로 접근 그리고 오버라이드
    fun carInfo() {
        println("$model ${Carimpl.go()}")
    }

    override fun go(): String {
        return "Test"
    }
}
*/
fun main() {
    val myDamas = CarModle("Damas", VanImpl("100마력"))
    val mySport = CarModle("350Z", SportImpl("300마력"))

    myDamas.carInfo() //만약 by위임하지 않고 CarModel()에서 오버라이드된 go()가 호출된다.
    mySport.carInfo()
}
```

프로퍼티의 `lazy`도 by lazy {...} 처럼 by가 사용되어 위임된 프로퍼티가 사용되었다는 것을 알 수 있다.

### 1. observable (감시)

프로퍼티를 위임하는 object인 Delegates로부터 사용할 수 있는 위임자이다.

```kotlin
class User{
    var name: String by Delegates.observable("NONAME"){
        prop, old, new -> println("$old -> $new")
    }
}

fun main() {
    val user = User()

    user.name = "kildong" // 값이 변경되어 콜백 실행
    user.name = "dooly"
}
/* 결과
NONAME -> kildong
kildong -> dooly
*/
```

### 2. vetoable (수여)

프로퍼티를 위임하는 object인 Delegates로부터 사용할 수 있는 위임자이다.

```kotlin
fun main() {
    var max: Int by Delegates.vetoable(0){
        property, oldValue, newValue -> newValue > oldValue
    }

    println(max) // 0
    max = 10
    println(max) // 10

    max = 5
    println(max) //10

}
```
