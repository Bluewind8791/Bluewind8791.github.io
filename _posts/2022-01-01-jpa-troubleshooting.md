---
layout: post
title: "[JPA] JPA Troubleshooting"
description: spring jpa troubleshooting
summary: spring jpa troubleshooting
tags: spring jpa
---

# Fetch Type = "EAGER"와 "LAZY"의 차이

`@OneToMany(fetch = FetchType.EAGER)`에서 EAGER와 LAZY의 차이를 확실하게 알아보자.

테스트 코드는 아래와 같다.

```java
@Test
@Transactional
void reviewTest() {
    List<Review> reviews = reviewRepository.findAll();

    System.out.println("전체를 가져왔습니다");
    System.out.println(reviews.get(0).getComments());
    System.out.println("첫번째 리뷰의 코멘트들을 가져왔습니다");
    System.out.println(reviews.get(1).getComments());
    System.out.println("두번째 리뷰의 코멘트들을 가져왔습니다");
}
```

---

### ● Fetch Type = EAGER

쿼리 결과 :

```text
Hibernate: 
    select
        ...
    from
        review review0_
    ...
전체를 가져왔습니다
[]
첫번째 리뷰의 코멘트들을 가져왔습니다
[]
두번째 리뷰의 코멘트들을 가져왔습니다
```

- `findAll`을 하는 시점에 각각의 코멘트를 각각 조회를 하게되어 쿼리가 먼저 실행되며,
- 그 이후에 `getComments`를 하게되더라도 추가적으로 쿼리를 실행하는것이 아니라
- 영속성 컨텍스트에서 가지고있는 캐쉬를 통하여 데이터가 출력된다.

### ● Fetch Type = LAZY

```text
Hibernate: 
    select
        review0_.id as id1_6_,
        review0_.created_at as created_2_6_,
        review0_.updated_at as updated_3_6_,
        review0_.book_id as book_id7_6_,
        review0_.content as content4_6_,
        review0_.score as score5_6_,
        review0_.title as title6_6_,
        review0_.user_id as user_id8_6_ 
    from
        review review0_
전체를 가져왔습니다
Hibernate: 
    select
        comments0_.review_id as review_i5_4_0_,
        comments0_.id as id1_4_0_,
        comments0_.id as id1_4_1_,
        comments0_.created_at as created_2_4_1_,
        comments0_.updated_at as updated_3_4_1_,
        comments0_.comment as comment4_4_1_,
        comments0_.review_id as review_i5_4_1_ 
    from
        comment comments0_ 
    where
        comments0_.review_id=?
[]
첫번째 리뷰의 코멘트들을 가져왔습니다
Hibernate: 
    select
        comments0_.review_id as review_i5_4_0_,
        comments0_.id as id1_4_0_,
        comments0_.id as id1_4_1_,
        comments0_.created_at as created_2_4_1_,
        comments0_.updated_at as updated_3_4_1_,
        comments0_.comment as comment4_4_1_,
        comments0_.review_id as review_i5_4_1_ 
    from
        comment comments0_ 
    where

```

- LAZY type 은 `findAll`을 할때 review 쿼리가 실행된다. 
- 그리고나서 `get(0)` 메소드가 나올때 쿼리를 실행하고, `get(1)` 메소드가 나오면 다시 쿼리가 실행된다.

- 이렇게 review 쿼리가 실행되고, 그 이후에 `getComment` 할때마다 각각의 id에 맞는 쿼리가 실행된다.  
- 필요한 시점에 쿼리가 실행되게 되니 일반적으로는 EAGER 타입 대비 성능에 대한 이점을 얻을 수 있다.

---

# JPA에서의 N+1 이슈

위와 같이 하위 entity들을 쿼리가 처음에 한개가 실행되어 한번에 가져오지 않고, 필요한 곳에서 사용되어 쿼리가 실행될때 n개가 실행되는 문제.

### 해결방법 1. Join Fetch

`@Query`를 이용하여 `join fetch`를 사용한다.

```java
public interface ReviewRepository extends JpaRepository<Review, Long> {
    
    @Query("select distinct r from Review r join fetch r.comments")
    List<Review> findAllByFetchJoin();
```

이렇게 하면 한줄의 쿼리로 결과값을 가져올 수 있으나, 불필요한 쿼리문이 추가된다.

### 해결방법 2. @EntityGraph

```java
public interface ReviewRepository extends JpaRepository<Review, Long> {
    
    @EntityGraph(attributePaths = "comments")
    @Query("select r from Review r")
    List<Review> findAllByEntityGraph();

    @EntityGraph(attributePaths = "comments")
    List<Review> findAll();
}
```

`@EntityGraph`의 `attributePaths`에 쿼리 수행시 바로 가져올 필드명을 지정하여 결과값을 가져오게 된다.

---

# 영속성 컨텍스트로 인해 발생하는 이슈

```java
public class Comment extends BaseEntity {
    ...
    @Column(columnDefinition = "datetime(6) default now(6)") // 초 단위까지만 기록하며 default는 now stamp 값으로 지정
    private LocalDateTime commentedAt;
}
```
```java
@Test
@Transactional
void commentTest() {
    Comment comment = commentRepository.findById(3L).get();
    comment.setCommentedAt(LocalDateTime.now());

    commentRepository.saveAndFlush(comment);

    // entityManager.clear();

    System.out.println(commentRepository.findById(3L).get());
```

cache를 사용하여 결과값을 도출
> Comment(..., id=3, comment=it was not good, commentedAt=2022-01-01T19:15:14.919436900)

`entityManager.clear()` 하여 DB값을 가져온 결과값
> Comment(..., id=3, comment=it was not good, commentedAt=2022-01-01T19:14:16)

commentedAt 결과값을 보면 데이터베이스의 실제 값과 entity 간에 불일치가 발생하는 문제가 발생한다.


- 우리가 코딩을할때 캐쉬를 할때 별도로 사용한다고 하면, 해당 캐쉬와 실제값이 틀어지는 경우에 많은 문제들이 발생한다.
- 예를 들어, 이미 탈퇴한 회원인데 캐쉬가 남아있어 로그인이 된다거나 이런 심각한 문제가 발생할 수 있다.
- 그래서 실제 값과 캐쉬 간의 이격이 벌어지지 않도록 캐쉬를 관리해야하며, jpa에서 활용하는 entity cache는 우리가 관리하는것도 아니라서 문제가 발생하기 쉽다.

```java
@Data
@Entity
@DynamicInsert // *
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class Comment extends BaseEntity {
    ...
```

`@DynamicInsert` : insert 시점에 dynamic하게 정의를 한다라는 의미로, insert문에 data가 존재하는 것들만 포함시켜서 실행한다.

---

## JPA에서 DirtyCheck와 성능이슈

CommentService 라는 service bean을 하나 만들고

```java
@Service
public class CommentService {
    
    @Autowired
    private CommentRepository commentRepository;

    @Transactional
    public void init() {
        for (int i=0; i<10; i++) {
            Comment comment = new Comment();
            comment.setComment("comment");

            commentRepository.save(comment);
        }
    }

    @Transactional
    public void updateSomething() {
        List<Comment> comments = commentRepository.findAll();

        for(Comment comment : comments) {
            comment.setComment("comment2");
            commentRepository.save(comment);
        }
    }
}
```

```java
@SpringBootTest
public class CommentServiceTest {

    @Autowired
    private CommentService commentService;

    @Test
    void commentTest() {
        commentService.init();

        commentService.updateSomething();
    }
}
```

`updateSomething` method를 통하여 테스트를 진행해보자.

```text
update
    comment 
set
    updated_at=?,
    comment=?,
    commented_at=?,
    review_id=? 
where
    id=?
```

comment column만 업데이트를 하였지만, 위와 같은 쿼리가 코멘트의 숫자만큼 찍혀서 나오게 된다.
- `(※ update {table} set {column} = '변경할값' where {조건})`

이것을 어떻게 최적화 할 수 있을까?

- Entity로 업데이트를 처리하게되면 전체 컬럼에 대한 업데이트를 진행하게 된다.  
- 그래서 이러한 경우에는 `@DynamicUpdate`를 이용한다. 그러면 영향을 받은 컬럼에 경우에서만 처리를 해주게 된다.

```java
@Data
@Entity
@DynamicInsert
@DynamicUpdate // *
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class Comment extends BaseEntity {
```

(DynamicUpdate annotation을 적용한 예시.)

```text
update
    comment 
set
    updated_at=?,
    comment=? 
```

`DynamicUpdate`를 적용하게 되면 위와 같이 변경되는 컬럼만 업데이트되는 방식으로 쿼리가 간단하게 바뀌는것을 알 수 있다.

---

## Dirty Check

Dirty Check에 대하여 조금 더 알아보자.

- CommentService에 아래와 같이 comment 를 insert 하는 메소드를 만들어서

```java
@Transactional
public void insertSomething() {
    Comment comment = new Comment();
    comment.setComment("insert comment");
}
```

```java
@SpringBootTest
public class CommentServiceTest {

    @Autowired
    private CommentService commentService;

    @Test
    void commentTest() {
        commentService.init();

        commentService.insertSomething(); // *
    }
}
```

- 새로 생성된 코멘트(`new Comment()`)이고 영속화되어 있지 않기 때문에, 위와 같이 테스트를 진행하면 dirty check가 일어나지 않는다.
- 그래서 별도로 save를 통하여 영속화를 진행해야한다.

```java
    @Transactional
    public void insertSomething() {
        Comment comment = new Comment();
        comment.setComment("insert comment");

        commentRepository.save(comment);
    }
}
```

```java
@Transactional
    public void insertSomething() {
        Comment comment = commentRepository.findById(1L).get(); // *
        comment.setComment("insert comment");
```

- 하지만 위와 같이 repository에서 꺼내와서 영속성 컨텍스트로 관리되어 있는것을 꺼내와서 `insertComment`를 진행하게 되면 dirty check가 일어나서 영속화가 일어나게 된다.

- dirty check라는 기능은 미숙한 개발자에게는 save를 하지않았지만 update를 치게되어 예상치못한 상황이 자주 발생하며 하나하나 다 체크하기 때문에 성능적인 이슈가 발생한다.

- 그렇다면 dirty check를 하지않으려면 어떻게 해야할까?

```java
@Transactional(readOnly = true) // *
public void updateSomething() {

    List<Comment> comments = commentRepository.findAll();

    for(Comment comment : comments) {
        comment.setComment("update comment");
        commentRepository.save(comment);
    }
}
```

위와 같이 `Transactional`에 `readOnly` 값을 true로 달아주게 되면 세션에 flush 모드(FlushModeType)를 기존의 AUTO에서 MANUAL 형식으로 변경되어 dirty check 자체가 skip되게 된다.
