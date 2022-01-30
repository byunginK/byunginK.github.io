---
layout: post
title:  "Java9 Immutable collections"
date:   2021-12-07
categories: [web]
---
# Java9의 불변 컬렉션
좀 더 간결한게 불변 컬렉션을 생성할 수 있도록 Java9에서 List, Set, Map 인터페이스에 새로운 팩토리 메서드들이 추가되었습니다.

### 불변 컬렉션이란?
불변(Immutable) 컬렉션(Collection)은 아이템 추가, 수정, 제거가 불가능합니다. 따라서 신규 아이템을 추가하거나 기존 아이템을 수정또는 제거하려고 하면 `java.lang.UnsupportedOperationException`이 발생합니다. 컬렉션이 생성된 후에 변경되기를 원하지 않는 경우에 사용하며, 의도치 않은 컬렉션 변경을 예방에 도움이 됩니다.

### 불변 컬렉션 생성의 번거로움
Java8까지는 불변(Immutable) 리스트를 만들기 위해서는 가변(mutalbe) 리스트를 먼저 만들고 `Collections` 클래스의 `unmodifiableList()` 정적 메서드를 사용하여 불변 리스트로 변환시켜줘야 했었습니다.
```java
List<String> fruits = new ArrayList<>();

fruits.add("Apple");
fruits.add("Banana");
fruits.add("Cherry");
fruits = Collections.unmodifiableList(fruits);

fruits.add("Lemon"); // UnsupportedOperationException
```
위와 같은 코드를 작성하는 것이 크게 불편하지는 않지만 아무래도 불변 컬렉션을 생성할 일이 잦아지게 되면 상당히 번거롭게 느껴질 수 있습니다.

## Java8의 해결 방법
아직까지 Java9을 사용할 수 없는 개발 환경이 많을 것이기 때문에, Java8까지는 위와 같이 불변 리스트 생성을 위해 불필요하게 길어지는 코드를 어떻게 회피했었는지 먼저 살펴 보겠습니다.

### Stream API
두번째 방법은 `Stream` API를 활용하는 것입니다.
```java
List<String> fruits = Stream
    .of("Apple", "Banana", "Cherry")
    .collect(collectingAndThen(toList(), Collections::unmodifiableList));
```

### Guava 라이브러리
Google Guava 라이브러리를 이용하는 것입니다. Guava는 불변 리스트 생성을 단 한 줄의 코드로 작성할 수 있도록 팩토리 메서드나 빌더 클래스를 제공합니다.
```java
import com.google.common.collect.ImmutableList;

List<String> fruits = ImmutableList.of("Apple", "Banana", "Cherry");

fruits.add("Lemon"); // UnsupportedOperationException
```

## Java9의 불변 컬랙션 생성
Java9에서는 불변 컬렉션 생성을 위한 팩토리 메서드들이 추가되어 Guava 라이브러리와 비슷한 방식으로 사용할 수 있습니다.

### List
먼저 `List` 인터페이스에는 `of()` 메서드가 추가되었습니다.
```java
List<String> fruits = List.of("Apple", "Banana", "Cherry"); // [Apple, Banana, Cherry]

fruits.add("Lemon"); // UnsupportedOperationException
```
비어있는 불편 리스트를 만드려면 팩토리 메서드에 아무 인자도 넘기지 않으면 됩니다.
```java
List<String> fruits = List.of(); // []
```

### Set
`List` 뿐만 아니라 `Set`과 `Map` 인터페이스에도 불편 리스트 생성을 위한 팩토리 메서드가 추가되었습니다.
```java
Set<String> fruits = Set.of("Apple", "Banana", "Cherry"); // [Banana, Apple, Cherry]
```
`Set` 인터페이스의 `of()` 메서드에 중복 인자를 넘기면 `IllegalArgumentException` 예외가 발생하니 주의바랍니다.
```java
Set<String> fruits = Set.of("Apple", "Banana", "Cherry", "Apple"); // IllegalArgumentException
```

### Map
`Map` 인터페이스에는 키와 값을 번갈아 넘길 수 있는 `of` 메서드와 키와 값을 묶어서 넘길 수 있는 `ofEntries()` 메서드가 추가되었습니다.

<b>- Map.of()</b>

```java
Map<Integer, String> fruits = Map.of(1, "Apple", 2, "Banana", 3, "Cherry"); // {1=Apple, 2=Banana, 3=Cherry}

fruits.put(4, "Lemon"); // UnsupportedOperationException
```

<b>- Map.ofEntries()</b>

```java
Map<Integer, String> fruits = Map.ofEntries(Map.entry(1, "Apple"), Map.entry(2, "Banana"), Map.entry(3, "Cherry"));
```