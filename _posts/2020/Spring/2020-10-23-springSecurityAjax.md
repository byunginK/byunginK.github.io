---
layout: post
title:  "Web Spring boot Security Ajax 에러"
date:   2020-10-23
categories: [web]
---

# 로그인 되지 않은 상태에서 POST 전송 시 403 ERROR

### csrf
##### 스프링 시큐리티를 적용하면, 별다른 설정을 하지 않더라도 기본적으로 csrf라는 기능이 활성화되어 있다.
###### CSRF : Cross-site Request fogery 의 의미로 A서비스에 로그인하면 브라우저에 A 서비스의 로그인 관련 쿠키정보가 남게 된다. 이후 동일 브라우저에서 악의적인 코드가 있는 B서비스에 접속하게 되면 B는 A의 서비스로 임의의 권한이 필요한 Request를 전송할 수 있다.

###### 이 문제는 과거 유명 경매 사이트인 옥션에서 발생한 개인정보 유출사건에 사용된 방식이다. 그렇기에 개발자 분들은 이문제를 CSRF 토큰을 통해서 권한문제를 해결해야 한다.

###### 보통 http.csrf().disable() 처럼 csrf에 대한 disable에 대해서 언급하는데 이럴 경우에 보안문제가 발생할 수 있기에 추천하지 않는다.

#### ajax를 이용하는 경우
###### ajax를 통해 post요청을 보낼 때에도, csrf 토큰을 같이 보내줘야한다. 이때는 직접 csrf도 함께 보내줘야한다. header에 포함시켜서 보내줘야하기 때문에 아래와 같이 자바스크립트를 사용한다.

```javascript
<script type="application/javascript" th:inline="javascript">
        $(function() {
            var csrfToken = /*[[${_csrf.token}]]*/ null;
            var csrfHeader = /*[[${_csrf.headerName}]]*/ null;
            $(document).ajaxSend(function (e, xhr, options) {
                xhr.setRequestHeader(csrfHeader, csrfToken);
            });
        });
</script>
```
###### 이렇게 해주면, ajax 요청을 보낼 때, 헤더에 csrf 토큰을 추가하여 보내준다.

#### form을 통해 전송하는 경우(post전송시)
```javascript
<form method="post"  action="/login">
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
<-- 보내고자하는 데이터 -->
</form>
```
