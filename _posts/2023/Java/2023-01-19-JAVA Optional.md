---
layout: post
title: "JAVA Optional 란? Optional 개념"
date: 2023-01-19
categories: [JAVA]
---

# Optional 사용 이유

개발을 할때 가장 많이 발생하는 예외중 하나인 NPE(NullPointerException). NPE를 피하기 위해 null 여부 검사를 하는데 코드가 복잡해지고 번거롭다.

## [Optoinal 이란?]

Optional<T> Wrapper 클래스를 활용하여 NPE를 방지하도록 도와준다. Optional 클래스의 value에 값을 저장하기 때문에 null이 바로 발생하지 않고, 각종 메소드를 제공하여 NPE방지에 도움을 준다.<br>
다만, Optional은 값을 Wrapping하고 다시 풀고, null 일 경우에는 대체하는 함수를 호출하는 등의 오버헤드가 있으므로 잘못 사용하면 시스템 성능저하에 영향을 준다. <br>
그렇기 때문에 메소드의 반환 값이 절대 null이 아니라면 Optional을 사용하지 않는 것이 좋다. _즉, Optional은 메소드의 결과가 null이 될 수 있으며, null에 의해 오류가 발생할 가능성이 매우 높을 때 반환값으로만 사용되어야 한다._ <br>
또한 Optional은 파라미터로 넘어가는 등이 아니라 반환 타입으로써 제한적으로 사용되도록 설계되었다.

## [Optional 사용법]

#### 1. Optional.empty() - 값이 없을 경우

값이 없을 경우 `Optional.empty()`로 생성할 수 있다.

```java
Optional<String> optional = Optional.empty();

System.out.println(optional); // Optional.empty
System.out.println(optional.isPresent()); // false
```

- 클래스 내부에 이미 Empty 객체를 생성하여 가지고 있으며 해당 객체를 공유하여 메모리를 절약한다.

#### 2. Optional.of() - 값이 null이 아닌 경우

만약 어떤 데이터가 절대 null이 아니라면 `Optional.of()`로 생성할 수 있다. (만약 null을 값으로 넣으면 NPE 발생)

#### 3. Optional.ofNullable() - 값이 Null일수도, 아닐수도 있는 경우

만약 어떤 데이터가 null일 수 도 있고 아닌 경우 사용. `Optional.ofNullalbe()`메소드 이후 `.orElse()`, `.orEsleGet()`메소드를 통해 값이 null일 경우에도 안전하게 값을 반환 할 수 있다.

```java
// getName()메소드의 반환값이 null일 수도 있고 값이 있을 수도 있다.
Optional<String> optional = Optional.ofNullable(getName());
String name = optional.orElse("anonymous"); // 값이 없다면 "anonymous" 를 리턴
```

## [Optional 예시]

기존에 null 체크를 하던 코드는 번거롭고 복잡해질 수 있으나 아래와 같이 간소화 될 수 있다.

```java
//기존 null 체크 1
List<String> names = getNames();
List<String> tempNames = list != null ? list : new ArrayList<>();

//Optional 사용하여 null 체크 1
List<String> nameList = Optional.ofNullable(getNames()).orElseGet(() -> new ArrayList<>());

//기존 null 체크 2
String name = getName();
String result = "";

try {
    result = name.toUpperCase();
} catch (NullPointerException e) {
    throw new CustomUpperCaseException();
}

//Optional 사용하여 null 체크 2
Optional<String> nameOpt = Optional.ofNullable(getName());
String result = nameOpt.orElseThrow(CustomUpperCaseExcpetion::new).toUpperCase();
```

## [Optional의 orElse, orElseGet 차이]

- orElse: 파라미터를 값으로 받는다.
- orElseGet: 파라미터를 함수형 인터페이스로 받는다.

```java
public void findUserEmailOrElse() {
    String userEmail = "kkk";
    String result = Optional.ofNullable(userEmail)
    	.orElse(getUserEmail());

    System.out.println(result);
}

public void findUserEmailOrElseGet() {
    String userEmail = "kkk";
    String result = Optional.ofNullable(userEmail)
    	.orElseGet(this::getUserEmail);

    System.out.println(result);
}

private String getUserEmail() {
    System.out.println("getUserEmail() Called");
    return "ttt";
}

```

위 함수를 각각 실행하면 아래와 같다.

```java
// 1. orElse인 경우
getUserEmail() Called
kkk

// 2. orElseGet인 경우
kkk
```

### orElse 의경우 처리 순서

1. Optional.ofNullable로 "kkk"를 갖는 Optional 객체 생성
2. `getUserEmail()` 실행후 반환값을 orElse에 전달
3. orElse 호출. null이 아니므로 "kkk" 반환

### orElseGet 의경우 처리 순서

1. Optional.ofNullable로 "kkk"를 갖는 Optional 객체 생성
2. `getUserEmail()` 함수를 orElseGet에 전달
3. orElseGet 호출. null이 아니므로 "kkk"를 반환 (`getUserEmail()`호출 X)

### orElse의 잘못된 사용

```java
public void findByUserEmail(String userEmail) {
    // orElse에 의해 userEmail이 이미 존재해도 유저 생성 함수가 호출되어 에러 발생
    return userRepository.findByUserEmail(userEmail)
            .orElse(createUserWithEmail(userEmail));
}

private String createUserWithEmail(String userEmail) {
    User newUser = new User(userEmail);
    return userRepository.save(newUser);
}
```

유니크한 키를 가지는 user의 상황에서 userEmail이 null이 아닌 상황에도 `orElse`로인해 또 user를 생성하는 함수를 호출하게 되어 에러가 발생한다.
따라서 아래와 같이 코드를 수정하여 사용해야한다.

```java
public void findByUserEmail(String userEmail) {
    // orElseGet에 의해 파라미터로 함수를 넘겨주므로 Null이 아니면 유저 생성 함수가 호출되지 않음
    return userRepository.findByUserEmail(userEmail)
           .orElseGet(createUserWithEmail(userEmail));
}
```

# Optional 사용 가이드

Optional을 사용하면 코드가 Null-Safe해지고, 가독성이 좋아지며 애플리케이션이 안정적이 된다는 등과 같은 얘기들을 많이 접할 수 있다. 하지만 이는 Optional을 목적에 맞게 올바르게 사용했을 때의 이야기이고, Optional을 남발하는 코드는 오히려 다음과 같은 부작용(Side-Effect)를 유발할 수 있다.

- NullPointerException 대신 NoSuchElementException가 발생함
- 이전에는 없었던 새로운 문제들이 발생함
- 코드의 가독성을 떨어뜨림
- 시간적, 공간적 비용(또는 오버헤드)이 증가함

## [올바른 사용법]

- Optional 변수에 Null을 할당하지 말아라
- 값이 없을 때 Optional.orElseX()로 기본 값을 반환하라
- 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라
- 생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라
- Collection의 경우 Optional이 아닌 빈 Collection을 사용하라
- 반환 타입으로만 사용하라

### Optional 변수에 Null을 할당하지 말아라

클래스에 null을 사용하지 말고 Optional.empty()로 사용.

```java
public Optional<Cart> fetchCart() {

    Optional<Cart> emptyCart = null; // X
    Optional<Cart> emptyCart = Optional.empty(); // O
    ...
}
```

### 값이 없을 때 Optional.orElseX()로 기본 값을 반환하라

isPresent()로 검사하고 get()으로 꺼내지 말고 , orElseGet 등을 활용한다.

```java
// AVOID
public String findUserName(long id) {

    Optional<String> optionalName = ... ;

    if (optionalName.isPresent()) {
        return optionalName.get();
    } else {
        return findDefaultName();
    }
}

// PREFER
public String findUserName(long id) {

    Optional<String> optionalName = ... ;
    return optionalName.orElseGet(this::findDefaultName);
}

private String findDefaultName() {
    return ...;
}
```

rElseGet은 값이 준비되어 있지 않은 경우, orElse는 값이 준비되어 있는 경우에 사용하면 된다. 만약 null을 반환해야 하는 경우라면 orElse(null)을 활용하도록 하자. 만약 값이 없어서 throw해야하는 경우라면 orElseThrow를 사용하면 되고 그 외에도 다양한 메소드들이 있으니 적당히 활용하면 된다.

### 단순히 값을 얻으려는 목적으로만 Optional을 사용하지 마라

단순히 값을 얻으려고 Optional을 사용하는 것은 Optional을 남용하는 대표적인 경우이다. 이럴 땐 직접 값을 다루자.

```java
// AVOID
public String findUserName(long id) {
    String name = ... ;

    return Optional.ofNullable(name).orElse("Default");
}

// PREFER
public String findUserName(long id) {
    String name = ... ;

    return name == null
      ? "Default"
      : name;
}
```

### 생성자, 수정자, 메소드 파라미터 등으로 Optional을 넘기지 마라

Optional을 파라미터로 넘기는 것은 상당히 의미없는 행동이다. 왜냐하면 넘겨온 파라미터를 위해 자체 null체크도 추가로 해주어야 하고, 코드도 복잡해지는 등 상당히 번거로워지기 때문이다. Optional은 반환 타입으로 대체 동작을 사용하기 위해 고안된 것임을 명심해야 하며, 앞서 설명한대로 Serializable을 구현하지 않으므로 필드 값으로 사용하지 않아야 한다.

### Collection의 경우 Optional이 아닌 빈 Collection을 사용하라

Collection의 경우 굳이 Optional로 감쌀 필요가 없다. 오히려 빈 Collection을 사용하는 것이 깔끔하고, 처리가 가볍다.

```java
// AVOID
public Optional<List<User>> getUserList() {
    List<User> userList = ...; // null이 올 수 있음
    ...
    return Optional.ofNullable(items);
}

// PREFER
public List<User> getUserList() {
    List<User> userList = ...; // null이 올 수 있음
    ...
    return items == null ? Collections.emptyList() : userList;
}

// AVOID
public Map<String, Optional<String>> getUserNameMap() {
    Map<String, Optional<String>> items = new HashMap<>();
    items.put("I1", Optional.ofNullable(...));
    items.put("I2", Optional.ofNullable(...));

    Optional<String> item = items.get("I1");

    if (item == null) {
        return "Default Name"
    } else {
        return item.orElse("Default Name");
    }
}

// PREFER
public Map<String, String> getUserNameMap() {
    Map<String, String> items = new HashMap<>();
    items.put("I1", ...);
    items.put("I2", ...);

    return items.getOrDefault("I1", "Default Name");
}
```

### 반환 타입으로만 사용하라

Optional은 반환 타입으로써 에러가 발생할 수 있는 경우에 결과 없음을 명확히 드러내기 위해 만들어졌으며, Stream API와 결합되어 유연한 체이닝 api를 만들기 위해 탄생한 것이다.

### 참고 링크 : https://mangkyu.tistory.com/203
