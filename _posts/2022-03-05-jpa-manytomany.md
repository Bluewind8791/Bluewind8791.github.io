---
layout: post
title: "[JPA] 다대다 연관관계 @ManyToMany"
description: jpa spring entity relation ManyToMany
summary: jpa spring entity relation ManyToMany
tags: jpa spring
---

## N:N 연관관계 살펴보기

실무에서는 잘 사용되지 않는다고 한다.

```sql
create table author_books (
   author_id bigint not null,
    books_id bigint not null
)
```

다대다 관계는 중간 entity 를 만들어서 일대다 일대다 로 처리한다.

- 예를 들어, 주문 시스템인 경우 'User' 와 'Product' 엔티티가 존재하게 된다.
- 그러면 한 유저는 여러 상품을 주문할 수 있고, 한 상품은 여러 유저에게 주문될 수 있다.

- 이 경우에는 'user_products' 라는 중간테이블을 생성한다.
  - 이 경우에는 'Order' 라는 이름의 중간테이블을 만들어서 처리한다.
  - 'User' 과 'Order' 의 관계는 1:N 으로 설정.
  - 'Product' 와 'Order'의 관계는 1:n 으로 설정하여 처리한다.
