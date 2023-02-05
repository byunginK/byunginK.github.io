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

두개 이상의 리소스가 리턴 됐을때 zip()을 사용하여 병합할 수 있다. <br/>
단 병합은 Mono, Flux에서 스트림안에서 병합되는 것임을 알아두자.

## Spring MVC + webClient 주의점

다른 API호출 시 논블러킹 방식으로 처리를 하게되면 메인쓰레드가 먼저종료가 되면서 호출에 대한 결과 완료와 상관없이 종료가된다. <br/>
따라서 메인 쓰레드에서 API 호출 쓰레드들이 처리가 완료가 될때까지 기다리도록 설정을 해주어야 한다.<br/>
그 방식은 `CountDownLatch`를 활용한다. `CountDownLatch`는 어느 쓰레드에서 다른 쓰레드에서 작업이 완료될 때까지 기다릴 수 있도록 지원해주는 클래스이다.

```java
CountDownLatch cdl = new CountDownLatch(2);

AtomicInteger result = new AtomicInteger();
List<Integer> resultList = new ArrayList<>();

webClient.get().uri("/test1")
                .retrieve()
                .bodyToMono(String.class)
                .doOnTerminate(() -> cdl.countDown())
                .subscribe(e -> result.set(e));

webClient.get().uri("/test2")
                .retrieve()
                .bodyToFlux(String.class)
                .doOnTerminate(() -> cdl.countDown())
                .subscribe(e -> resultList.add(e));

cdl.await();
```

### CountDownLatch 란

```java
CountDownLatch countDownLatch = new CountDownLatch(5);
```

위와 같이 생성할 수 있으며 인자로 Latch의 개수를 전달한다.<br/>

```java
countDownLatch.countDown();
```

`countDown()`은 클래스 생성시 전달 받은 Latch의 개수를 1개 씩 감소한다.

```java
countDownLatch.await();
```

`await()`는 Latch의 숫자가 0이 될 때까지 기다리는 코드이다.

### CountDownLatch 예시

쓰레드가 5개이고 5개의 쓰레드 작업이 완료될 때까지 기다리는 예제이다

```java

public class CountDownLatchExample {

    public static void main(String args[]) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(5);
        List<Thread> workers = Stream
                .generate(() -> new Thread(new Worker(countDownLatch)))
                .limit(5)
                .collect(toList());

        System.out.println("Start multi threads (tid: "
                + Thread.currentThread().getId() + ")");

        workers.forEach(Thread::start);

        System.out.println("Waiting for some work to be finished (tid: "
                + Thread.currentThread().getId() + ")");

        countDownLatch.await();

        System.out.println("Finished (tid: "
                + Thread.currentThread().getId() + ")");
    }

    public static class Worker implements Runnable {
        private CountDownLatch countDownLatch;

        public Worker(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            System.out.println("Do something (tid: " + Thread.currentThread().getId() + ")");
            countDownLatch.countDown();
        }
    }
}
```

##### 결과

> Start multi threads (tid: 1) <br/>
> Doing something (tid: 11) <br/>
> Doing something (tid: 12) <br/>
> Doing something (tid: 13) <br/>
> Doing something (tid: 14) <br/>
> Waiting for some work to be finished (tid: 1) <br/>
> Doing something (tid: 15) <br/>
> Finished (tid: 1) <br/>

생성된 5개의 쓰레드에서 `countDown()`을 통해 실행수 Latch를 감소하고 모든 Latch가 감소한 후 `countDownLatch`의 `await()`로 기다렸다가 종료되면서 메인 쓰레드에서 모든 작업을 마친다.

- 응용하여 아래와 같이 모든 쓰레드가 생성된 후 실행 되게끔 사용할 수 있다.

```java
public class CountDownLatchExample2 {

    public static void main(String args[]) throws InterruptedException {
        CountDownLatch readyLatch = new CountDownLatch(5);
        CountDownLatch startLatch = new CountDownLatch(1);
        CountDownLatch finishLatch = new CountDownLatch(5);
        List<Thread> workers = Stream
                .generate(() -> new Thread(new Worker(readyLatch,
                                    startLatch, finishLatch)))
                .limit(5)
                .collect(toList());

        System.out.println("Start multi threads (tid: "
                + Thread.currentThread().getId() + ")");

        workers.forEach(Thread::start); // 1.쓰레드 실행

        readyLatch.await(); // 3.readyLatch 감소까지 대기

        System.out.println("Waited for ready and started doing some work (tid: "
                + Thread.currentThread().getId() + ")");
        startLatch.countDown(); // 4.동시에 모든 쓰레드 시작을 위함 감소

        finishLatch.await(); // 7.종료 될때까지 대기 후 메인 쓰레드 진행
        System.out.println("Finished (tid: "
                + Thread.currentThread().getId() + ")");
    }

    public static class Worker implements Runnable {
        private CountDownLatch readyLatch;
        private CountDownLatch startLatch;
        private CountDownLatch finishLatch;

        public Worker(CountDownLatch readyLatch, CountDownLatch startLatch,
                      CountDownLatch finishLatch) {
            this.readyLatch = readyLatch;
            this.startLatch = startLatch;
            this.finishLatch = finishLatch;
        }

        @Override
        public void run() {
            readyLatch.countDown(); // 2.5개의 쓰레드가 시작되면서 준비된 readyLatch 하나 씩 감소
            try {
                startLatch.await(); // 5.메인에서 1개였던 startLatch가 감소되고 기다리던 이후작업 실행
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Do something (tid: "
                    + Thread.currentThread().getId() + ")");
            finishLatch.countDown(); // 6.작업 완료 후 finishLatch 하나씩 감소
        }
    }
}
```

### 정해진 시간만 기다리기(Timeout)

어느 쓰레드가 작업을 완료하지 못하면 `countDown()`호출이 되지 않아 메인 쓰레드에서 무한대기를 할 수 있다. <br/>
`await()`에 Timeout을 설정하면, 정해진 시간만 기다리도록 할 수 있다. <br/>
아래 코드는 5초를 Timeout을 설정한 것이다.

```java
await(5, TimeUnit.SECONDS)
```

메인 쓰레드에서 5개의 쓰레드를 생성 및 실행하고 생성된 쓰레드는 10초동안 기다리는 작업을 하고 `await()`에는 5초를 기다리고 종료하는 예시이다.

```java
public class CountDownLatchExample3 {

    public static void main(String args[]) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(5);
        List<Thread> workers = Stream
                .generate(() -> new Thread(new Worker(countDownLatch)))
                .limit(5)
                .collect(toList());

        System.out.println("Start multi threads (tid: "
                + Thread.currentThread().getId() + ")");

        workers.forEach(Thread::start);

        System.out.println("Waiting for some work to be finished (tid: "
                + Thread.currentThread().getId() + ")");

        countDownLatch.await(5, TimeUnit.SECONDS);

        System.out.println("Finished (tid: "
                + Thread.currentThread().getId() + ")");
    }

    public static class Worker implements Runnable {
        private CountDownLatch countDownLatch;

        public Worker(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            System.out.println("Doing something (tid: " + Thread.currentThread().getId() + ")");
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            countDownLatch.countDown();
            System.out.println("Done (tid: " + Thread.currentThread().getId() + ")");
        }
    }
}
```

##### 결과

> Start multi threads (tid: 1) <br/>
> Doing something (tid: 11) <br/>
> Doing something (tid: 12) <br/>
> Doing something (tid: 13) <br/>
> Doing something (tid: 14) <br/>
> Waiting for some work to be finished (tid: 1) <br/>
> Doing something (tid: 15) <br/>
> Finished (tid: 1) <br/>
> Done (tid: 12) <br/>
> Done (tid: 11) <br/>
> Done (tid: 14) <br/>
> Done (tid: 15) <br/>
> Done (tid: 13) <br/>

자세한 내용은 아래 링크 참고

- [리액터 메소드](https://wecandev.tistory.com/60)
- [사용 예제](https://github.com/toy-factory/bap-pool/tree/develop/bap-pool-server/src/main/java/com/toyfactory/bappool)
- [tuple이란?](https://projectreactor.io/docs/core/release/api/reactor/util/function/package-summary.html)
- [CountDownLatch](https://codechacha.com/ko/java-countdownlatch/)
