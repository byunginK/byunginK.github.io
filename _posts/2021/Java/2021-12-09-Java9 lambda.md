---
layout: post
title:  "Java9 lambda"
date:   2021-12-09
categories: [web]
---
# 람다 표현식

### 람다 표현식 이란?
람다 표현식(lambda expression)이란  "선언없이 실행가능한 함수" 입니다.

### 람다식 장단점
#### 장점
1. 불필요한 코드를 제거하여 간결하게 만들 수 있습니다.
2. 코드가 간결하고 식에 개발자의 의도가 명확히 드러나므로 가독성이 향상됩니다.
3. 함수를 만드는 과정없이 한번에 처리할 수 있기에 코딩시간이 단축됩니다.
4. 다중 cpu를 활용하는 형태로 구현되어 병렬 처리에 유리합니다.

#### 단점
1. 람다를 사용하면서 만드는 무명함수는 재사용이 불가능합니다.
2. 디버깅 시 함수 콜 스택 추적이 다소 어려움.
3. 재귀로 만들어 완전탐색하는 경우, 느릴수 있다.
4. 한 클래스내에 많은 람다식을 사용하는 경우, 오히려 가독성이 떨어진다.


### 람다식 작성법
1. 화살표(->) 기호를 사용하여 매개변수부와 선언부를 나눕니다.
2. 람다식 사용을 위해서는, 함수형 인터페이스에 접근하여 사용됩니다.
3. 매개변수의 타입을 추론할 수 있는 경우에는 타입을 생략할 수 있습니다.
4. 매개변수 작성부에서  변수가 하나인 경우에는 괄호(())를 생략할 수 있습니다.
5. 함수의 몸체가 하나의 명령문만으로 이루어진 경우에는 중괄호({})를 생략할 수 있습니다. (이때 세미콜론(;)은 붙이지 않음)
6. 함수의 몸체에 return 문이 있는 경우에는 중괄호({})를 생략할 수 없습니다.
7. return 문 대신 표현식을 사용할 수 있으며, 이때 반환값은 표현식의 결괏값이 됩니다. (이때 세미콜론(;)은 붙이지 않음)

> 함수형 인터페이스 선언

```java
package com.acompany.lambda;

@FunctionalInterface
public interface Calc {
    public int min(int x, int y);
}
```

> 함수형 인터페이스에 접근하여 람다식 사용

```java
package com.acompany.lambda;

public class LambdaTest3 {

    public static void main(String[] args) {
        //추상 메소드를 람다 표현식으로 구현
        Calc minNum = (x, y) -> x < y ? x : y;

        System.out.println(minNum.min(3,4));
    }
}
```

> 또는 기본 함수형 인터페이스를 상속 받아 선언 후 사용

```java
//comparator 인터페이스를 사용하기 위해 생성
static class AnimalComparator implements Comparator<String> {
    @Override
    public int compare(String o1, String o2) {
        return o1.length() - o2.length();
    }
}
```

> 생성자로 사용

```java
public static void main(String[] args) {
    String[] animals = {"cat","hippo","giraffe","elepahnt","monkey",""};

    // 기존 자바는 함수형 파라미터를 넘길 수 없기 때문에
    // 1. 아래와 같이 인터페이스 객체를 생성하여 파라미터로 제공
    AnimalComparator comparator = new AnimalComparator();
    Arrays.sort(animals,comparator);


    for (String item : animals){
        System.out.println(item);
    }
}
```

> 위 comparator을 람다식 표현하게 될 경우 아래와 같다.

```java
public static void main(String[] args) {
    String[] animals = {"cat","hippo","giraffe","elepahnt","monkey",""};

    //람다식 (o1, o2 파라미터는 동일한 타입으로 생략 가능)
    // 람다식은 단일 추상 메소드를 가진 인터페이스에 사용 가능 (함수형 인터페이스)
    Arrays.sort(animals,(o1, o2) -> o1.length() - o2.length());


    for (String item : animals){
        System.out.println(item);
    }
}
```

> `Predicate`함수형 인터페이스에서도 다음과 같이 사용할 수 있다.

```java
public static void main(String[] args) {
    List<String> animals = new ArrayList<>(
            Arrays.asList("cat","hippo","a","giraffe","elephant","monkey","")
    );

    animals.removeIf(s -> s.length() < 2);

    System.out.println(animals); //[cat, hippo, giraffe, elephant, monkey]
}
```

> 만약 람다식을 사용하지 않고 선언하여 사용할 경우

```java
public static void main(String[] args) {
    List<String> animals = new ArrayList<>(
            Arrays.asList("cat","hippo","a","giraffe","elephant","monkey","")
    );

    //객체로 파라미터 전달
    AnimalPredicate animalPredicate = new AnimalPredicate();
    animals.removeIf(animalPredicate);
    
    System.out.println(animals); //[cat, hippo, giraffe, elephant, monkey]
}

//인터페이스 객체를 파라미터로 전달 하기위해 생성
static class AnimalPredicate implements Predicate<String> {
    @Override
    public boolean test(String s) {
        return s.length() < 2;
    }
}

```