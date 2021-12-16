---
layout: post
title:  "Java9 stream & Optional"
date:   2021-12-09
categories: [web]
---
# Stream & Optional Class
기존에 컬렉션을 처리할 때 요소를 순환하면서 하나씩 꺼내서 다루었다. 간단한 경우면 괜찮지만 로직이 복잡하고 코드의 양이 많아지면 가독성, 유지보수 등의 어려움이 있다.
스트림은 연산을 구현체에 맡긴다. 또한, 람다식을 이용하여 코드 양을 줄이고 여러 함수를 조합하여 원하는 결과를 필터링 하고 가공할 수 있다. 그리고 성능적으로도 *병렬처리(multi-threading)*
이 가능하여 빠르게 처리할 수 있다.

### stream의 특징
1. 스트림은 요소를 저장하지 않는다. 요소는 스트림을 지원하는 컬렉션에 저장하거나 필요할 때 생성한다.
2. 스트림 연산은 원본을 변경하지 않는다. 예를 들어 filter 메서드는 스트림에서 요소를 지우는 것이 아닌, 필터링이 된 요소만 있는 스트림을 새로 생성한다.
3. 스트림 연산은 **지연(lazy)** 방식으로 작동한다. 즉 필요하기 전까지는 연산 결과를 실행하지 않는다.

### stream의 작업 흐름
파이프라인으로 작업이 진행된다.
1. 스트림 생성
2. 초기 스트림을 다른 스트림으로 변환하는 **중간 연산**을 지정한다. (여러 단계로 지정 가능)
3. **종료 연산**을 통해 결과를 산출한다. (종료 연산 수행 후에는 해당 스트림을 더는 사용할 수 없다.)


### stream 생성
#### - 배열 스트림
`Arrays.stream` 메소드 사용, 정적 메소드인 `stream.of`를 사용해서 만들 수 있다.

```java
//Arrays.stream
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);

//stream.of (split은 String[] 배열 반환)
Stream<String> words = Stream.of(contents.split(","));
```

#### - 컬렉션 스트림
`stream` 메소드 사용 , 일반적으로 스트림을 만들때 해당 메소드를 사용한다.

```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 병렬 처리 스트림
```

#### - 무한 스트림
1. `generate` 메소드
`Supplier<T>(리턴값만 있는 함수형 인터페이스)`에 해당하는 람다로 값을 무한정 넣을 수 있다.

```java
//limit을 통해 특정 사이즈 제한
Stream<Double> randoms = Stream.generate(Math::random).limit(5);
```

2. `ilterate` 메소드
`UnaryOperator<T>`에 해당하는 람다로 값을 반복적 적용

```java
Stream<Integer> iteratedStream = 
  Stream.iterate(30, n -> n + 2).limit(5); // [30, 32, 34, 36, 38]
```

#### - 파일 스트림
java nio `Files`클래스의 `lines`메소드를 통해 스트링 타입의 스트림 생성 가능하다.

```java
Path path = Paths.get("cities.txt");

Stream<String> stringStream = Files.lines(path);
```

### stream 가공 (중간 연산)
중간 연산을 통해 원하는 요소만 뽑아낼 수 있으며, 해당 작업은 `stream`을 리턴하기 때문에 여러 작업을 이어서 (chaining) 작성할 수 있다.

#### 1. filter
필터(filter)은 스트림 내 요소들을 하나씩 평가해서 걸러내는 작업입니다. 인자로 받는 Predicate 는 boolean 을 리턴하는 함수형 인터페이스로 평가식이 들어가게 됩니다.

```java
//스트림의 각 요소에 대해서 평가식을 실행하게 되고 ‘a’ 가 들어간 이름만 들어간 스트림이 리턴됩니다.
Stream<String> stream = names.stream().filter(name -> name.contains("a"));
// [Elena, Java]
```
#### 2. Map
맵(map)은 스트림 내 요소들을 하나씩 특정 값으로 변환 후 새로운 스트림으로 리턴 해줍니다. 이 때 값을 변환하기 위한 람다를 인자로 받습니다.

```java
//stringStream은 string타입의 스트림
//map을 통해 ","로 나누기 -> 공백 제거 -> 정수로 변환 한 값으로 변환 하였다.
Stream<String> stream = stringStream.map(l -> Integer.parseInt(l.split(",")[0].trim()));
```

위의 예제 처럼 간단한 함수의 경우 람다식 안에 작성하여도 되며 함수가 길거나 함수형 인터페이스를 사용할 경우 따로 선언 및 불러서 바로 사용할 수 있다.

#### 3. sort
정렬의 방법은 다른 정렬과 마찬가지로 Comparator 를 이용합니다.

인자 없이 그냥 호출할 경우 오름차순으로 정렬합니다.

```java
List<String> lang = 
  Arrays.asList("Java", "Scala", "Groovy", "Python", "Go", "Swift");

lang.stream()
  .sorted()
  .collect(Collectors.toList());
// [Go, Groovy, Java, Python, Scala, Swift]

lang.stream()
  .sorted(Comparator.reverseOrder())
  .collect(Collectors.toList());
// [Swift, Scala, Python, Java, Groovy, Go]
```

#### 4. 서브스트림 메소드
1. `limit(n)` 무한 스트림을 원하는 크기로 자를 때 특히 유용하다.
2. `skip(n)` 처음 n개의 요소를 버린다.
3. `takewhile(*predicate*)` 프레디케이트가 참인 동안 모든 요소를 가져온다. <-> `dropWhile(*predicate*)`조건이 거짓인 요소만 가져온다.
4. `concat()` 두 스트림을 연결한다. (첫 번째 스트림은 무한 스트림이 될 수 없다.)
5. `distinct()` 중복요소 제외한 스트림 반환


### stream 결과 (종료 연산)
결과 메소드는 이후 설명할 Optional과 관계가 있다. 결과가 null일 경우 Optional<T> 클래스를 이용하여 예외 상황을 방어할 수 있다.
#### 1. Calculating
- `count`
- `sum`
위 두개의 메소드는 스트림이 비어있을 경우 0을 반환 하지만, 평균, 최소, 최대의 경우 null로 반환되기 때문에 Optional<T>를 이용하여 값을 얻는다.

```java
long count = IntStream.of(1, 3, 5, 7, 9).count();
long sum = LongStream.of(1, 3, 5, 7, 9).sum();

OptionalInt min = IntStream.of(1, 3, 5, 7, 9).min();
OptionalInt max = IntStream.of(1, 3, 5, 7, 9).max();

//ifPresent()를 사용하여 출력 가능하다.
DoubleStream.of(1.1, 2.2, 3.3, 4.4, 5.5).average().ifPresent(System.out::println);


Optional<Integer> integer =
                // map을 통해 가공을 한다. 라인의 첫번째 숫자만 가져옴 (요소의 갯수는 변화 없음)
                stringStream.map(l -> Integer.parseInt(l.split(",")[0].trim()))
                        //filter를 통해 특정 범위의 숫자를 가져옴
                        .filter(p -> p >= 500)
                        .reduce((a, b) -> a + b);

//orElse()는 Optional<T>의 메소드로 null일 경우 0을 반환한다.
int data = integer.orElse(0);
        System.out.println(data);                        
```

#### 2. reduce
스트림에서 값을 계산하는 일반적인 메커니즘이다. 단순한 형태로는 이항 함수적용이다.

```java
List<Integer> values = Arrays.asList(1, 2, 3);
Optional<Integer> sum = values.stream().reduce((x, y) -> x + y); // reduce(Integer::sum) 도 가능
System.out.println(sum.get()); // 6
```

두개의 인자를 받을 수 있으며, 예제는 아래와 같다.
```java
int reducedTwoParams = 
  IntStream.range(1, 4) // [1, 2, 3]
  .reduce(10, Integer::sum); // method reference

  //결과는 16 (10 + 1 + 2 + 3)
```

#### 3. collect
1. `forEach()` 요소를 임의의 순서로 순회하여 결과를 모은다. 
2. Collector 인터페이스의 인스턴스를 받는 `collect()`메소드를 사용

```java
//리스트
List<String> result = stream.collect(Collectors.toList());
//집합
Set<String> result = stream.collect(Collectors.toSet());
```
3. `joining()` 스트림에 있는 모든 문자열을 연결해서 모으고 싶을때

```java
String result = stream.collect(Collectors.joining());

//구분자를 추가할 수 있다.
String result2 = stream.collect(Collectors.joining(","));
```
4. `summarizingInt()` 합계, 카운트, 평균, 최댓값, 최솟값으로 리듀스

```java
IntSummaryStatistics summary = values.stream().collect(
        Collectors.summarizingInt(String::length)
);
summary.getAverage();
summary.getSum();
summary.getMax();
```

#### 4. map
키와 값으로 만들어 내는 함수 두개를 인주로 전달 받는다.

```java
Map<Integer, String> idToName = people.collect(Collectors.toMap(Person::getId, Person::getName));
```

#### 5. matching

매칭은 조건식 람다 Predicate 를 받아서 해당 조건을 만족하는 요소가 있는지 체크한 결과를 리턴합니다. 다음과 같은 세 가지 메소드가 있습니다.

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java");

boolean anyMatch = names.stream()
  .anyMatch(name -> name.contains("a"));
boolean allMatch = names.stream()
  .allMatch(name -> name.length() > 3);
boolean noneMatch = names.stream()
  .noneMatch(name -> name.endsWith("s"));
```

참고 : [https://futurecreator.github.io/2018/08/26/java-8-streams/](https://futurecreator.github.io/2018/08/26/java-8-streams/)

stream에 대한 추가 고급 내용 : [https://futurecreator.github.io/2018/08/26/java-8-streams-advanced/](https://futurecreator.github.io/2018/08/26/java-8-streams-advanced/)