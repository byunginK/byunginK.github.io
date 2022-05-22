---
layout: post
title: "JPA JPQL- fetch join"
date: 2022-05-22
categories: [JPA]
---

# Fetch join

- JPQL에서 성능 최적화를 위해 제공
- 연관된 엔티티나 컬렉션을 SQL한번에 함께 조회하는 기능

#### 기본적인 지연로딩 & 팀을 가져올때 실행 순서

- member 1 = teamA
- member 2 = teamA
- member 3 = teamB

```java
String query = "select m from Member m";

List<Member> resultList =
        em.createQuery(query, Member.class)
        .getResultList();

for ( Member m : resultList ){
    System.out.println("m = " + m.getUsername() + ", " + m.getTeam().getName());
}
```

- 우선 멤버를 가져온다

```SQL
select
    member0_.id as id1_0_,
    member0_.age as age2_0_,
    member0_.TEAM_ID as TEAM_ID5_0_,
    member0_.type as type3_0_,
    member0_.username as username4_0_
from
    Member member0_
```

- 지연로딩으로 팀이름을 호출해서 그때서야 팀데이터를 가져옴

```sql
select
    team0_.id as id1_3_0_,
    team0_.name as name2_3_0_
from
    Team team0_
where
    team0_.id=?


m = member1, teamA
m = member2, teamA
```

- member2는 같은 teamA이기 때문에 영속성(1차 캐시)에서 값을 가져오고

```sql
select
    team0_.id as id1_3_0_,
    team0_.name as name2_3_0_
from
    Team team0_
where
    team0_.id=?
m = member3, teamA
```

- member3은 다른 팀의 데이터이기 때문에 또 쿼리를 통해 데이터를 가져온다.

##### 최악의 경우 N번 모두 쿼리를 날림

### fetch join사용

- `String query = "select m from Member m join fetch m.team";`
- 한번에 join을 통해 가지고옴

```sql
select
    member0_.id as id1_0_0_,
    team1_.id as id1_3_1_,
    member0_.age as age2_0_0_,
    member0_.TEAM_ID as TEAM_ID5_0_0_,
    member0_.type as type3_0_0_,
    member0_.username as username4_0_0_,
    team1_.name as name2_3_1_
from
    Member member0_
inner join
    Team team1_
        on member0_.TEAM_ID=team1_.id

m = member1, teamA
m = member2, teamA
m = member3, teamB
```

## 컬렉션 fetch join

- 일대다 조인 (데이터가 뻥튀기 된다)
- 같은 값이여도 중복을 데이터가 나온다. (`DISTINCT`를 사용 ,엔티티 중복 제거)
- 단, sql입장에서 distinct는 안먹는다 jpa가 같은 값의 엔티티는 제거한다. (내부적으로는 사실 다른 데이터 이므로)
- 최적화가 필요한 곳은 fetch join 적용

### 일반조인과 fetch join 차이

- 일반 조인은 쿼리에 join은 있지만 select 절에 가져오려는 데이터가 없기 때문에 지연로딩과 같이 여러번 쿼리가 나간다.
- 일반 조인은 연관된 엔티티를 조회 안함

### 일대다 fetch join 한계

- fetch join 대상에는 별칭을 줄 수 없음
- 둘 이상의 컬렉션은 fetch join 할수 없다.
- 컬렉션을 fetch join하면 페이징을 사용할 수 없다.(일대다)
  > 일대일, 다대일은 fetch join해도 페이징 가능

#### 단 일대다의 엔티티에 `@BatchSize`를 돌리면 한번에 불러와지긴한다. (in 쿼리에 값들을 넣어 쿼리 실행)

- persistence.xml에 글로벌 설정으로 할 수 있다.
