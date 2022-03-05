---
layout: post
title: "[JPA] 일대다 다대일 연관관계 @OneToMany, @ManyToOne"
description: jpa spring OneToMany entity relation ManyToOne
summary: jpa spring OneToMany entity relation ManyToOne
tags: jpa spring
---

## 1:N / N:1 연관관계 알아보기

- 일대다 매핑은 테이블의 한 행이 다른 테이블의 여러 행에 매핑됨을 의미한다.
- 예를 들어, 하나의 카트에는 많은 상품을 담을 수 있으므로 여기에서는 일대다 매핑(`@OneToMany`)을 사용한다.
- null point exception을 피하기 위하여 `new ArrayList` 기본생성자를 지정 할 수 있다.

```java
@Entity
@Table(name="CART")
public class Cart {
    //...     

    @OneToMany(mappedBy="cart")
    private Set<Items> items;
    //...
}
```

- 또한 반대되는 item 엔티티에는 `@ManyToOne`을 사용하여 cart에 대한 참조를 추가하여 이를 양방향 관계로 만들 수 있다.
- 양방향이란 cart에서 item 항목에 액세스할 수 있고 item 에서 cart에 액세스할 수 있음을 의미한다.
- `mappedBy` 속성은 우리가 child class 에서 parent class를 나타내기 위해 어떤 변수를 사용하고 있는지 Hibernate에게 알려주는 데 사용한다.

```java
@Entity
@Table(name="ITEMS")
public class Items {

    //...
    @ManyToOne
    @JoinColumn(name="cart_id", nullable=false)
    private Cart cart;

    public Items() {}

    // getters and setters
}
```

- `@JoinColumn` 주석은 매핑된 컬럼을 참조한다.

---

## 연관관계의 주인

잘못된 시나리오 테스트 환경을 구현해본다.

```java
Cart cart1 = new Cart();
Cart cart2 = new Cart();

Items item1 = new Items(cart1);
Items item2 = new Items(cart2); 

Set<Items> itemsSet = new HashSet<Items>();
itemsSet.add(item1);
itemsSet.add(item2); 
cart1.setItems(itemsSet); // wrong!
```

- cart1, cart2, item1, item2 를 생성한다.
- item1 은 cart1에, item2는 cart2에 추가한다.
- 하지만 실수로 cart1에 item2를 집어넣은 상황이다.

이는 관계의 주인을 정함으로써 해결한다. 관계의 주인 측에 속하는 참조가 우선 적용되어 DB에 저장되게 된다.

- 연관관계의 주인은 `@ManyToOne` 측을 주인으로 정하는것이 좋다.
- 즉, 위와 같은 관계라면 item 엔티티가 주인이 되고, cart는 반대측이 된다.
- 위의 예제와 같이, 연관관계의 주인이 아닌 쪽에 `@OneToMany(mappedBy="cart")`를 달아준다.
- 마찬가지로 연관관계의 주인 측에는 `@ManyToOne` 어노테이션을 달아서 연관관계의 주인임을 나타낸다.

이제 위의 테스트 시나리오로 돌아가면, item2가 연관관계의 주인이기 때문에 item2의 참조를 DB에 저장한다.

그러므로 위의 테스트 시나리오의 결과는 아래와 같다.

```
item1 ID=1, Foreign Key Cart ID=1
item2 ID=2, Foreign Key Cart ID=2
```

---

## option

option       | description  | default
|   -        | -          | -
optional     | false 로 설정하면 연관된 엔티티가 항상 있어야 한다. | false
fetch        | 글로벌 fetch 전략을 설정한다 | @ManyToOne=FetchType.EAGER / @OneToMany=FetchType.LAZY
cascade      | 영속성 전이 기능을 사용한다.
targetEntity | 연관된 엔티티의 타입 정보를 설정한다. 이 기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제네릭으로 타입 정보를 알 수 있다.

---

## 양방향 매핑시 가장 많이 하는 실수

- 연관 관계의 주인이 아닌 값에 입력하는 것.
- `mappedBy` 에 필드 값을 수정하는 것.
- 무한 루프 조심, toString, lombok, JSON 생성 라이브러리
  - 스프링 부트는 컨트롤러 레이어에서 유저로 데이터를 반환할 때 `@ResponseBody` 로 반환하는 경우 Json 파싱을 하게 되는데, 이 때 Jackson 라이브러리를 사용한다.
  - 만약 엔티티를 컨트롤러에서 사용하게 되면 무한 참조가 발생하게 되는데, 처리하는 방법은 `@JsonIgnore` 나 엔티티를 사용하지 않는 방식이나


일반적으로 엔티티를 사용하지 않고 DTO 로 변환해서 사용하는 것이 매우 매우 매우 권장된다.

설계시에는 단방향 매핑만으로 끝내고, 역방향으로 참조해야 되는 경우만 양방향 매핑을 추가하는것을 권장한다.

---

## 참고

- https://www.baeldung.com/hibernate-one-to-many
- https://velog.io/@devsh/JPA-%EC%97%B0%EA%B4%80-%EA%B4%80%EA%B3%84-%EB%A7%A4%ED%95%91-OneToMany-ManyToOne-OneToOne-ManyToMany