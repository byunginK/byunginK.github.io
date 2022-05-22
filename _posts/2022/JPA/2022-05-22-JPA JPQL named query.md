---
layout: post
title: "JPA JPQL- Named 쿼리"
date: 2022-05-22
categories: [JPA]
---

# named query

- 미리 정의해서 이름을부여 후 사용
- 정적 쿼리만 가능
- 어노테이션, xml에 정의
- 애플리켕시녀 로딩 싲머에 초기화 후 사용
- 애플리케이션 로딩 시점에 쿼리를 검증

```java
@Entity
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username"
)
public class Member {
    ...
}
```

- 지정된 쿼리 실행

```java
List<Member> resultList =
        em.createNamedQuery("Member.findByUsername", Member.class)
            .setParameter("username","member1")
            .getResultList();
```

### 특징

- xml로 설정할 경우 xml이 우선권
- spring data jpa에서 repository의 `@Query`는 named query다.
