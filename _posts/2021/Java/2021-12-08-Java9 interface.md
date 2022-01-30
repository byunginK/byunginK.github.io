---
layout: post
title:  "Java9 interface"
date:   2021-12-08
categories: [web]
---
# Java8 ~ 9 Interface
자바8 이전에는 인터페이스의 모든 메서드가 추상 메서드여야 했다. 즉 메서드의 바디가 없어야했다. 하지만 자바8 이후 정적메서드와 기본 메서드를 인터페이스에 넣을 수 있고, 자바9 부터는 비공개 메서드를 사용할 수 있다.

### 1.정적 메소드
static 메소드는 기존의 `companinon class(유틸리티 클래스)`에 정적 메소드을 사용하지않고 인터페이스를 사용하게 된다.
아래 인터페이스와 static 메소드의 예시를 보면 메소드를 바로 사용할 수 있는걸 알 수 있다.

```java
package com.acompany.inheritance;

//함수형 인터페이스를 명시 (추상메소드 1개만 존재)
@FunctionalInterface
public interface Payable {

    long paySalary();

    //default 메소드를 선언하여 인터페이스에 구현을 할 수 있다.
    default long payAllowance() {
        callLocal();
        return 0;
    }

    //java 9
    private void callLocal() {

    }

    static long testStatic() {
        return 1;
    }

}
```
위 `interface`는 어노테이션으로 `@FunctionalInterface`이 선언되어있다. 해당 어노테이션은 함수형 인터페이스를 명시한다. 
> 함수형 인터페이스 : 추상메소드가 1개만 존재하며 이후 포스트에 있는 람다표현식과 관련이 있다.

`testStatic()`정적 메소드는 아래와 같이 바로 사용할 수 있다.
```java
package com.acompany.inheritance;

public class InheritanceTest {
    public static void main(String[] args) {
        //인터페이스의 static 메소드에 바로 접근하여 사용 가능하다.
        Payable.testStatic();
    }
}
```
### 2. 기본 메소드 (default)
인터페이스에서 기본 메소드는 반드시 default 제어자가 붙어야한다. 위에서 구현한 `Payable`은 추상메소드 `paySalary()`와 기본 메소드 `payAllowance()`가 있다. 다른 클래스에서 해당 클래스를 상속할 때 기본 메소드는 오버라이드하여 바로 사용이 가능하다.
또는 **인터페이스를 상속 했지만 기본 메소드는 반드시 오버라이드 하지 않아도 클래스가 컴파일된다.**
```java
package com.acompany.inheritance;

//만약 abstract 없으면 Payable에 정의된 추상 메소드를 무조건 구현 해야한다.
public abstract class Employee implements Payable {

    protected String name;
    protected long salary;

    public Employee(String name, long salary) {
        this.name = name;
        this.salary = salary;
    }

    //기본 메소드는 반드시 선언하지 않아도 컴파일이 가능하다.
    @Override
    public long payAllowance() {
        return Payable.super.payAllowance();
    }

    public String getName() {
        return name;
    }

    public long getSalary() {
        return salary;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", salary=" + salary +
                '}';
    }
}
```
여기서 한가지 살펴볼 점은 `class`에 `abstract`을 추가해준 것이다. 기본메소드는 선언하지 않아도 컴파일이 되지만 `Payable`인터페이스에는 하나의 추상메소드가 존재한다. 따라서 상속받는 클래스는 오버라이드를 해주어야 하지만 `abstract`을 추가하면 오버라이드 하지않고도 컴파일이 가능해 진다.

### 3. 비공개 메소드 (Java9)
비공개 메소드는 인터페이스 자체에 있는 메소드에서만 쓸 수 있으므로, 인터페이스 안에 있는 다른 메소드의 헬퍼 메소드로만 사용할 수 있다.
`Payable`에서 `callLocal()` 비공개 메소드는 `payAllowance()` 메소드에서만 사용되고 있다.