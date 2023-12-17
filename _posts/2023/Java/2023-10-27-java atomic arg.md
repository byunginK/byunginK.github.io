---
layout: post
title: "Java Atomic 변수"
date: 2023-10-28
categories: [spring]
---
# [Java] Atomic 변수

## **Atomic 변수란**

> Atomic 변수는 원자성을 보장하는 변수이다.
> 

멀티스레드 환경에서 동기화 문제를 해결하기 위해 synchronized 키워드를 사용하여 락을 걸곤 한다. 그런데 synchronized는 특정 스레드가 해당 블럭 전체를 lock 하기 때문에 낭비가 심하다. 이러한 문제를 해결하기 위해 고안된 방법이 Atomic 변수이다.

Atomic 변수는 **CAS(Compare And Swap) 알고리즘**을 사용하여 NonBlocking하면서 동기화 문제를 해결할 수 있다.

## **CAS(Compare And Swap) 알고리즘**

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbOvZrW%2FbtrFBhAUHyi%2FNXgRskbTDzczcPhaNhJ1P0%2Fimg.png)

1. CAS 알고리즘

![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FK4Izd%2FbtrFzNt6Ipg%2FtRPKoicflEUUImMMZPw1u1%2Fimg.png)

2. CPU 캐시 메모리

멀티스레드 환경, 멀티코어 환경에서 각 CPU는 메인 메모리에서 변수값을 참조하는 것이 아니라, 각 CPU의 개시 영역에서 메모리를 참조하게 된다. (그림 2 참조)

이 때, 메인 메모리에 저장된 값과 CPU 캐시에 저장된 값이 다른 경우가 있는데 (가시성 문제) 이럴 때 사용되는 것이 CAS 알고리즘이다. **현재 스레드에 저장된 값**과 **메인 메모리에 저장된 값**을 비교하여 **일치하는 경우 새로운 값으로 교체**되고, **일치하지 않는다면 실패하고 재시도**를 한다.

이런 방법으로 처리하면 CPU 캐시에 잘못된 값을 참조하는 가시성 문제를 해결할 수 있다.

참고로 synchronized 의 경우 synchronized 진입 전 메인 메모리와 CPU 캐시 메모리 값을 동기화하여 문제가 없도록 처리한다.

## **Atomic 변수 예**

```java
public class AtomicInteger extends Number implements java.io.Serializable {

	private volatile int value;

        public final int incrementAndGet() {
	    int current;
            int next;
            do {
	        current = get();
                next = current + 1;
            } while (!compareAndSet(current, next));
            return next;
        }

        public final boolean compareAndSet(int expect, int update) {
        	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
        }

}
```

AtomicInteger 클래스의 내부이다.

incrementAndGet() 메소드 내부에서 CAS 알고리즘 로직을 구현하고 있다.

compareAndSet()을 호출하고, 그 결과값이 성공일 때까지 반복하게 된다.

compareAndSet() 내부에서는 compareAndSwapInt()를 호출하여 메모리에 저장되어진 값과 현재 CPU에 캐시된 expect 값을 비교하여 동일한 경우만 변경하는 기능을 수행한다.

여기서 봐야할 점은 volatile int value; 이다.

volatile 변수는 가시성 문제를 해결하기 위하여 사용된다.

**volatile 키워드는 CPU 캐시가 아닌 메모리에서 값을 참조하게 한다.**

그렇다면, volatile 키워드를 사용하여 메모리에서 직접 값을 참조하는데 CAS 알고리즘 적용이 필요한가? 답은 "필요하다" 이다.

volatile 키워드는 오직 한 개의 스레드에서 작업을 할 때, 그리고 다른 스레드는 읽기 작업만을 할 때 안정성을 보장한다.

하지만 AtomicInteger는 여러 스레드에서 읽기/쓰기 작업을 병행한다.

그래서 CAS 알고리즘을 사용하여 2중 안전을 기하는 방법을 사용한다.