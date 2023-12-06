---
layout: post
title: "QueryDSL 동적 정렬 orderSpecifier"
date: 2023-10-16
categories: [spring]
---

# Springboot JPA Querydsl 동적 정렬 OrderSpecifier

### **SQL 동적 정렬이란?**

하나의 API에서 정렬 조건을 동적으로 변경해, 정렬 혹은 정렬 + 페이징을 진행하는 것을 의미합니다. 예시를 보며, 필요한 상황이 언제이며, 어떻게 해결하는지 알아가 보겠습니다.

### **어떤 경우에 필요할까?**

우리는 Querydsl만을 통해서가 아닌 Springboot Data JPA를 통해서도 쉽게, 페이징과 정렬을 할 수 있습니다. 하지만 문제 되는 경우는 같은 API 호출임에도 불구하고, 내림 차순 or 오름 차순과 같이 동적으로 정렬이 바뀌는 경우가 존재합니다. 물론 API를 2개 만들어 호출하면 문제없습니다. 하지만 리소스, 동작이 동일한데 API를 분리하는 것은 복잡성을 증가시킬 뿐입니다. 2가지의 방법으로 해결할 수 있습니다.

### 간단한 처리 **= SQL을 3개 작성하고, 분기 처리한다.**

가장 간단한 방법으로 아래와 같이 SQL을 3가지 작성하면 됩니다. Service계층에서 정렬 조건에 대해 null 검증을 진행한 후에 각 조건에 맞는 SQL을 실행하면 됩니다.

이렇게 Service 로직에서 분기를 이용해서 쿼리를 선택해 동적 정렬을 진행할 수 있습니다. 하지만 문제점이 있습니다.

**- 정렬 조건이 많아진다면??**

- **정렬 조건이 계속해서 변화한다면??**

**- 정렬뿐 아니라 페이징이 적용됐다면?**

계속해서 Service 로직을 변경하고, 쿼리 또한 계속해서 변경해야 합니다. 무엇보다 가장 큰 문제는 페이징이 적용됐을 때 쿼리 단에서 해결할 수 있어야 합니다.

**이러한 문제점을 해결할 수 있는 것은 Querydsl의 OrderSpecifier입니다.**

## **OrderSpecifier 적용**

**Querydsl 설정법은 넘어가고 바로 시작하겠습니다.**

```java
@RequiredArgsConstructor
@Repository
public class PersonQuerydslRepository {
 
    private final JPAQueryFactory jpaQueryFactory;
 
 
    public List<Person> findAll(OrderCondition orderCondition){
        OrderSpecifier[] orderSpecifiers = createOrderSpecifier(orderCondition);
 
        return jpaQueryFactory.selectFrom(person)
                .orderBy(orderSpecifiers)
                .fetch();
    }
 
    private OrderSpecifier[] createOrderSpecifier(OrderCondition orderCondition) {
 
        List<OrderSpecifier> orderSpecifiers = new ArrayList<>();
 
        if(Objects.isNull(orderCondition)){
            orderSpecifiers.add(new OrderSpecifier(Order.DESC, person.name));
        }else if(orderCondition.equals(OrderCondition.AGE)){
            orderSpecifiers.add(new OrderSpecifier(Order.DESC, person.age));
        }else{
            orderSpecifiers.add(new OrderSpecifier(Order.DESC, person.region));
        }
        return orderSpecifiers.toArray(new OrderSpecifier[orderSpecifiers.size()]);
    }
}
```

OrderSpecifier 객체를 만들고, Querydsl의 그래프 탐색을 통해서 손쉽게 정렬 조건을 추가할 수 있습니다. 여기서

OrderSpecifier 리스트로 선언한 이유는 현재 한 개의 정렬 조건이지만, 여러 개의 정렬 조건이 추가될 수도 있기에 배열 선언하고 넣어줍니다. (페이징이 필요하시면 넣으시면 됩니다.)

추가적으로 페이징 + 여러 개의 정렬 조건이 필요하다면 아래와 같이 작성하시면 됩니다.

```java
public List<Person> findAll(OrderCondition orderCondition) {
    OrderSpecifier[] orderSpecifiers = createOrderSpecifier(orderCondition);
 
    Pageable pageable = PageRequest.of(0, 10);
 
    return jpaQueryFactory.selectFrom(person)
            .orderBy(orderSpecifiers)
            .limit(pageable.getPageSize())
            .offset(pageable.getOffset())
            .fetch();
}
 
private OrderSpecifier[] createOrderSpecifier(OrderCondition orderCondition) {
    List<OrderSpecifier> orderSpecifiers = new ArrayList<>();
 
    orderSpecifiers.add(new OrderSpecifier(Order.DESC, person.name));
    orderSpecifiers.add(new OrderSpecifier(Order.DESC, person.region));
 
    return orderSpecifiers.toArray(new OrderSpecifier[orderSpecifiers.size()]);
}
```

## **정리**

동적 정렬을 진행하는 방법에 대해서 알아봤습니다. 단순히 Service 계층에서 분기 처리 후 정렬하는 것에는 유지보수와 페이징이 더해졌을 때 진행이 어렵습니다. 따라서 OrderSpecifier를 사용해서 동적으로 정렬을 진행하고, 페이징까지 진행할 수 있는 것을 살펴볼 수 있었습니다.