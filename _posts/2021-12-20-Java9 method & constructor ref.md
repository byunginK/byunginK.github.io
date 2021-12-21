---
layout: post
title:  "Java9 method & constructor Ref"
date:   2021-12-20
categories: [web]
---
# Method & Constructor Reference

### Method Reference
메서드를 호출하는 람다 표현식보다 간결하게 사용가능하게 참조가 가능한 문법

아래와 같이 문자열을 정렬하는 람다식이 있다.

```java
Arrays.sort(strings, (x,y) -> x.compareToIgnoreCase(y));
```

위의 코드 대신 다음 메서드 표현식으로 작성 할 수 있다.

```java
Arrays.sort(strings, String::compareToIgnoreCase);
```

따라서 다음 컬렉션(List)에서 object 클래스의 isNull 메서드를 활용 할 수 있다.

```java
//null인 값은 제거하고 list에 남김
list.removeIf(Object::isNull);
```

`::` 연산자는 클래스 이름과 메서드 이름을 분리하거나 객체의 이름과 메서드 이름을 분리한다. 다음 세가지 형태로 사용 가능하다.

1. Class::instanceMethod
2. Class::staticMethod
3. object::instanceMethod

### Constructor Reference

메서드 참조자에서 메서드의 이름 대신 new 를 입력하면 된다. 예를 들어 Employee::new는 Employee 클래스 생성자 참조이다.

```java
// Employee 객체로 map 함수 적용하여 결과를 stream으로 모아준다.
Stream<Employee> stream = names.stream().map(Employee::new);
```

