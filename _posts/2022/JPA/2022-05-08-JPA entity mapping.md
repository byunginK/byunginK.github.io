---
layout: post
title: "JPA 엔티티 매핑"
date: 2022-05-08
categories: [JPA]
---

# 엔티티 매핑

### @Column

- insertable, updatable 등록, 변경 가능 여부 (default = true)
- nullable null허용여부
- columnDefinition 컬럼 정보를 직접 줄 수있다.

### @Enumerated

- enum 타입을 매핑 할때 사용
- 순서로 insert하는 ORDINAL은 사용 X, 자바 로직에 순서가 바뀌면 전부 꼬인다.

### @Temporal

- 날짜 타입 매핑 (DATE, TIME, TIMESTAMP)
- 최신 자바에서는 LocalDate를 사용하면 위의 설정은 굳이 안해도 상관없다.

```java
private LocalDate localDate; //DATE 로 매핑
private LocalDateTime localDateTime; // Timestamp로 매핑
```

### @Lob

- 타입에 따라 LOB 타입 컬럼 생성(String 일 경우 CLOB, byte 일 경우 BLOB)

## 기본키 매핑

### @Id

- 직접 할당

### @GeneratedValue

** ID는 Long을 사용하는것을 권장 **

- IDENTITY : 각 DB에 맞게 알아서 위임 (오라클에서는 GenerationType.SEQUENCE로 Object를 생성 후 사용)
  > SequenceGenerator 를 사용하여 시퀀스에 대한 자세한 옵션 설정 가능. nextcall로 인해 성능 이슈는 allocationSize를 활용하여 미리 가져와 메모리에서 쓰는방식
- 제약 : entityManager.persist를 호출하는 시점에 바로 DB insert쿼리를 날림 (원래 commit에 날림) JPA가 영속을 하기 위해 persist시점에 insert하고 바로 select해서 id를 가져온다.
