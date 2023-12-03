---
layout: post
title: "@Trancational 독립 적용"
date: 2023-08-29
categories: [Spring]
---
# Transactional REQUIRES_NEW에 대한 오해


**@Transactional**을 사용할 때 전파 속성을 "***REQUIRES_NEW***"로 지정해도, 부모 트랜잭션이랑 **독립된 트랜잭션으로 실행되지 않는다** 라는 내용이었다.

이 주장의 근거로 **"자식 트랜잭션에서 예외가 발생하면, 부모 트랜잭션에서도 롤백이 발생한다"**라는 예제를 보여주고 있었다. 다음 코드는 해당 글의 코드를 간결하게 재구성한 내용이다.

```java
@Service
@RequiredArgsConstructor
public class ChildService {
 
    private final ChildRepository childRepository;
    
    @Transactional(propagation = Propagation.REQUIRES_NEW) // 주목
    public void save() {
        childRepository.save(new Child());
        throw new IllegalStateException();
    }
 
}@Service
@RequiredArgsConstructor
public class ParentService {
 
    private final ParentRepository parentRepository;
    private final ChildService childService;
 
    @Transactional
    public void save(int age) {
        parentRepository.save(new Parent(age));
        childService.save();
    }
}
```

ParentService -> ChildService(save)를 호출하는데, **ChildService**에서 **uncatched exception을 발생시킨다**.

해당 글에선 ChildService는 ***REQUIRED_NEW*** 속성으로 트랜잭션을 설정했기 때문에, 두 서비스의 **트랜잭션은 서로 독립적이어야하는데,**

즉 Parent가 정상적으로 저장되어야 하는데, **Parent가 저장되지 않았기 때문에 두 트랜잭션이 독립적이지 않다라는 얘기였다.**

## **원인 진단**

사실 원인은 매우 간단하다. Parent가 정상적으로 저장되지 않는 이유는 바로 **예외** 때문이다.

자바에서 기본적으로 **예외가 발생했을때 처리해주지** 않으면 콜스택을 하나씩 제거하면서 최초 호출한곳까지 **예외가 전파된다.**

즉 **ParentService에서 예외처리를 하지 않았기 때문에** 예외가 전파되어서 ParentService도 **롤백을** 진행하게 된 것이다.

## **독립적 ??!**

여기서 **"독립적"**이란 단어에 대해서 얘기해볼 필요가 있을 거 같다.

***REQUIRES_NEW***은 독립적인 것인가?

보통 독립적이라하면 어떤 대상 A가 있을때, A는 다른 것으로 부터 영향을 받지 않는 것을 이야기한다.

**즉 다른 존재에 의해서 영향을 받거나 주지 않는 것을 의미한다.**

이를 비춰봤을때 ***REQUIRES_NEW***는 독립적이라는 표현보다 **별도의 트랜잭션을** 생성하는 표현이 맞다고 생각한다.

왜냐면 스프링에서 트랜잭션은 thread-local을 기반으로 동작한다. ***TransactionSynchronizationManager***는 관련 리소스들을 쓰레드 로컬에 보관하고있다. 트랜잭션 매니저는 이녀석을 이용해서 리소스를 사용한다.


REQUIRES_NEW는 별도의 새로운 트랜잭션(커넥션도 실제 다르다)을 만들 뿐, 쓰레드는 동일하다.

따라서 개인적으로 독립적이란 표현이 적합하지 않다고 생각한다. 독립적이려면 사실 위 코드에서 예외를 처리하지 않아도 Parent가 정상적으로 동작 했어야한다. Javadoc에서도 새로운 트랜잭션을 생성한다고 표현하고 있다. 독립적이란 용어는 사용하지 않는다.

REQUIRES_NEW속성을 이용하면 실제 두 트랜잭션이 서로 다른 커넥션을 이용하는 것을 확인할 수 있다.

REQUIRES_NEW에 대한 Javadoc 발췌 (독립적이란 표현을 쓰지 않는다)

## **해결 방법**

그럼 ParentService가 롤백되는 것은 어떻게 해결해야될까? 우선 해당 글에서는 **@Async**를 이용해서 해결했다고한다. 실제로 이 방법은 해결 방법은 맞지만, 문제 원인에 맞는 진단은 아니라고 생각한다.

왜냐면 @Async는 **별도의 쓰레드로 작업을 처리하는 비동기 처리를 목적으로 하는** 녀석이기 때문이다. 애초에 쓰레드 로컬이 다르기 때문에 독립적일 수 밖에 없다.

따라서 해당 해당 예제에서 문제를 해결하는 **방법은 예외 처리만 해주면 끝이다**.

```java
    @Transactional
    public void save() {
        parentRepository.save(new Parent());
        try {
            childService.save();
        } catch (IllegalStateException e) {
            log.info(">>> child service 예외 발생");          
        }
    }
```

이런식으로 예외를 처리하면 ParentService는 정상적으로 커밋이 되고, Parent가 정상적으로 저장되는 것을 확인할 수 있다.

그럼 이런 의문이 들 수 있다. 그럼 ***REQUIRES*** 쓰지않고 ***REQUIRES_NEW*** 를 쓰면 뭐가 다른 것이지?

위 코드랑 똑같이 예외 처리를 한 상태에서 ChildService에서 ***REQUIRES_NEW***를 빼버리고 ***REQUIRES***로 바꾸면 어떻게 될까?

**ParentService는 롤백된다**. 이유는 하나의 물리적 트랜잭션에서 여러개의 논리적인 트랜잭션으로 구성되어 있을 경우, 하나의 트랜잭션에서 uncatched exception이 발생하면, 롤백 마크를하고 해당 물리적 트랜잭션은 전체 롤백이 되기 때문이다.