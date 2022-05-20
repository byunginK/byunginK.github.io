---
layout: post
title: "JPA 영속성 전이(Cascade)"
date: 2022-05-15
categories: [JPA]
---

# 영속성 전이

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들때 사용

### 부모 클래스와 자식 클래스

- 부모

```java
@Entity
public class Parent {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child){
        childList.add(child);
        child.setParent(this);
    }
}
```

- 자식

```java
@Entity
public class Child {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;
}
```

- 영속을 하기위해 다음과 같이 3번을 다해줘야한다. (CASCADE를 안쓸경우)

```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();

parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
em.persist(child1);
em.persist(child2);
```

- cascade를 적용 `@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)`

```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();

parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);

```

- parent만 영속을 해도 child가 전부 영속된다.

### 주의

- 영속성 전이는 연관관계 맵핑과 관련 없다.
- 보통 ALL, PERSIST만 사용한다.

### 사용하는 경우

- 단일 소유자: 하나의 부모 클래스 (하나의 엔티티)만 여러개의 자식 클래스를 관리하면 사용, 다른곳에서도 자식 클래스를 관리도 한다면 사용 X
- 단일 엔티티에 완전히 종속적일때 (같은 라이프 싸이클일 경우)만 사용

### 고아객체

- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동 삭제 `orphanRemoval = true`를 사용
- 만약 컬렉션에서 remove 됐을경우 delete 쿼리가 나간다.

```java
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> childList = new ArrayList<>();
```

#### 주의

- 참조하는 곳이 하나일 때만 사용
- 특정 엔티티가 개인 소유할때
- `@OneToOne`,`@OneToMany`만 사용가능
- `cascade = CascadeType.ALL, orphanRemoval = true`이렇게 두개를 모두 키면 부모 엔티티로 자식 엔티티를 컨트롤 할 수 있다. (도메인 주도 설계의 Aggregate Root 개념 구현시 유용)
