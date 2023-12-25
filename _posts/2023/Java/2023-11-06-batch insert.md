---
layout: post
title: "Batch Insert (spring.data.jdbc)"
date: 2023-11-06
categories: [oauth]
---

# 문제

만약에 MySQL, Spring Data JPA를 개발환경으로 채택했다면 Entity를 save할때 생기는 문제는 무엇일까요?

CRUD를 최적화하는 고민중 C를 최적화 즉 쿼리를 최대한 적게 날리는 방법을 고민하고 서치해봤습니다.

여러 방법들이 있겠지만 조사해본 자료를 한번 뿌려보겠슴다

---

먼저 Bulk Insert가 뭔지 알아야 겠죠?

**단일쿼리**

```
INSERT INTO table1 (col1, col2) VALUES (val11, val12);
INSERT INTO table1 (col1, col2) VALUES (val21, val22);
INSERT INTO table1 (col1, col2) VALUES (val31, val32);

```

**Bulk Insert**

```
INSERT INTO table1 (col1, col2) VALUES
(val11, val12),
(val21, val22),
(val31, val32);

```

이런식으로 3개의 쿼리를 하나로 묶어서 처리하는 방식입니다.

잘만 최적화 하면 성능 최적화를 엄청 올릴 수 있겠죠?

이제 구현하러 가봅시다!

기존의 JPA `SaveAll` 을 사용하면 Query가 2개 썩 좋은 성능은 아닙니다..

Entity의 ID 생성전략이 INCREMENT로 설정하면 Hibernate가 JDBC 수준에서 batch insert를 비활성화한다고 나와있습니다.

[Hibernate ORM 12.2.1 Batch Inserts](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#batch-session-batch-insert)

그렇기 때문에 Spring Data JPA를 사용하는 환경이라면 BatchInsert 옵션이 꺼져있다는 뜻이죠.

왜 기본 값으로 BatchInsert를 비활성화했을까요?

[StackOverFlow 질문 참고](https://stackoverflow.com/questions/27697810/why-does-hibernate-disable-insert-batching-when-using-an-identity-identifier-gen)

ID 생성 Strategy인 **채번전략** IDENTITY를 사용할 때 Batch Support를 지원하면 Hibernate가 채택한 flush 방식(전형적인 Trasaction 처리방식인, write-behind)인 ‘Transactional Write Behind’와 충돌이 발생하기 때문에, IDENTITY 방식에서는 Batch Insert를 비활성화 한다는 뜻 같은데 Official Document에 나오지 않으니 정확히는 알 수 없을 것 같아요..

따라서 그냥 일상적으로 가장 널리 사용하는 IDENTITY 방식을 사용하면 Batch Insert는 동작하지 않습니다.

MySQL에서는 SEQUENCE를 제공하지 않으니 다른 DB에서 가능하다면 SEQUENCE를 적용해야 되겠습니당

그럼 결론이 바로 나오곘죠? JPA를 사용하지 않고 JDBC의 AUTO\_INCREMENT 방식이 아닌 채번전략으로 bulk insert를 구현하면 된다.

`

# 구현
## 1. Batch 채번(번호채택) 전략
### 채번 자체를 Batch로 처리하면 BulkInsert를 처리할 수 있다.

```
    @Id
    @GenericGenerator(
            name = "SequenceGenerator",
            strategy = "org.hibernate.id.enhanced.SequenceStyleGenerator",
            parameters = {
                    @Parameter(name = "sequence_name", value = "hibernate_sequence"),
                    @Parameter(name = "optimizer", value = "pooled"),
                    @Parameter(name = "initial_value", value = "1"),
                    @Parameter(name = "increment_size", value = "500")
            }
    )
    @GeneratedValue(
            strategy = GenerationType.SEQUENCE,
            generator = "SequenceGenerator"
    )
    private Long id;

```

이런식으로 id를 설정해주면 jdbc로 batch insert할 수 있습니다.

하지만 saveAll 같은 함수를 JPARepository 를 extends 한 Repository에서 가져오지 않고 JDBC에서 사용해야 하기 때문에 그 방법을 쓰고싶다면?

## 2. Spring Data JDBC
### `JdbcTemplate.batchUpdate` 사용

JdbcTemplate에는 Batch를 지원하는 batchUpdate() 메서드가 마련돼있다. 여러 가지로 Overloading 돼 있어서 편리한 메서드를 골라서 사용하면 되는데, 여기에서는 batch 크기를 지정할 수 있는 BatchPreparedStatementSetter를 사용하는 아래의 메서드를 사용해서 구현해본다.

`batchUpdate(String sql, BatchPreparedStatementSetter pss)`

이제 Repository를 따로 구현하면

- batchSize 변수로 배치 크기를 지정
- 전체 데이터를 배치 크기만큼 나눠서 Batch Insert를 실행
- 남은 데이터를 Batch Insert로 저장
1. ItemJdbcRepository를 따로 내부 기능을 JPA를 쓰지 않고 사용한다. (JdbcTemplate를 사용해서 )

```java
@Repository
@RequiredArgsConstructor
public class ItemJdbcRepositoryImpl implements ItemJdbcRepository {

    private final JdbcTemplate jdbcTemplate;

    @Value("${batchSize}")
    private int batchSize;

    public void saveAll(List<ItemJdbc> items) {
        int batchCount = 0;
        List<ItemJdbc> subItems = new ArrayList<>();
        for (int i = 0; i < items.size(); i++) {
            subItems.add(items.get(i));
            if ((i + 1) % batchSize == 0) {
                batchCount = batchInsert(batchSize, batchCount, subItems);
            }
        }
        if (!subItems.isEmpty()) {
            batchCount = batchInsert(batchSize, batchCount, subItems);
        }
        System.out.println("batchCount: " + batchCount);
    }

    private int batchInsert(int batchSize, int batchCount, List<ItemJdbc> subItems) {
        jdbcTemplate.batchUpdate("INSERT INTO ITEM_JDBC (`NAME`, `DESCRIPTION`) VALUES (?, ?)",
                new BatchPreparedStatementSetter() {
                    @Override
                    public void setValues(PreparedStatement ps, int i) throws SQLException {
                        ps.setString(1, subItems.get(i).getName());
                        ps.setString(2, subItems.get(i).getDescription());
                    }
                    @Override
                    public int getBatchSize() {
                        return subItems.size();
                    }
                });
        subItems.clear();
        batchCount++;
        return batchCount;
    }
}

```

이 방법은 Repository Interface를 만들어서 구현을 하고 hibernate를 쓰지 않고 삽입하는 로직입니다.

JPA를 혼용해서 bulk Insert할 수는 없을까요?

# 결론

#### ※ 1만건 이상의 데이터를 입력할 때는 Spring Data JDBC 의 batchUpdate를 사용한다.
 - Spring Data JPA와 혼용해서 Bulk Insert만 JDBC를 활용하는 전략을 활용해도 좋다.
 - Transactional 로 관리될 수 있어 추천
#### ※ SpringDataJPA를 활용시 MySQL를 쓰지 않고 Batch SEQUENCE를 사용하는 방법
- 기존에 IDENTITY, AUTO, TABLE을 사용하는 경우 변경이 어려울 수 있다.
- Batch 크기를 유동적으로 관리하기 어렵다.(다른 클래스의 Batch 사이즈만 바꾸기 어려울 수 있다. 싫으면 하드코딩..)

#### _참고하면 하루 100만건 이하 정도면 JPA를 써도 무방하다 라고 하네요._
서버 성능이 좋지 않은 경우는 그래도 적용해 보는게 좋지 않을까요?

### Reference

[Identity 채번 전략 성능 비교](https://github.com/HomoEfficio/dev-tips/blob/master/JPA-GenerationType-%EB%B3%84-INSERT-%EC%84%B1%EB%8A%A5-%EB%B9%84%EA%B5%90.md)

[이동욱님 블로그 글](https://jojoldu.tistory.com/558)

[Spring Data에서 Batch Insert 최적화](https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/)