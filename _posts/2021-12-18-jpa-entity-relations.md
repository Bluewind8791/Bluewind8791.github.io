---
layout: post
title: "[JPA] 연관관계 개요 및 ERD 알아보기"
description: spring jpa entity
summary: spring jpa entity
tags: jpa spring
---

# 연관관계 개요 및 ERD 알아보기

- ERD : Entity Relationship Diagram

- Entity는 양방향으로도 사용되는 경우가 많기때문에, ERD를 그려서 설계하는것이 좋음.

- 본 [코드](https://github.com/Bluewind8791/learning_programming/tree/main/Learning-Spring/part2_jpa/bookmanager)의 *'ERD.drawio'* 참고

---

## 1:1 연관관계 살펴보기

### @OneToOne

```java
public class BookReviewInfo extends BaseEntity {
    ...
    // private Long bookId;

    @OneToOne(optional = false, mappedBy = "bookReviewInfo")
    private Book book;
    ...
```

기존에 있던 Long bookId (FK)를 삭제하고, Book table을 one-to-one, 즉 일대일 매핑 관계를 추가했더니, create table할때, bigint book_id 가 자동생성되었다.

- optional : optional을 false로 지정하면 반드시 존재하는 값이 됨, 즉 null을 허용하지 않겠다는 의미(not null).
- mappedBy : book에는 bookReviewInfo_id가 존재하지만, bookReviewInfo 에는 book_id가 사라짐.  
  - 즉, 설정하게되면 연관키를 해당 테이블에서는 가지지 않게 됨.

```java
@OneToOne(mappedBy = "book")
@ToString.Exclude 
private BookReviewInfo bookReviewInfo;
```

toString을 제거하지 않으면, 순환참조가 되어서 stack overflow error가 걸리게됨.

---

## 1:N 연관관계 알아보기

- userHistory를 만들기위하여 여러개를 넣을수있는 collection type으로 지정.
- 1:N 타입의 OneToMany 어노테이션 지정.
- null point exception을 피하기 위하여 `new ArrayList` 기본생성자 지정.

```java
public class User extends BaseEntity {
    ...
    @OneToMany
    private List<UserHistory> userHistories = new ArrayList<>();
}
```

user id를 검색하여 user history를 불러오는 interface를 생성

```java
public interface UserHistoryRepository extends JpaRepository<UserHistory, Long> {
    List<UserHistory> findByUserId(Long userId);
}
```

Testing class 에서

```java
List<UserHistory> result = userHistoryRepository.findByUserId(
userRepository.findByEmail("daniel@email.com").getId());
```

위와같이 길게 코딩해야할것을 getUserHistories() method로 간단하게 코딩할 수 있다.

```java
List<UserHistory> result = userRepository.findByEmail("daniel@email.com").getUserHistories();
```

- log를 보면 user_user_histories 라는 의도하지않은 테이블이 생성되는데 이를 피하기 위하여 `@JoinColumn` 을 지정하는데, 이는 엔티티가 *어떤 컬럼으로 조인하게될지 지정해주는 어노테이션*이다.

```java
public class User extends BaseEntity {
...
    @OneToMany(fetch = FetchType.EAGER) // for LazyInitializationException
    @JoinColumn(name = "user_id", insertable = false, updatable = false)
    private List<UserHistory> userHistories = new ArrayList<>();
}
```

- JoinColumn의 name으로 필드명 지정.
- insertable과 updatable을 false로 지정함으로써 read only 속성으로 바꿈.

---

## N:1 연관관계 살펴보기

@ManyToOne annotation 사용

---

## N:N 연관관계 살펴보기

```sql
create table author_books (
   author_id bigint not null,
    books_id bigint not null
)
```

다대다 관계는 중간 entity 를 만들어서 일대다 일대다 로 처리한다.

- 주문 시스템인 경우 'User' 와 'Product' table이 존재.
- 'User' / 'Product' -> `@ManyToMany` relationship.
- 그럼 'user_products' 라는 중간테이블이 생성된다.
  - 'Order' 라는 Entity를 만들어서 중간테이블로 처리.
  - 'User' 과 'Order' 의 관계는 1:n 으로 설정.
  - 'Product' 와 'Order'의 관계는 1:n 으로 설정.
