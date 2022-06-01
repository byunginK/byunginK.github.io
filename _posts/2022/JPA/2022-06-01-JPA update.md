---
layout: post
title: "JPA 변경감지와 병합"
date: 2022-06-01
categories: [JPA]
---

# 변경 감지(dirty checking)와 병합

- JPA가 식별할 수 있는 ID값을 가지고 있지만 새로운 객체로 생성되었을 경우에는 준영속 엔티티이다.
- (준영속 엔티티 : jpa가 관리를 하지 않는 엔티티)

### 변경감지

- 영속성 컨텍스트에서 엔티티를 조회한 후에 데이터를 수정
- commit하는 순간 update 쿼리가 나가게된다.

```java
@Transactional
public void update(Item itemParam){
    Item findItem = em.find(Item.class, itemParam.getId());
    findItem.setPrice(itemParam.getPrice()); //여기서 데이터를 수정
}
```

### 병합 (merge)

- 준영속상태의 엔티티를 영속 엔티티로 변경해준다.
- _주의_ : 엔티티의 변경하지 않으려고 한 값들도 다 같이 변경이 되어 버린다.(만약 값 설정을 하지 않으면 null로 들어간다.)

### 병합 보다는 변경감지를 사용하는 것이 좋다.
