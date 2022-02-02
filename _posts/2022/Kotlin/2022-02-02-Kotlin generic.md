---
layout: post
title: "Kotlin generic"
date: 2022-02-02
categories: [Kotlin]
---

## 제네릭

형식 매개변수를 받는 함수나 매서드를 제네릭 함수 혹은 메서드라고 하며 해당 함수나 메서드 앞쪽에 <T>와 같이 지정합니다. <K, V> 와 같이 형식 매개변수를 여러 개 사용할 수 있다.

```kotlin
// 매개변수와 반환 자료형에 형식 매개변수 T가 사용됨
fun <T> genericFunc(arg: T): T? { ... }

// 형식 매개변수가 여러 개인 경우
fun <K, V> put(key: K, value: V): Unit { ... }
```

## 가변성의 3가지 유형

![image](https://user-images.githubusercontent.com/65350890/152153409-71f05a19-c2d3-4891-9839-577dfff0154a.png)

먼저 간단히 설명하면 기존의 Int 클래스는 Number 클래스의 하위 클래스입니다. 제네릭에서는 class Box<T>와 같은 경우에는 Box<Number>와 Box<Int>에는 아무런 관련이 없는 무변성입니다. 이제 생산자 입장의 out을 사용해 class Box<out T>로 정의되면 Box<Int>는 Box<Number>의 하위 자료형이 됩니다. 반대로 소비자 입장의 in을 사용하면 Box<Number>가 Box<Int>의 하위 자료형이 되고 반공변성이라고 부릅니다.

### 가변성 지정 방법

*선언 지점 변성(declaration-site variance)*이란 클래스를 선언하면서 클래스 자체에 가변성을 지정하는 방식으로 클래스에 in/out을 지정할 수 있다.

선언하면서 지정하면 클래스의 공변성을 전체적으로 지정하는 것이 되기 때문에 클래스를 사용하는 장소에서는 따로 자료형을 지정해 줄 필요가 없다.

```kotlin
class Box<in T: Animal>(var item: T)
```

*사용 지점 변성(use-site variance)*은 메서드 매개변수에서 또는 제네릭 클래스를 생성할 때와 같이 사용 위치에서 가변성을 지정하는 방식

형식 매개변수가 있는 자료형을 사용할 때마다 해당 형식 매개변수를 하위 자료형이나 상위 자료형 중 어떤 자료형으로 대체할 수 있는지를 명시

```kotlin
class Box<T>(var item: T)

...

fun <T> printObj(box: Box<out Animal>) {
    val obj: Animal = box.item // item의 값을 얻음(get)
    println(obj)
}
```
