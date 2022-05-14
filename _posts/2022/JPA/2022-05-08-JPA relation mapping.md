---
layout: post
title: "JPA 관계 매핑"
date: 2022-05-08
categories: [JPA]
---

# 연관관계 맵핑

- 연관관계의 주인 :객체 양방향 연관관계는 관리 필요
- 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.

#### 테이블은 외래 키로 조인을하고 객체는 참조를 통해서 연관 관계를 가짐

## 단방향

### @ManyToOne

- N:1방향으로 설정을 하며, **@JoinColumn** 어노테이션을 통해 객체로 연관관계 매핑 가능
- `JoinColumn`을 넣어줘야한다. 그렇지 않으면 `JoinTable`이 디폴트로 설정되어 운영이 어려워 질 수 있다.
- 연관관계의 주인이 되어야 한다.

### @OneToMany

- 1:N으로 스펙상 존재는 하지만 실제로 거의 쓰이지 않음
- 연관관계에서 테이블상 외래키를 가지지 않은 객체가 외래키의 주인이 될경우 update구문으로 외래키 테이블의 쿼리가 처리 (성능상 단점)
- 다대일 `@ManyToOne`을 사용 권장

### @OneToOne

- 대칭적으로 서로 외래 키를 설정 할 수 있음
- 외래키를 유니크키로 지정

## 양방향

### 매핑

- Member 객체에서는 Team 객체를 ManyToOne으로 참조를하고 Team에서는 List<Member> 로 OneToMany로 매핑
- mappedBy로 연결된 인수 설정
- 둘 중 하나로 외래 키를 관리해야 한다.

##### Member

```java
@Entity
public class Member {

    public Member() {
    }

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

##### Team

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

}
```

### 연관관계의 주인

- 객체의 두 관계중 하나를 주인으로 지정
- 주인만이 외래 키를 관리
- 주인이 아닌쪽은 읽기만 가능
- 주인은 mappedBy 속성 사용 X
- 주인이 아니면 mappedBy 속성으로 지정

**외래 키가 있는 곳을 주인으로 지정**

### 주의점

- 주인에 값을 변경 및 추가를 해야함
- 양방향으로 값을 입력 할때는 객체지향적으로 봤을때 둘다 값을 셋팅해야한다.
- 또는 아래와 같이 편의 메서드를 객체에 선언하여 사용

```java
//Member 객체
    public void changeTeam( Team team ) {
        this.team = team;
        team.getMembers().add(this); //this는 Member
    }
```

- 무한 루프 조심 (toString(), Json라이브러리, lombok 등)
  > toString을 쓰지않도록 하며, json라이브러리는 dto를 별도 생성해서 반환 및 가공(Entity를 건드리지 않는다)

### 다대다 @ManyToMany

- 관계형 DB는 정규화된 테이블 2개로 표현 불가능
- 일대다, 다대일로 풀어야한다.
- 중간 테이블이 생성 (매핑) 단, 매핑 테이블에 다른 정보 추가 불가

### 해결법

- @OneToMany 와 @ManyToOne으로 풀어야한다.
- 중간 테이블을 엔티티로 승격한다.

### 상속관계 매핑

- DB 슈퍼타입 서브타입 관계의 모델링 기법이 객체의 상속과 유사

#### 1. DB입장에서 상속의 관계를 3가지 방법으로 구성가능

- 조인전략 (공통 테이블에 상속을 받을 테이블의 구분 컬럼을 추가)(가장 정규화된 방법)
- 각각 테이블 전략
- 단일 테이블 전략 (컬럼으로 전부 넣어놓고 타입 컬럼으로 구분)(성능상 선택)(상속시 JPA 기본 적략)

#### 2. 주요 어노테이션

- `@Inheritance(strategy = InheritanceType.JOINED)`으로 설정 시 조인전략으로 테이블 구성

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Item {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;

}

@Entity
public class Movie extends Item{

    private String director;
    private String actor;
}

@Entity
public class Album  extends Item{

    private String artist;
}

...
```

```sql
    create table Item (
       id bigint not null,
        name varchar(255),
        price integer not null,
        primary key (id)
    )

        create table Album (
       artist varchar(255),
        id bigint not null,
        primary key (id)
    )

        create table Movie (
       actor varchar(255),
        director varchar(255),
        id bigint not null,
        primary key (id)
    )
```

- `@DiscriminatorColumn` default가 조인이된 엔티티 명이 들어가게 된다. (DTYPE 이 생성되게됨) `@DiscriminatorValue("A")`를 자식 엔티티에 붙이면 설정된 값("A")이 Item 테이블에 DTYPE으로 들어간다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public abstract class Item {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;

}

@Entity
@DiscriminatorValue("A")
public class Album  extends Item{

    private String artist;
}
```

- `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`으로 했을때 문제는 부모 클래스로 자식 클래스를 조회 했을때 union all로 전부 검색

### @MappedSuperclass

- 공통 매핑 정보가 필요할때 사용 (id, name, 등 )
- 아래 예제는 날짜등의 공통 속성을 상속 받도록 진행
- 직접 사용하지 않기 때문에 추상 클래스로 사용 권장

```java
@MappedSuperclass
public abstract class BaseEntity {

    private String createBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}


@Entity
public class Member extends BaseEntity{

    public Member() {
    }

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;
...
```
