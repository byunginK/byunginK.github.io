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
