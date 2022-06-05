---
layout: post
title: "JPA Entity와 DTO"
date: 2022-06-05
categories: [JPA]
---

# 응답 값으로 엔티티를 직접 외부 노출

```java
@GetMapping("/api/v1/members")
public List<Member> membersV1(){
    return memberService.findMembers();
}
```

### 문제점

- 엔티티에 프레젠테이션 계층을 이ㅜ한 로직이 추가된다.
- 기본적으로 엔티티의 모든 값이 노출됨
- 응답 스펙을 맞추기 위해 로직이 추가된다. (@JsonIgnore, 별도의 뷰 로직 등등)
- 실무에서는 같은 엔티티에 대해 API가 용도에 따라 다양하게 만들어 지는데, 한 엔티티에 각각의 API를 위한 프레젠테이션 응답 로직을 담기는 어렵다.
- 엔티티가 변경되면 API 스펙이 변한다. (client에게 사이드 이펙트 발생)
- 추가로 컬렉션을 직접 반환하면 향후 API 스펙을 변경하기 어렵다.(별도의 Result 클래스 생성으로 해결)

### 해결법

- DTO를 생성하여 응답 반환 한다.

```java
@GetMapping("/api/v2/members")
public Result memberV2() {
    List<Member> findMembers = memberService.findMembers();
    List<MemberDto> collect = findMembers.stream().map(m -> new MemberDto(m.getName()))
            .collect(Collectors.toList());
    return new Result(collect);
}


@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}

@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```
