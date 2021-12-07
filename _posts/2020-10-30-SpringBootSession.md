---
layout: post
title:  "Web Spring boot Security세션 값 받아오기"
date:   2020-10-30
categories: [web]
---

# Security Session 값 불러오기
###### 스프링 시큐리티 활성화시 로그인을 하면 UserDetails를 상속받은 Dto가 자동으로 세션에 담기게 된다.
```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
```
위의 함수를 이용하여 세션에 저장된 값에 접근 할 수 있다. 이후 받고 싶은 정보는 아래 코드를 이용한다.

-   Get the username of the logged in user: `getPrincipal()`
-   Get the password of the authenticated user: `getCredentials()`
-   Get the assigned roles of the authenticated user: `getAuthorities()`
-   Get further details of the authenticated user: `getDetails()`

###### 만약 기본 생성 값 이외의 추가적인 내용이 있다면 캐스트변환을 통해 불러올 수 있다. (아래 CustomSecurityDetails 클래스는 커스튬 생성한 객체이다)
```java
CustomSecurityDetails user = (CustomSecurityDetails)SecurityContextHolder.getContext().getAuthentication().getPrincipal();
```

### View 에서 값 불러오기
##### thymeleaf 사용시 input value에 spring security authentication 값을 넣을떄
```html
th:value="${#authentication.principal.something}"
```
