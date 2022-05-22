---
layout: post
title: "JPA JPQL- bulk 연산"
date: 2022-05-22
categories: [JPA]
---

# 벌크 연산

- 쿼리 한번으로 여러 테이블 로우 변경
- `executeUpdate()`를 사용

```java
em.createQuery("update Member m set m.age = 20")
                .executeUpdate();
```

- 한번에 모두 업데이트 된다.

### 주의

- 영속성 컨텍스트를 무시하고 dB에 직줩 쿼리
- 벌크 연산을 먼저 실행
- 벌크 연산 수행 후 영속성 컨텍스트 초기화 해야 나중에 벌크한 데이터를 가져올때 영속성에 있는 값을 가져오지않고 연산이 된 값들을 다시 가져온다. (만약 clear 하지 않으면 db에만 연산되었고 영속성에는 연산되지 않아 주의 필요)
