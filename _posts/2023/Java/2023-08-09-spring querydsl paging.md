---
layout: post
title: "spring QueryDSL 페이징"
date: 2023-08-09
categories: [spring]
---

# Querydsl Paging 페이징 처리, Custom PageRequest 사용하는 이유

## **Querydsl Paging 페이징 처리**

프로젝트에서 Get요청을 통해 여러 건의 데이터를 가져올 때, 페이징(Paging) 처리가 필요한 경우가 많습니다. Spring Boot에서는 Pageable, PageRequest를 사용하여 페이징 처리를 하는데요.

페이징은 Pageable, PageRequest를 활용한 큰 틀 안에서 조금씩 다른 방법으로 사용될 수 있는데, 여기서는 Custom PageRequest를 사용한 페이징 처리에 대한 예시를 볼 수 있으며, 왜 Custom PageRequest를 만들어서 사용하는지에 대해서도 알 수 있습니다.


(해당 포스팅은 JPA, Querydsl을 사용할 수 있는 환경이 세팅된 상태에서 작업했으며, Querydsl 환경 설정은 바로 아래 글을 참조하시고, 페이징 전체 코드는 포스팅 맨 하단에 링크해놓겠습니다.)

[Querydsl 개념 및 Gradle 환경설정 (gradle-7.x.x)
- Querydsl 개념 및 Gradle 환경설정 QUser user = QUser.user; List result = queryFactory .select(user) .from(user) .where(user.name.eq("Jan")) .fetch(); // SELECT * FROM user WHERE user.name = 'Jan'..
wildeveloperetrain.tistory.com](https://wildeveloperetrain.tistory.com/92)

## **Pageable, PageRequest**

```java
org.springframework.data.domain
```

먼저 페이징에 사용되는 Pageable, PageRequest는 'org.springframwork.data.domain' 패키지 안에 있습니다.

Pageable은 페이징에 대한 정보를 담고 있는 인터페이스이며, AbstractPageRequest에서 implements 합니다. PageRequest는 추상 클래스인 AbstractPageRequest를 상속받은 페이징에 대한 정보를 담은 Pageable interface의 실제 구현 클래스입니다.

```java
    @GetMapping("")
    public ResponseEntity<?> getAll(Pageable pageable) {
        ...
    }
```

Spring Boot에서 쉽게 페이징 처리를 하는 방법 중 하나는 Controller에서 Pageable을 파라미터로 바로 받아서 PageRequest 객체로 변환하여 처리하는 것입니다.

**/boards?page=0&size=10&sort=title,desc&sort=writer,desc**

예를 들어 다음과 같은 쿼리를 보낸다면 Pageable에서 현재 페이지 정보인 page(0), 한 페이지에 노출할 데이터 수 size(10), 정렬 조건인 sort를 받아와 내부적으로 처리하는 것인데요. 이렇게 Pageable -> PageRequest로 받아서 처리할 경우 문제가 될 수 있는 부분이 있습니다.

첫 번째 문제는 Pageable의 size 값의 limit가 없다는 문제이고, 두 번째 문제는 page가 0부터 시작한다는 것입니다.

page가 0부터 시작하기 때문에 화면단에서 2페이지가 클릭되었을 때, pageable의 page는 0부터 시작되기 때문에 내부에서 처리 전 1을 빼줘야 하는 상황이 발생하며, 잘못하여 1을 빼주지 않는 경우 잘못된 데이터가 노출될 수 있습니다.

---

여기서 Pageable을 PageRequest 객체로 변환한다고 했는데, 이 원리는 Spring MVC에서 Pabeable에 사용을 지원하기 때문입니다.

Pageable 인스턴스는 페이징 정보를 담고 있는 객체로, 컨트롤러 메서드에 Pageable 인수를 전달할 때 Spring MVC는 자체적으로 PageableHandlerMethodArgumentResolver 클래스를 사용하여 Pageable 인스턴스를 구현체인 PageRequest로 변환합니다.

추가로 페이징 처리를 하는 다른 방법은 page, size, sort를 파라미터로 따로 받아서 PageRequest를 통해 전달하는 방법이 있는데, 이 방법은 번거로울 수 있고, 코드상 좋지 않다고 생각되기 때문에 사용하지 않았습니다.

## **Custom PageRequest**

```java
public class PageRequest {

    private int page = 1;
    private int size = 10;
    private Direction direction = Direction.DESC;

    public void setPage(int page) {
        this.page = page <= 0 ? 1 : page;
    }

    public void setSize(int size) {
        int DEFAULT_SIZE = 10;
        int MAX_SIZE = 50;
        this.size = size > MAX_SIZE ? DEFAULT_SIZE : size;
    }

    public void setDirection(Direction direction) {
        this.direction = direction;
    }

    public org.springframework.data.domain.PageRequest of() {
        return org.springframework.data.domain.PageRequest.of(page - 1, size, direction, "create_date");
    }
}
```

그래서 사용한 방법이 Custom PageRequest를 만들어서 사용하는 것인데요.

최종적으로 org.springframework.data.domain 패키지의 PageRequest 객체를 반환하는 것이며, 이 방법을 통해 Pageable의 size 값의 limit가 없는 부분을 보완할 수 있고, 입력받는 page 역시 그대로 입력받아 org.springframework.data.domain.PageRequest.of() 메서드 내에서 page 값을 빼주는 것으로 처리가 가능합니다.

## **Paging 페이징 처리 코드**

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/pagingTest")
public class PagingTestController {

    private final BoardRepository boardRepository;

    @GetMapping("")
    public PageImpl<Board> getAll(PageRequest pageRequest) {
        Pageable pageable = pageRequest.of();
        PageImpl<Board> result = boardRepository.getAll(pageable);
        return result;
    }
}
```

먼저 Controller입니다. (단순한 예시 코드이기 때문에 Service단은 생략하였습니다.)

Controller에서 받는 PageRequest는 바로 위에서 만든 Custom PageRequest이며, of() 메서드를 통해 Pageable으로 만들어서 넘기는 이유는 뒤에 paging 처리에서 페이징 정보를 가져와야 하는데 페이징 정보를 가져올 수 있는 get 메서드들이 Pageable에 있기 때문입니다.

```java
// BoardRepository interface
public interface BoardRepository extends JpaRepository<Board, Long>, BoardRepositoryCustom {
}

// BoardRepositoryCustom imterface
public interface BoardRepositoryCustom {
    PageImpl<Board> getAll(Pageable pageable);
}

// BoardRepositoryImpl class
public class BoardRepositoryImpl implements BoardRepositoryCustom {
    ....
}
```

repository 부분은 repository interface, repositoryCustom interface, repositoryImpl class 구조로 사용하였고, 핵심이 되는 repoistoryImpl 부분은 바로 아래에서 더 자세하게 살펴보겠습니다.

```java
public class BoardRepositoryImpl implements BoardRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    public BoardRepositoryImpl(JPAQueryFactory queryFactory) {
        this.queryFactory = queryFactory;
    }

    QBoard board = QBoard.board;

    @Override
    public PageImpl<Board> getAll(Pageable pageable) {
        List<Board> boardList = queryFactory.select(board)
                .from(board)
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        Long count = queryFactory.select(board.count())
                .from(board)
                .fetchOne();

        return new PageImpl<>(boardList, pageable, count);
    }
}
```

Querydsl에어 Pageable 인스턴스의 offset과 limit를 받아와 select에 사용하고, 최종적으로 PageImpl 객체로 반환할 때도 pageable을 함께 넘겨줍니다.

*(해당 쿼리의 전체 수량을 구하기 위해 count를 구하는 쿼리를 따로 날려주고 있습니다.)*

```java
    @Override
    public PageImpl<Board> getAll(Pageable pageable) {
        QueryResults<Board> results = queryFactory.select(board)
                .from(board)
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();
        return new PageImpl<>(results.getResults(), pageable, results.getTotal());
    }
```

---
### **추가로 알아두면 좋은 내용**

기존에는 Querydsl select 부분에서 QueryResult<> -> fetchResults() 방식을 사용했었는데, Querydsl 5.0 버전부터 fetchResults()와 fetchCount()가 deprecated 되었습니다.

이유는 모든 dialect에서 QueryResults로 count 쿼리를 날리는 것이 완벽하게 지원되지 않기 때문에 안전성을 위해 fetch()를 사용하라는 것인데요. 때문에 이렇게 사용하던 코드를 위 코드와 같이 변형하였습니다.

### **< 함께 보면 좋은 자료 >**

[Querydsl 개념 및 Gradle 환경설정 (gradle-7.x.x)
- Querydsl 개념 및 Gradle 환경설정 QUser user = QUser.user; List result = queryFactory .select(user) .from(user) .where(user.name.eq("Jan")) .fetch(); // SELECT * FROM user WHERE user.name = 'Jan'..
wildeveloperetrain.tistory.com](https://wildeveloperetrain.tistory.com/92)

[Querydsl DTO 조회하는 방법(Projection, @QueryProjection)
Projection 연산이란, - 한 Relation의 Attribute들의 부분 집합을 구성하는 연산자입니다. - 결과로 생성되는 Relation은 스키마에 명시된 Attribute들만 가집니다. - 결과 Relation은 기본 키가 아닌 Attribute..
wildeveloperetrain.tistory.com](https://wildeveloperetrain.tistory.com/94)