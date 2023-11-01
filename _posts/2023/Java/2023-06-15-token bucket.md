---
layout: post
title: "Token bucket 알고리즘"
date: 2023-06-15
categories: [Spring]
---

## Token Bucket 알고리즘

Leaky Bucket과 유사한 구조지만, Bucket을 FIFO 큐로 사용하지 않고, 트래픽을 제어하기 위한 제어용 토큰을 관리하는 용도로 사용한다. Leaky Bucket과의 가장 큰 차이는 요청을 처리할 때 leaky bucket은 constant한 속도로 처리할 수 있는 반면 token bucket은 버스트 요청도 허용한다.

구현할 때 주의사항은 token을 fixed window처럼 특정 기간 단위로 추가하는게 아니라 일정 rate(속도)에 맞추어 추가해주어야 한다. (1분에 10개면 6초당 1개씩 추가해주는식)

-   트래픽은 토큰의 유무에 따라 흐름을 제어받는다.
    -   버킷에 토큰이 있으면 요청을 허용하고, 토큰이 고갈되면 에러를 리턴한다.
-   버킷에 정해진 rate에 맞추어 토큰을 다시 채워준다(refill).
    -   특정 시간 구간동안 모든 토큰을 다 사용한 유저에게는 request를 허용하지 않는다.
-   토큰 버킷은 토큰이 배치되는 속도를 기반으로 액세스 속도를 제어한다.

그림을 보면서 생각해보자.

![](https://blog.kakaocdn.net/dn/mGDjK/btryJiUQTNa/Nvea3H59SZlSzexFxDVMeK/img.png)

1. bucket에는 d개만큼의 토큰이 채워져있다. bucket에는 최대 n개만큼의 토큰이 채워질 수 있다.

2. 앞으로는 1/r 초마다 의 속도만큼 토큰이 버킷에 채워진다. (n개를 넘지 않음)

3. 요청이 들어오게 되면 버킷에서 토큰을 꺼낸다. 토큰이 부족하면 요청을 거절한다.

버킷을 조금 더 시각화 해보면

![](https://blog.kakaocdn.net/dn/LiGpS/btryHG91CQ4/PR3ctNFixZYZpa2mis5N3k/img.png)

그림에서 보이듯 10:10에는 3개 토큰이 있고, 하나씩 줄다가 10:10:45에는 0개라 요청을 받지 않는다.

-   이 때부터 10:11까지는 요청을 받을 수 없음

그리고 10:11이 되고나니 토큰이 3개가 리필된다.

-   이제 요청을 받을 수 있음

### Token Bucket의 장점

1. Max concurrency관리에 용이하다

2. burst of request도 일정수준은 유연하게 처리 가능하다.

### Token Bucket의 단점

1. distributed환경에서 그냥 그대로 사용할 경우 race condition이 발생할 수 있다.


## Bucket4j 란?

Bucket4j는 Token bucket 알고리즘을 기반으로 하는 Java 속도 제한 라이브러리이다.  
트래픽을 제어하여 과도한 요청으로부터 서버의 자원을 보호할 수 있다.

-   Token Bucket 알고리즘  
    - Token이 담긴 Bucket을 정의하고, 요청을 처리하는 비용으로 Token을 지불하는 방식으로 처리에 제한을 설정한 알고리즘이다.  
    - Bucket에 남겨진 Token이 부족하면 요청은 거절된다.  
    - Bucket에 Token은 일정 시간이 지나면 다시 채워진다.

## Bucket4j 기능 소개

Bucket4j 라이브러리를 구성하고 있는 대표적인 클래스에 대해서 알아보자.

-   Refill  
    \- 일정시간마다 몇 개의 Token을 충전할지 지정하는 클래스이다.
-   Bandwidth  
    \- Bucket의 총 크기를 지정하는 클래스이다.
-   Bucket  
    \- 실제 트래픽을 제어하기 위해 사용되는 클래스이며, 앞에 설명한 Refill 클래스와 Bandwidth 클래스를 토대로 만들어진다.

## Bucket4j 적용

**\- build.gradle**

의존성을 명시해준다.

```java
dependencies {
    ...
    implementation 'com.github.vladimir-bukhtoyarov:bucket4j-core:7.1.0'
    ...
}
```

**- BucketService.java**

동일한 사용자별로 반복 요청을 제어한다고 가정하에 사용자를 구분하기 위해서 요청 IP를 쓴다고 가정해 보자.

이를 위해 요청 헤더의 'Host' 값을 사용하며, 동일한 IP로 요청 시 일정 트래픽이 초과할 경우 요청을 거절할 예정이다.

  
버킷은 아래의 기준으로 생성된다.

-   버킷의 총 크기는 5이다.
-   버킷 안에 토큰은 10초마다 1개씩 재성성된다.

즉, 10초 안에 동일 사용자가 총 6번의 요청을 한다고 가정하면 5번째 요청에 대한 응답까지는 정상적으로 받을 수 있으나 6번째 응답에 대해서는 응답을 받을 수 없게 된다. 

```java
import io.github.bucket4j.Bandwidth;
import io.github.bucket4j.Bucket;
import io.github.bucket4j.Bucket4j;
import io.github.bucket4j.Refill;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.time.Duration;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class BucketService {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    // 요청자 IP 추출
    private String getHost(HttpServletRequest httpServletRequest) {
        return httpServletRequest.getHeader("Host");
    }

    // 버킷 가져오기
    public Bucket resolveBucket(HttpServletRequest httpServletRequest) {
        return cache.computeIfAbsent(getHost(httpServletRequest), this::newBucket);
    }

    // 버킷 생성
    private Bucket newBucket(String apiKey) {
        return Bucket4j.builder()
                // 버킷의 총 크기 = 5, 한 번에 충전되는 토큰 수  = 1, 10초마다 충전
                .addLimit(Bandwidth.classic(5, Refill.intervally(1, Duration.ofSeconds(10))))
                .build();
    }

}
```

**\- BucketController.java**

사용자 요청을 처리하는 컨트롤러이다. Bucket 기능에 중점을 두어 다른 비즈니스 로직은 없는 상태이다.  
정상적인 요청의 경우,  

```java
@RequiredArgsConstructor
@Slf4j
@RestController
public class BucketController {

    private final BucketService bucketService;

    @GetMapping("/api/bucket/access")
    public ResponseEntity bucketAccess(HttpServletRequest request) {
        Bucket bucket = bucketService.resolveBucket(request);
        log.info("접근 IP = {}", request.getRemoteAddr());

        if (bucket.tryConsume(1)) { // 1개 사용 요청
            return ResponseEntity.ok("[정상응답] 잔여토큰 : " + bucket.getAvailableTokens());
        } else {
            return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS).body("[비정상응답] 트래픽 초과");
        }
    }

}
```

## Bucket4j 적용 확인

프로젝트에 트래픽을 제어하는 기능이 정상적으로 구현되었는지 테스트해 보자.  
테스트케이스와 기댓값은 아래와 같다.

1.  첫 번째 요청에 대한 기댓값 : 정상 응답
2.  두 번째 요청에 대한 기댓값 : 정상 응답
3.  세 번째 요청에 대한 기댓값 : 정상 응답
4.  네 번째 요청에 대한 기댓값 : 정상 응답
5.  다섯 번째 요청에 대한 기댓값 : 정상 응답
6.  여섯 번째 요청에 대한 기댓값 : 비정상 응답
7.  일곱 번째 요청에 대한 기댓값 : 정상 응답 (설정된 토큰 재생성 시간인 10초가 지난 후에 요청 시)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FJrr2q%2Fbtr0xQvRuJK%2FwSX7TQxVt3JWKkMwRXjby1%2Fimg.png)

Case 1. 첫 번째 요청 - 성공

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc5lusn%2Fbtr0r5N6DCb%2FRc5it6Z3hmBjf9Pdlj6jHK%2Fimg.png)

Case 2. 두 번째 요청 - 성공

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcw3FJa%2Fbtr0yhzYwLm%2FvtQx80IMASY8iVylLTPZG0%2Fimg.png)

Case 3. 세 번째 요청 - 성공

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FNXwKj%2Fbtr0w2wy8TY%2FgK3ZSBB1MhKH7g2TKBMZa0%2Fimg.png)

Case 4. 네 번째 요청 - 성공

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcw8dgQ%2Fbtr0oAVS5aH%2FNV2ZUmZvFLUgVGK9ZMvc3K%2Fimg.png)

Case 5. 다섯 번째 요청 - 성공

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FUhe2U%2Fbtr0yiyTbEP%2FCMfzIu8SaxqZMm73045qRk%2Fimg.png)

Case 6. 여섯 번째 요청 - 실패 \[트래픽 초과\]

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoejaJ%2Fbtr0yhNwNEZ%2FTeEMWX40ZuMtSGB8HwKxLk%2Fimg.png)

Case 7. 일곱 번째 요청 - 성공

---

> ### 참고 url
>
>token bucket 알고리즘	https://etloveguitar.tistory.com/126
>
>spring 구현 참고 1	https://caffeineoverflow.tistory.com/21
>
>rate limmit 이란	https://etloveguitar.tistory.com/126
>
>bucket4j 문서	https://github.com/MarcGiffing/
>
>baeldung bucket4j 구현	https://www.baeldung.com/spring-bucket4j