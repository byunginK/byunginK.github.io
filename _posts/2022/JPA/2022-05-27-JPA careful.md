---
layout: post
title: "JPA 설계 주의점"
date: 2022-05-27
categories: [JPA]
---

# 주의점

1. setter 사용 하지 말자
   > 변경포인트가 너무 많아서 유지보수가 어렵다.
2. 모든 연관관계는 지연로딩으로 설정
   > 즉시로딩은 예측이 어렵고 JPQL 실행시 N + 1문제
   > 연관된 엔티티를 함께 조회를 해야한다면 `fetch join` 또는 엔티티 그래프 기능 사용
   > OneToOne, ManyToOne관계는 기본이 즉시로딩이므로 지연로딩으로 설정
3. 컬렉션은 필드에서 초기화하자

   > null에 안전
   > 하이버네이트는 영속화 할때 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경 컬렉션을 변경 및 잘못 생성하게되면 내부 매커니즘 오류 발생. (컬렉션은 되도록 변경 하지 말자)

4. cascade

5. 연관관계 편의 메소드

```java
@Entity
@Table(name = "orders")
@Getter
@Setter
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    //===연관관계 메서드===//
    public void setMember(Member member){
        this.member = member;
        member.getOrders().add(this); //member에 있는 orders의 리스트에 넣어주는것 까지 진행
    }

    public void addOrderItem(OrderItem orderItem){
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery){
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}
```
