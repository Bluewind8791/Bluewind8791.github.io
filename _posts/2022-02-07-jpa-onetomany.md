---
layout: post
title: "[JPA] OneToMany"
description: jpa OneToMany entity relation
summary: jpa OneToMany entity relation
tags: jpa
---

## orphanRemoval

```java
@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true, mappedBy = "paper")
private List<PaperAnswer> paperAnswerList;
```

보통 1:N 관계 테이블 설정할 때 옵션을 추가해준다.

자식 엔티티의 변경이 있다면, JPA 에서 자식엔티티의 수정은 insert update update delete 순으로 이어지는데,

변경된 자식을 먼저 insert 하고, 기존의 자식을 NULL로 update 한다.

그리고 orphanRemoval 옵션을 true 로 하면 기존 NULL 처리된 자식을 DELETE 한다.

PK(JoinColumn)값이 NULL로 변한 자식은 고아객체라고 하여 연결된 점이 없는 객체이다. 

orphanRemoval 옵션은 바로 이 고아객체를 삭제해주는 역할을 한다.

이 속성은 참조하는 곳이 하나일 때만 사용해야 한다. 특정 엔티티가 개인 소유할 때만 사용해야한다. 

`@OneToOne` 과 `@OneToMany` 에서만 사용이 가능하다.


## mappedBy

연관관계의 주인을 정하는 방법은 `mappedBy` 속성을 사용하는 것이다.

-  주인은 `mappedBy` 속성을 사용하지 않는다.
-  주인이 아니면 `mappedBy` 속성을 사용해서 속성의 값으로 연관관계의 주인을 정할 수 있다.


## @JoinColumn 

JoinColumn 어노테이션은 외래 키를 매핑 할 때 사용한다. 



---

## 참고

- [현직개발자](https://dev-elop.tistory.com/entry/JPA-orphanRemoval-용도)
- https://victorydntmd.tistory.com/208