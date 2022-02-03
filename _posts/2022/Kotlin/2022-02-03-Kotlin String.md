---
layout: post
title: "Kotlin String"
date: 2022-02-03
categories: [Kotlin]
---

## 문자열

문자열은 불변(immutable) 값으로 생성되기 때문에 참조되고 있는 메모리가 변경될 수 없다. 새로운 값을 할당하려고 한다면 기존 메모리 이외에 새로운 문자열을 위한 메모리를 만들어 할당

만일 문자열에서 특정 범위의 문자열을 추출하기 위해 substring()이나 subsequence()를 사용해 특정 인덱스의 범위를 지정가능

```kotlin
var s = "abcdef"
s = s.substring(0..1) + "x" + s.substring(3..s.length-1) // ab를 추출하고 x를 덧붙이고 다시 def를 추출
```

a.compareTo(b) a와b가 같으면 0을 반환 a가 b보다 작으면 양수, 그렇지 않으면 음수

### StringBuilder 이용

StringBuilder를 사용하면 문자열이 사용할 공간을 좀 더 크게 잡을 수 있기 때문에 요소를 변경할 때 이 부분이 사용되어 특정 단어를 변경할 수 있게 됩니다. 단, 기존의 문자열보다는 처리가 좀 느리고, 만일 단어를 변경하지 않고 그대로 사용하면 임시 공간인 메모리를 조금 더 사용하게 되므로 낭비된다는 단점이 있다.

따라서 자주 변경되는 문자열일 경우에만 사용하도록하면 좋다.
