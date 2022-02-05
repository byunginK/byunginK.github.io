---
layout: post
title: "Kotlin coroutine"
date: 2022-02-05
categories: [Kotlin]
---

## Thread

코루틴 이전에 기존 자바에서 사용하던 쓰레드 생성 방식은 코틀린에서도 사용 가능하며, 아래와 같다.

```kotlin
//클래스
class SimpleThread : Thread() {
    override fun run() {
        println("Class Thread ${Thread.currentThread()}")
    }
}


//인터페이스 (장점 : 다른 인터페이스에 상속가능)
class SimpleRunnable : Runnable {
    override fun run() {
        println("interface Thread ${Thread.currentThread()}")
    }

}


fun main() {
    //클래스
    val thread = SimpleThread()
    thread.start()

    //인터페이스
    val runnable = SimpleRunnable()
    val thread2 = Thread(runnable)
    thread2.start()

    //익명객체
    object : Thread(){
        override fun run() {
            println("object Thread: ${Thread.currentThread()}")
        }
    }.start()

    //람다식
    Thread{
        println("Lambda Thread: ${Thread.currentThread()}")
    }.start()
}
```

## 코루틴

- launch : 일단 실행하고 잊어버리는 형태의 코루틴 (메인 프로그램과 독립), 실행 결과 반환 X , 상위 코드를 넌블로킹관리를 위해 job 객체를 즉시 반환, join을 통해 상위코드가 종료되지 않고 완료를 기다리게 할 수 있다.

- async : 비동기 호출 코루틴으로 결과나 예외 반환, 결과를 await을 통해 받을 수 있다. await은 작업 완료까지 기다린다.

프로세스나 스레드는 해당 작업을 중단(stopped)하고 다른 루틴을 실행하기 위한 문맥 교환을 시도할 때 많은 비용이 듭니다. 코루틴은 문맥 교환 없이 해당 루틴을 일시 중단(suspended)해서 이러한 비용을 줄일 수 있습니다. 다르게 표현하면 운영체제가 스케줄링에 개입하는 과정이 필요하지 않다는 것입니다. 또한 일시 중단은 사용자가 제어할 수 있다.

### 지연(suspend)

먼저 코루틴에서 사용되는 함수는 suspend로 선언된 지연 함수여야 코루틴 기능을 사용할 수 있습니다. suspend로 표기함으로서 이 함수는 실행이 일시 중단(suspended)될 수 있으며 필요한 경우에 다시 재개(resume)할 수 있게 된다.

```kotlin
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

fun main() {
    GlobalScope.launch {
        delay(1000L) //지연 함수 (넌블로킹)
        println("World")
        doSomething()
    }
    println("Hello")
    Thread.sleep(2000L) //쓰레드가 없으면 쓰레드가 바로 종료되어 코루틴이 실행되지 않음
}

suspend fun doSomething(){ //코루틴 안에서 사용할 수 있는 함수
    println("Do Something")
}
```

### job

Job은 백그라운드에서 실행하는 작업을 가리킵니다. 개념적으로는 간단한 생명 주기를 가지고 있고 부모-자식 관계가 형성되면 부모가 작업이 취소될 때 하위 자식의 작업이 모두 취소. 자식의 실패는 부모에 전달되어 부모도 실패. (SupervisorJob을 사용하면 실패 전달 안됨)

### join()

job 객체의 join()을 사용해 완료를 기다릴 수 있다.

```kotlin
fun main() {
    runBlocking { //코루틴이며 블록안에 실행될때까지 기다려줌
        val job = GlobalScope.launch {
            delay(1000L) //지연 함수
            println("World")
            doSomething()
        }
        println("Hello")
        println("job : ${job.isActive}, ${job.isCompleted}")
        job.join() //job을 기다린다. (아까 thread.sleep을 대체)
        println("job : ${job.isActive},${job.isCompleted}")
    }
}

suspend fun doSomething() { //코루틴 안에서 사용할 수 있는 함수
    println("Do Something")
}
```

### async

launch와 다른 점은 Deferred<T>를 통해 결과값을 반환한다는 것입니다. 이때 지연된 결과 값을 받기 위해 await()를 사용할 수 있다.

태스크들과 같이 병행 수행되므로 어떤 루틴이 먼저 종료될 지 알기 어렵습니다. 따라서 태스크가 종료되는 시점을 기다렸다가 결과를 받을 수 있도록 `await()`를 사용해 현재 스레드의 블로킹 없이 먼저 종료되면 결과를 가져올 수 있습니다. 여기서는 `combined`라는 변수에 두 개의 비동기 루틴이 종료되고 결과가 반환되면 문자를 합쳐서 할당

```kotlin
private fun worksInParallel() {
    // Deferred<T> 를 통해 결과값을 반환
    val one = GlobalScope.async {
        doWork1()
    }
    val two = GlobalScope.async {
        doWork2()
    }

    GlobalScope.launch {
        val combined = one.await() + "_" + two.await()
        println("Kotlin Combined : $combined")
    }
}
```

### Context

어떤 루틴이 수행될 때 수행되고 있는 기반 환경의 여러가지 정보를 문맥(Context)라고 합니다. 그 문맥에 따라 실행되는 방법이 달라질 수 있다.

코루틴이 실행될 때 여러 가지 컨텍스트는 CoroutineContext에 의해 정의됩니다. launch {..}와 같이 인자가 없는 경우에는 CoroutineScope에서 상위의 문맥이 상속되어 결정.

launch(Dispatchers.Default) {...}와 같이 사용되면 GlobalScope에서 실행되는 문맥과 동일하게 사용됩니다. GlobalScope는 main의 생명 주기가 끝나면 같이 종료

```kotlin
fun main() = runBlocking(Dispatchers.Default){
    launch(Dispatchers.IO) {
        delay(1200L)
        println("Task from runBlocking") //2

    }
    coroutineScope {
        launch {
            delay(1500L)
            println("Task from nested launch")//3
        }
        delay(200L)
        println("Task from coroutineScope") //1
    }
    println("end of runBlocking") // 4
}
```

##### 종류

1. Dispatchers.Default는 기본 문맥인 CommonPool에서 실행되기 때문에 새로운 스레드를 생성하지 않고 기존에 있는 것을 이용합니다. 그러므로 연산 중심의 코드에 적합

2. Dispatchers.IO는 입출력에 적합한 공유 풀로써, 볼로킹 동작이 많은 파일 or 소켓 I/O 처리에 사용하면 좋다.

3. Unconfined 문맥은 비한정 문맥에서 실행된 코루틴은 첫번째 중단점을 만날때까지만 호출자 스레드에서 실행됩니다. 중단점 이후에 재개 되었을때는 서스펜드 함수가 실행된 스레드에서 수행됩니다. 그렇기 때문에 비한정 문맥은 예측 불능한 상태로 수행되기 때문에 해당 기능을 사용하는 것은 권장되지 않는다.

4. newSingleThreadContext는 새로운 스레드가 생성되기 때문에 비용이 많이 들고 더 이상 필요하지 않으면 해제하거나 종료시켜야 합니다. 부모 코루틴이 취소되는 경우 자식 코루틴도 재귀적으로 취소

### withContext(context)

인자로 코루틴 문맥을 지정하며 해당 문맥에 따라 코드 블록을 실행한다. 해당 코드 블록은 다른 스레드에서 수행되며 결과를 반환. 부모 스레드를 블록하지 않는다.

#### 빌더의 속성

launch빌더의 속성은 context, start, parent, onCompletion이 있으며, 그중 start속성은 다양한 옵션이 있다.

1. DEFAULT : 즉시 시작
2. LAZY : 코루틴을 느리게 시작 (블록을 중단하며 start()나 await()으로 시작)
3. ATOMIC : 원자적으로 즉시 시작 (DEFAULT와 비슷하나 코루틴 실행을 무조건 진행 실행전 취소 불가)
4. UNDISPATCHED : 현재 스레드에서 즉시 시작

### 스코프

모든 코루틴들은 각자의 스코프를 갖습니다. 그래서 runBlocking{ } 코루틴 빌더등을 이용해 생성 된 코루틴 블록 안에서 launch{ } 코루틴 빌더를 이용하여 새로운 코루틴을 생성하면 현재 위치한 부모 코루틴에 join() 을 명시적으로 호출할 필요 없이 자식 코루틴들을 실행하고 종료될 때까지 대기 할 수 있다.

특정 스코프를 정의하고 하위에 코루틴 블록을 정의하면 해당 코루틴은 그 스코프에 의해 제어되는 자식이다.

만일 어떤 코루틴들을 위한 사용자 정의 스코프가 필요한 경우가 있다면 coroutineScope{ } 빌더를 이용할 수 있습니다. 이 빌더를 통해 생성 된 코루틴은 모든 자식 코루틴들이 끝날때까지 종료되지 않는 스코프를 정의하는 코루틴이다.

아래 코드 및 주석 확인

```kotlin
fun main() = runBlocking {
    println("runBlocking : $coroutineContext")
    val request = launch {
        println("request: $coroutineContext")
        GlobalScope.launch { // 프로그램 전역으로 독립적인 수행
            println("job1: before suspend function, $coroutineContext")
            delay(1000)
            println("job1: after suspend function, $coroutineContext")
        }
        launch { //부모의 문맥을 상속(상휘 launch의 자식) //1
        //launch(Dispatchers.Default){ //부모의 문맥을 상속(상위 launch의 자식) 2
        //CoroutineScope(Dispatchers.Default).launch{ //새로운 스코프가 구성됨 request와 무관 3
            delay(100)
            println("job2: before suspend function, $coroutineContext")
            delay(1000)
            //request(부모)가 취소되어도 취소되지 않음
            println("job2: after suspend function, $coroutineContext")
        }
    }
    delay(200)
    request.cancel()// 부모의 코루틴 취소
    delay(1000)
}
```

_결과_

- launch (1) : job2 after가 실행되지 않고 코루틴 취소
- launch(Dispatchers.Default) (2) : 위와 동일
- CoroutineScope(Dispatchers.Default).launch : job2 after까지 전부 실행

### MainScope

MainScope는 UI 컴포넌트를 위한 스코프입니다. MainScope는 SupervisorJob을 생성하고 이 스코프에서 만들어진 모든 코루틴을 Main스레드에서 실행합니다. SupervisorJob을 생성하기 때문에 하나의 코루틴이 예외를 던지면서 실패해도 다른 코루틴들은 취소되지 않습니다.

자식 코루틴의 실패는 CoroutineExceptionHandler를 사용하여 처리 할 수 있습니다.

### runBlocking()

runBlocking은 새로운 코루틴을 시작하고 코루틴이 완료될 때까지 현재 스레드를 차단합니다. 일시 중단 및 논블록킹 스타일로 작성된 일반 블록 코드와 라이브러리를 연결하도록 설계되어 있다.
