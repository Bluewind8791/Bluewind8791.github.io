---
layout: post
title: "[JPA] 일대일 연관관계 @OneToOne"
description: spring jpa entity OneToOne
summary: spring jpa entity OneToOne
tags: jpa spring
---

# @OneToOne


## 서론

- JPA entity 연관관계 중 1:1 연관관계를 나타낸다.
- 예를 들어, 한 유저는 한 개의 주소만을 가질 수 있다. 그럼 user entity 와 address entity 는 1:1 연관관계(one-to-one mapping)이다.

---

## FK로 구현하기

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;
    //... 

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private Address address;

    // ... getters and setters
}
```

- 이 경우, Address entity 와 `@OneToOne` 를 통하여 일대일 관계를 맺고있다.

- `@JoinColumn` 을 통하여 Address table의 기본 키에 매핑되는 User table의 열 이름을 구성할 수 있다.

- 또한, 연관관계의 주인만이 `@JoinColumn`을 지정할 수 있다.

```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;
    //...

    @OneToOne(mappedBy = "address")
    private User user;

    //... getters and setters
}
```

- 양쪽의 관계는 **양방향 관계**이기 때문에 Address entity 측에도 `@OneToOne` 를 설정해야 한다.

---

## PK를 공유하여 구현하기

- 위의 FK를 구현하는 전략에서는 User 테이블에 FK인 `address_id` 를 만들었지만, 
이 전략에서는 Address 테이블의 PK를 User 테이블에 대한 FK로 표시한다.

- 이는 저장 공간을 최적화하는 효과가 있다.

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;
    //...

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    @PrimaryKeyJoinColumn
    private Address address;

    //... getters and setters
}
```
```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @Column(name = "user_id")
    private Long id;
    //...

    @OneToOne
    @MapsId
    @JoinColumn(name = "user_id")
    private User user;

    //... getters and setters
}
```

- 외래키가 User쪽에 있었던 전과 달리 이제 Address 엔티티로 이동했기 때문에, `mappedBy` 속성을 준다.

- User 엔터티의 PK가 연결된 Address 엔터티의 외래 키 값으로 사용됨을 나타내는 `@PrimaryKeyJoinColumn` 어노테이션을 추가한다.

- Address 엔티티의 `@Id` 필드를 정의해야 하지만, user_id를 참조하기 때문에 더이상 `@GeneratedValue` 를 사용하지 않는다.

- 또한, User를 참조하는 필드에 PK 값이 User 엔터티에서 복사됨을 나타내는 `@MapsId` 어노테이션을 추가한다.

---

## Join Table 사용하기

- 지금까지의 위의 두가지 방법은 양쪽 관계가 필수적인 관계였다.
- 하지만 위의 관계에서 Address가 선택기입 사항이라 존재하지 않는 경우도 있을 수 있다.
- 이 경우에 이미 다룬 두가지의 같은 방법으로는 null 값을 넣어야 한다.
- 이런 상황에서는 join table 을 사용할 수 있다.

user | user_addr  | address
| -  | -          | -
id   | user_id    | id
...  | address_id | ...


위와 같이 entity를 구성하여 관계가 있을 때 마다 user_addr 테이블에 항목을 만들고 null을 방지한다.

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    //...

    @OneToOne(cascade = CascadeType.ALL)
    @JoinTable(name = "user_addr", 
      joinColumns = 
        { @JoinColumn(name = "user_id", referencedColumnName = "id") },
      inverseJoinColumns = 
        { @JoinColumn(name = "address_id", referencedColumnName = "id") })
    private Address address;
    //... getters and setters
}
```
```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;
    //...

    @OneToOne(mappedBy = "address")
    private User user;
    //... getters and setters
}
```

- `@JoinTable`은 관계를 유지하면서 조인 테이블 전략을 사용하도록 Hibernate에 지시한다.
- 또한 `@JoinTable` 을 사용하였기 때문에, 이 관계의 주인이 된다.



---

## 참고

```java
@OneToOne(mappedBy = "book")
@ToString.Exclude 
private BookReviewInfo bookReviewInfo;
```

- `@ToString.Exclude` 으로 toString을 제거하지 않으면, 순환참조가 되어서 stack overflow error가 걸리게됨.

---

## 출처

- https://www.baeldung.com/jpa-one-to-one
- [개발자의 기록습관](https://ict-nroo.tistory.com/126)