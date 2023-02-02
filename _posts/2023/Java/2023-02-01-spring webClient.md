---
layout: post
title: "Spring WebClient"
date: 2023-02-01
categories: [JAVA, Spring]
---

# Spring webClient

WebClient는 RestTemplate를 대체하는 HTTP 클라이언트이다. RestTemplate는 추후 스프링에서 더이상 지원하지 않는다. <br/>
WebClient에는 기존의 동긱 API를 제공할 뿐만 아니라 논블러킹 및 비동기 접근 방식을 지원하여 효율적인 통신이 가능하다.<br/>
이러한 통신은 최근 MSA 구조에서 서비스간의 통신을 더 원할하게 하며, API간의 통신에 더 효율적으로 자원을 사용할 수 있게 해준다.<br/>

빌더 방식의 인터페이스이며, 리액티브 타입의 전송과 수신을 한다. (Mono, Flux)<br/>

### 제어권 반환

1. Blocking<br/>
   Application이 커널로 작업 요청을 할때, 커널은 요청에대한 로직을수행한다. 이때, 요청에 대한 응답을 받을 때 까지 대기를 한 후 작업을 진행하는 방식이다.

2. Non-blocking<br/>
   Application이 커널로 작업 요청을 한후 바로 제어권을 받는다. 커너을 요청에대한 응답을 계속 처리하고 Application은 다른 작업을 수행한다.

### synchronouse(동기) vs Asynchronouse(비동기)

동기 비동기는 결과를 어떻게 받아서 처리하는지에 대한 부분이다. 일반적으로 동기방식과 블로킹방식으로 요청 후 커널로부터 응답을 대기한 후 이후 작업을 처리한다. <br/>
비동기 논블러킹은 커널에 요청 후 바로 다른 작업을 처리하고 있다가 커널에서 콜백이 오게되면 그때 응답의 결과를 받고 작업을 진행하는 방식이다.

![image](https://user-images.githubusercontent.com/65350890/216325533-41cdf116-dec2-4528-9538-f0efd6c0dd43.png)

1. sync-blocking<br/>
   application -> kernel로 요청 -> 대기 -> kernel 작업완료 -> application 결과 전달

2. sync-nonblocking<br/>
   application -> kernel로 요청 -> application 다른작업 (Thread가 지속 확인 polling) -> kernel 작업 중 -> application 다른작업 (Thread가 지속 확인 polling) -> kernel 작업 완료 후 결과 전달

3. async-blocking (가장 비효율)<br/>
   application -> kernel로 요청 -> 대기 -> kernel 작업완료 -> application 결과 전달

4. async-nonblocking<br/>
   application -> kernel로 요청 -> application 다른작업 -> kernel 작업 완료 후 callback -> application 결과 수신

## 의존성

```
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-webflux'
}
```

## 생성

1. create()
2. build()

```java
WebClient.create("http://localhost:8080");


WebClient client = WebClient.builder()
  .baseUrl("http://localhost:8080")
  .defaultCookie("cookieKey", "cookieValue")
  .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
  .defaultUriVariables(Collections.singletonMap("url", "http://localhost:8080"))
  .build();
```

빌드할 시 immutable하여 옵션 변경시 mutate()메소드를 사용해야한다.

```java
WebClient client1 = WebClient.builder()
        .filter(filterA).filter(filterB).build();

WebClient client2 = client1.mutate()
        .filter(filterC).filter(filterD).build();

```

## GET

1. Flux (리소스 모음)

```java
@Autowired
WebClient webClient;

public Flux<Employee> findAll() {
	return webClient.get()
		.uri("/employees")
		.retrieve()
		.bodyToFlux(Employee.class);
}
```

2. Mono (단일 리소스)

```java
@Autowired
WebClient webClient;

public Mono<Employee> findById(Integer id) {
	return webClient.get()
		.uri("/employees/" + id)
		.retrieve()
		.bodyToMono(Employee.class);
}

```

## POST

```java
@Autowired
WebClient webClient;

public Mono<Employee> create(Employee empl) {
	return webClient.post()
		.uri("/employees")
		.body(Mono.just(empl), Employee.class)
		.retrieve()
		.bodyToMono(Employee.class);
}
```

body에는 파라미터및 클래스정의를한다. bodyToMono는 `Mono<Employee>`반환타입에 맞게 클래스를 정의해주면된다.

## Response

retrieve() 와 exchange()가 있으며, retrieve()는 body를 받아 디코딩하는 간단한 메소드이며, exchange()는 세세하게 컨트롤이 가능하지만 메모리 누수 가능성때문에 지양한다.

## 처리 방식

- blocking 방식 : .block()
- non-blocking 방식 : .subscribe()를 통해 callback함수 지정

```java
// blocking
Mono<Employee> employeeMono = webClient.get(). ...
employeeMono.block()

// non-blocking
Mono<Employee> employeeFlux = webClient.get(). ...
employeeFlux.subscribe(employee -> { ... });

```

## 묶음

두개 이상의 리소스가 리턴 됐을때 zip()을 사용하여 병합할 수 있다.
자세한 내용은 아래 링크 참고

- [리액터 메소드](https://wecandev.tistory.com/60)
- [사용 예제](https://github.com/toy-factory/bap-pool/tree/develop/bap-pool-server/src/main/java/com/toyfactory/bappool)
- [tuple이란?](https://projectreactor.io/docs/core/release/api/reactor/util/function/package-summary.html)
