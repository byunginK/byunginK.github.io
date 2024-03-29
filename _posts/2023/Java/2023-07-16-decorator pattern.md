---
layout: post
title: "Decorator pattern에 대해"
date: 2023-07-16
categories: [Java]
---
# [Design Pattern] 데코레이터 패턴(Decorator pattern)에 대하여

데코레이터 패턴(Decorator Pattenr)은 주어진 상황 및 용도에 따라 **어떤 객체에 책임(기능)을 동적으로 추가하는 패턴**을 말합니다. 데코레이터라는 말 그대로 장식이라고 생각하시면 편합니다. 기본 기능을 가지고 있는 클래스를 하나 만들어주고 추가할 수 있는 기능들을 추가하기 편하도록 설계하는 방식입니다.

![데코레이터 패턴](https://blog.kakaocdn.net/dn/bnP6V5/btq1JVdeaQb/0DLKqgOGPfhb2qhfZmKWRk/img.png)

**Component :** 실질적인 인스턴스를 컨트롤하는 역할

**ConcreteComponent :** Component의 실질적인 인스턴스의 부분으로 책임의 주체의 역할

**Decorator :** Component와 ConcreteDecorator를 동일시 하도록 해주는 역할

**ConcreteDecoreator :** 실질적인 장식 인스턴스 및 정의이며 추가된 책임의 주체

## 데코레이터 패턴의 장단점

### 장점

**1.** 기존 코드를 수정하지 않고도 데코레이터 패턴을 통해 행동을 확장시킬 수 있습니다.

**2.** 구성과 위임을 통해서 실행중에 새로운 행동을 추가할 수 있습니다.

### 단점

**1.** 의미없는 객체들이 너무 많이 추가될 수 있습니다.

**2.** 데코레이터를 너무 많이 사용하면 코드가 필요 이상으로 복잡해질 수 있습니다.

### ※ 데코레이터 패턴을 아래와 같은 상황일 때 사용해주시면 좋습니다.

**1.** 클래스의 요소들을 계속해서 수정을 하면서 사용하는 구조가 필요한 경우

**2.** 여러 요소들을 조합해서 사용하는 클래스 구조인 경우

위와 같은 상황일때가 언제일까요? 예를들어 커피를 제조하는 방법에 대해서 말해보도록 하겠습니다. 커피는 종류마다 아메리카노는 에스프레소에 물을 섞고 카페라떼는 에스프레소에 스팀우유나 휘핑크림을 얹기도 하는 등 커피를 만들때 다양한 재료들의 구성으로 하나의 커피가 완성됩니다.

이 재료들을 모두 클래스로 구현해 준다면 굉장히 많은 클래스들을 구현해줘야 할 것입니다. 또 이 구조는 상당히 비효율적입니다. 그 이유는 새로운 커피를 하나 개발할때마다 그 커피에 들어가는 재료의 객체를 만들고 기능을 추가해주어야 하기 때문입니다. 커피의 종류가 많으면 많아질수록 코드가 비효율적이겠죠. 또한 커피를 만들때마다 매번 해당하는 클래스들의 객체를 생성해주어야 하는 문제도 있습니다. 이를 해결하기 위해 기본 커피의 재료인 에스프레소라는 틀을 추상적으로 가지고 커피를 만들때 들어가는 재료들을 장식하는 방식을 사용하는것입니다.

## 데코레이터 패턴 사용 예제

### Component.interface

```java
public interface Component {
    String add(); //재료 추가
}

```

커피를 제조할때는 여러가지 재료를 추가해야 합니다. Component 인터페이스에서는 재료들을 추가해주는 함수를 구현합니다.

### BaseComponent.java

```java
public class BaseComponent implements Component {

    @Override
    public String add() {
        // TODO Auto-generated method stub
        return "에스프레소";
    }
}

```

BaseComponent에서는 Component를 상속받아 커피의 기본 재료가 되는 에스프레소를 넣는것으로 정의해줍니다.

### Decorator.java

```java
abstract public class Decorator implements Component {
    private Component coffeeComponent;

    public Decorator(Component coffeeComponent) {
        this.coffeeComponent = coffeeComponent;
    }

    public String add() {
        return coffeeComponent.add();
    }
}

```

Decorator는 커피의 재료들의 근간이 되는 추상클래스입니다. 재료들은 이 Decorator를 상속받아 재료를 추가합니다.

### WaterDecorator.java

```java
//물을 추가해주는 클래스
public class WaterDecorator extends Decorator {
    public WaterDecorator(Component coffeeComponent) {
        super(coffeeComponent);
    }

    @Override
    public String add() {
        // TODO Auto-generated method stub
        return super.add() + " + 물";
    }
}
```

WaterDecorator는 물을 추가하는것으로 add메소드를 정의해줍니다.

### MilkDecorator.java

```java
//우유를 추가해주는 클래스
public class MilkDecorator extends Decorator {
    public MilkDecorator(Component coffeeComponent) {
        super(coffeeComponent);
    }

    @Override
    public String add() {
        // TODO Auto-generated method stub
        return super.add() + " + 우유";
    }
}

```

MilkDecorator는 우유를 추가하는것으로 add메소드를 정의해줍니다.

### Main.java

```java
public class Main {

    public static void main(String[] args) {
        Component espresso = new BaseComponent();
        System.out.println("에스프레소 : " + espresso.add());

        Component americano = new WaterDecorator(new BaseComponent());
        System.out.println("아메리카노 : " + americano.add());

        Component latte = new MilkDecorator(new WaterDecorator(new BaseComponent()));
        System.out.println("라떼 : " + latte.add());
    }
}

```

데코레이터 패턴을 사용하면 위와 같이 자기가 감싸고 있는 구성요소의 메소드를 호출한 결과에 새로운 기능을 더함으로써 행동을 확장할 수 있습니다.