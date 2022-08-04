---
layout: post
title: "JPA Spring Data Qeury Method"
date: 2022-08-01
categories: [JPA]
---

# 쿼리 메소드 기능

## 메소드 이름으로 쿼리 생성

Spring Data JPA 사용시 Repository에 메소드 이름으로 쿼리를 자동으로 생성하도록 할 수 있다.

```java
public interface UserRepository extends JpaRepository<User, Integer> {

	public User findByUsername(String username);

	Optional<User> findByEmail(String email);

}
```

위의 코드에서 `findByUsername`는 입력된 username의 값으로 DB를 조회하고 `findByEmail`는 입력된 email의 값으로 쿼리를 생성하여 조회, 값을 리턴한다.

## NamedQuery

Entity에 `@NamedQuery`설정을 하고 Repository에서 호출한다.

- 실무 사용 거의 없음

## @Query 레포지토리 메소드에 정의

```java
@Query("select m from Member m where m.username = :username and m.age = :age")
List<Member> findUser(@Param("username") String username, @Param("age") int age);
```

레포지토리에 `@Query`에 쿼리문을 작성하여 실행 할 수 있다. (정적 쿼리)

## @Query, 값, DTO 조회

### 단순 타입 값 가져오기

```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```

컬럼의 값이 문자열일 경우 받는 타입을 String으로 할 경우 받아 올 수 있다.

### DTO 반환

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
List<MemberDto> findMEmberDto();
```

dto의 경로를 모드 select 절에 넣어주고 생성자로 생성하듯이 호출할 값들을 감싸준다. (MemberDto에는 해당 값들로 객체를 생성할 수 있도록 생성자를 만들어 준다)

## 반환 타입

```java
List<Member> findByUsername(String username); //컬렉션
Member findMemberByUsername(String username); // 단건
Optional<Member> findOptionalByUsername(String username); //단건
```

- 반환 타입은 유연하게 리턴이 가능하다.
- 만약 쿼리를 통해 나오는 결과가 없다면(컬렉션), 빈 컬렉션을 넘겨준다 (null 일 경우가 없다.)
- 만약 단건(객체)일 경우에는 null로 반환한다.
- java 8 이후로 부터는 Optional을 사용하여 null 반환에 대해 방지 할 수 있다.
- 단, 단건 조회 쿼리에서 결과가 다수가 나왔을 경우 예외가 발생
