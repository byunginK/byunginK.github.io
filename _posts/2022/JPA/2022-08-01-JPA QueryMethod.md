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

## 페이징

1. 기존 JPQL에서 사용하던 방식

```java
public List<Member> findByPage(int age, int offset, int limit){
	return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
			.setParameter("age",age)
			.setMaxResults(limit)
			.setFirstResult(offset)
			.getResultList();
}
```

2. spring data jpa의 페이징 및 정렬

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

Page<Member> findByAge(int age, Pageable pageable);
```

spring data에서 지원하는 기능을 사용

```java
int age = 10;
//0페이지부터, 3개를 username으로 정렬해서 PageRequest로 반환
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

//자동으로 totalcount까지 진행 후 페이징
Page<Member> page = memberRepository.findByAge(age, pageRequest );

List<Member> content = page.getContent(); //페이징 후 결과 값 list로 반환
long totalElements = page.getTotalElements(); //총 갯수
```

`Slice<Member>`는 아래와 같이 interface로 설정 후 실행한다.

```java
Slice<Member> findByAge(int age, Pageable pageable);
```

전체 카운트 쿼리가 나가지 않는다. (보통 모바일 "더보기"기능으로 할때 유용)

- 쿼리가 복잡해지면 (left join 등) 카운트 쿼리가 성능적으로 영향을 미칠 때 별도의 쿼리로 실행이 되게끔 할 수 있다.

```java
@Query(value = "select m from Member m",
		countQuery = "select count (m) from Member m")
Page<Member> findByAge(int age, Pageable pageable);
```

- 외부로 결과값을 리턴할 때 DTO로 변환 할때는 `page.map`을 활용하여 반환

```java
Page<MemberDto> memberDtos = page.map(member -> new MemberDto(member.getId(), member.getUsername(), null));

List<MemberDto> content = memberDtos.getContent();
```

## Bluk Update

한번에 많은 데이터를 수정하기 위해 `@Modifying`어노테이션을 사용하여 업데이트
(update 쿼리는 `@Modifying`어노테이션 붙여줘야한다.)

```java
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

**_(주의)업데이트를 벌크로 했을경우 jpa 영속성에는 아직 반영이 안되고 DB에만 값이 수정 된다._**

따라서 벌크 연산을 진행하고나서는 모든 영속성의 값을 clear 해줘야한다.

```java
@PersistenceContext
private EntityManager em;

em.flush();
em.clear();
```

위와 같이 날려주고 다시 조회를 해야한다.
(JPQL을 사용하게되면 쿼리를 날리고 flush를 안하고 clear만 하면된다.)

- spring data jpa에서는 아래와 같이 어노테이션을 지원하여 별도의 `EntityManager`로 clear해줄 필요가 없다.

```java
@Modifying(clearAutomatically = true)
```

## EntityGraph

join의 Lazy 호출의 문제인 N + 1를 발생할때 fetch join을 사용.
fetch join을 spring data에서는 아래와 같이 사용

1. 아래 코드는 기존 JPQL의 fetch join 방식

```java
@Query("select m from Member m left join fetch m.team")
List<Member> findMemberFetchJoin();
```

2. spring data에서 제공하는 fetch join 어노테이션

```java
//@Query("select m from Member m left join fetch m.team")
@EntityGraph(attributePaths = {"team"})
List<Member> findMemberFetchJoin();
```

## JPA Hint & Lock

1. JPA Hint
   SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트

```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly",value = "true"))
Member findReadOnlyByUsername(String username);
```

위와 같이하면 성능최적화를 통해 스냅샷을 생성하지않고 변경감지 체크를 하지 않는다.

2. Lock

DB 사용시 다른 어플리케이션에서 접근에 대해 Lock을 걸 수 있다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findLockByUsername(String username);
```
