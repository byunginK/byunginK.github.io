---
layout: post
title: "JPA 성능 최적화 (컬렉션 조회 & BatchSize) "
date: 2022-06-19
categories: [JPA]
---

## 컬렉션으로 조회
기본적으로 DTO 클래스를 생성하여 반환하는 방식을 그대로 사용

### fetch join
fetch join으로 성능 최적화를 하지 않으면 컬렉션에 담기는 모든 결과를 JPA가 쿼리를 통해 (지연로딩) 조회 하므로 무수히 많은 쿼리 호출

따라서, fetch join을 통해 SQL을 1번만 실행

- `distinct`를 사용하여 1대 N 조인시 발생하영 row 증가를 방지
- 만약 사용하지 않으면, order 엔티티의 조회 수도 증가하게 된다. `distinct`는 애플리케이션에서 중복을 걸러준다.

#### 단점
페이징 불가

- 컬렉션 fetch join을 사용하면 페이징 불가. 하이버네이트는 모든 데이터를 우선 DB에서 읽어 오고 __메모리에서 페이징을 한다.(매우 위험)__

## 페이징과 한계 돌파
### 컬렉션을 fetch join하면 페이징이 불가

### 해결법
1. ToOne(OneToOne, ManyToOne)관계는 모두 fetch join한다. (row수를 증가시키지 않기 때문에 괜찮다)
2. 컬렉션은 지연 로딩으로 조회
3. 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`, `@BatchSize`를 설정 (fetch join 시 지연로딩 쿼리 호출 시 where 절에 in 쿼리로 한번에 가져온다)
4. 쿼리 호출 수가 1 + N 에서 1+1로 최적화

- yml 파일에 글로벌로 설정 법
```yml
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true
        format_sql: true
        default_batch_fetch_size: 1000 #최적화 옵션
```
- order에서 ToOne은 지연로딩으로 가져오고 item을 가져올 때 인해 발생하던 여러 쿼리 호출이 한번에 가져온다
```java
    /**
     * V3.1 엔티티를 조회해서 DTO로 변환 페이징 고려
     * - ToOne 관계만 우선 모두 페치 조인으로 최적화
     * - 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화
     */
    @GetMapping("/api/v3.1/orders")
    public List<OrderDto> ordersV3_page(@RequestParam(value = "offset", defaultValue = "0") int offset,
                                        @RequestParam(value = "limit", defaultValue = "100") int limit) {

        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());

        return result;
    }


---
@Repository
public class OrderRepository {
	private final EntityManager em;

    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
}
```
- DTO 예시 (order를 불러오고 [지연로딩으로 

```java
    @Data
    static class OrderDto {

        private Long orderId;
        private String name;
        private LocalDateTime orderDate; //주문시간
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItemDto> orderItems;

        public OrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
            orderItems = order.getOrderItems().stream()
                    .map(orderItem -> new OrderItemDto(orderItem))
                    .collect(toList());
        }
    }

    @Data
    static class OrderItemDto {

        private String itemName;//상품 명
        private int orderPrice; //주문 가격
        private int count;      //주문 수량

        public OrderItemDto(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getOrderPrice();
            count = orderItem.getCount();
        }
    }
```

### 결론
ToOne 관계는 fetch join해도 페이징에 영향을 주지않는다. 따라서 ToOne 관계는 fetch join으로 쿼리 수를 줄이고 해결하고, 컬렉션은 `hibernate.default_batch_fetch_size`로 최적화

### `hibernate.default_batch_fetch_size` 크기 설정
- 1000개가 한계이며, 100 ~ 1000개 사이에서 선택한다.(권장) SQL IN 쿼리가 1000개 제한.
- 1000개로 잡을경우 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB에 순간 부하가 증가할 수 있다. (애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩 해야 하므로 메모리 사용량 동일)
- 1000으로 할 경우 성능상 좋지만 was가 어디 까지 버틸 수 있는 지에 따라 결정 해야 한다.
