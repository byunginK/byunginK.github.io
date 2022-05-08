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
