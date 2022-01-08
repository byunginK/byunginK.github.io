---
layout: post
title: "Kotlin DataType"
date: 2022-01-08
categories: [Kotlin]
---

## 기본 자료형과 변수

1. val : 불변형
2. var : 변경가능형

## 코틀린 자료형

보통 프로그래밍 언어의 자료형은 기본형 자료형과 참조형 자료형으로 구분하며 코틀린은 참조형 자료형을 사용합니다.

기본형(Primitive Data Type)은 말 그대로 가공되지 않은 순수한 자료형을 말하며 프로그래밍 언어에 내장되어 있습니다. 참조형(Reference Type)은 객체를 생성하고 동적 공간에 데이터를 둔 다음 이것을 참조하는 자료형을 말합니다 . 자바에서는 int, long, float, double 등 기본형과 String, Date와 같은 참조형을 모두 사용하지만 코틀린에서는 코딩할 때는 참조형만 사용하며 이것은 다시 코틀린의 성능 최적화에 따라 JVM에 실행하기 위해 코틀린 컴파일러에서 기본형으로 대체됩니다.

## 문자열 자료형

이제 문자열 자료형(String)에 대해 이야기 하겠습니다. 문자열 자료형은 문자 자료형에서 더 나아가 여러 문자를 배열하여 저장할 수 있는 자료형입니다. 그런데 왜 문자열 자료형은 따로 설명할까요? 문자형인 Char은 char과 같은 기본형으로 처리되지만 문자열 자료형은 기본형에 속하지 않는 배열 형태로 되어있는 특수한 자료형이기 때문입니다.

문자열은 힙 메모리 영역의 String Pool이라고 부르는 공간에 문자열을 저장해두고 이 값을 변수에서 참조합니다.

```kotlin
package chap02.section2

fun main() {
    var str1: String = "Hello"
    var str2 = "World"
    var str3 = "Hello"

    println("str1 === str2 ${str1 === str2}") //false
    //변수명만 다르고 참조 메모리 공간은 같다
    println("str1 === str3 ${str1 === str3}") //true
}
```

## 코틀린 null

코틀린은 기본적으로 데이터 타입에 null을 사용할 수 없다. 변수에 null 할당을 허용하려면 자료형 뒤에 물음표 기호(?)를 명시해야 합니다.

_세이프 콜이란 null이 할당되어 있을 가능성이 있는 변수를 검사하여 안전하게 호출하도록 도와주는 연산자로 사용할 변수 이름 뒤에 ?.를 작성하면 됩니다._

_non-null 단정 기호는 null이 아님을 단정하므로 컴파일러가 null검사 없이 무시합니다. 따라서 null이 할당되어 있을지라도 컴파일은 잘 진행되지만 실행 중에는 NPE를 발생시킵니다. 따라서 되도록이면 사용하지 않는것이 좋은데 반드시 널이 아니라는게 보장될 때만 사용_

```kotlin
fun main() {
    var str1: String
    //null 선언이 불가하다
    //str1= null
    //println(str1)

    var str2: String?
    str2 = null
    //?. 은 만약 null이면 뒷 부분은 실행하지 않는다. (세이프콜)
    println("str2 : $str2, length: ${str2?.length}")

        var str3: String?
    str3 = "Hello"
    //!! 은 뒷부분을 무조건 실행한다. non-null (보통 사용 안하는게 좋음)
    println("str3 : $str3, length: ${str3!!.length}")
}
```

### 세이프 콜과 엘비스 연산자를 활용해 null을 허용한 변수 더 안전하게 사용하기

null을 허용한 변수를 조금 더 안전하고 간단하게 사용하려면 세이프 콜 ?.와 엘비스(Elvis) 연산자 ?:를 함께 사용하면 됩니다. 엘비스 연산자는 변수가 null인지 아닌지 검사하여 null이 아니라면 왼쪽의 식을 그대로 실행하고 null이라면 오른쪽의 식을 실행합니다.

```kotlin
fun main() {
    var str4: String?
    str4 = null
    //elvis 형식 으로 str4가 null이 아니면 length값을 넘기고 null이면 -1을 넘긴다
    var len = str4?.length ?: -1
    println("str4 : $str4, length: $len") //str4 : null, length: -1

}
```
