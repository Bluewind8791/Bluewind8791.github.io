---
layout: post
title: "[JPA] Native Query"
description: spring jpa native query
summary: spring jpa native query
tags: jpa spring
---

# Native Query

Native Query는 앞에서 배웠던 `@Query `에서 아래와 같이 `nativeQuery = true` 값만 지정해주면 된다.

```java
@Query(value = "select * from book", nativeQuery = true)
List<Book> findAllCustom();
```

아래의 테스트를 통하여 native query는 무엇이 다른지 알아보자.

```java
@Test
void nativeQueryTest() {
    bookRepository.findAll().forEach(System.out::println);
    bookRepository.findAllCustom().forEach(System.out::println);
}
```

아래는 첫번째 줄의 테스트값이다. 일전에 지정해놓은 where 값도 들어가서 deleted flag가 false인 결과값만 도출해낸다.

```text
Hibernate: 
    select
        book0_.id as id1_2_,
        book0_.created_at as created_2_2_,
        book0_.updated_at as updated_3_2_,
        book0_.author_id as author_i4_2_,
        book0_.category as category5_2_,
        book0_.deleted as deleted6_2_,
        book0_.name as name7_2_,
        book0_.publisher_id as publishe8_2_ 
    from
        book book0_ 
    where
        (
            book0_.deleted = 0
        )

Book(super=BaseEntity(createdAt=2021-12-26T16:43:07.902964, updatedAt=2021-12-26T16:43:07.902964), id=1, name=JPA 초격차 패키지, category=null, authorId=null, deleted=false)
Book(super=BaseEntity(createdAt=2021-12-26T16:43:07.917569, updatedAt=2021-12-26T16:43:07.917569), id=2, name=Spring Security 초격차 패키지, category=null, authorId=null, deleted=false)
```

native query를 사용한 두번째 결과값은 아래와 같다. @Query value에 적어놓은 값만 충실히 사용함으로써 3개의 db가 도출된다.

```text
Hibernate: 
    select
        * 
    from
        book

Book(super=BaseEntity(createdAt=2021-12-26T16:43:07.902964, updatedAt=2021-12-26T16:43:07.902964), id=1, name=JPA 초격차 패키지, category=null, authorId=null, deleted=false)
Book(super=BaseEntity(createdAt=2021-12-26T16:43:07.917569, updatedAt=2021-12-26T16:43:07.917569), id=2, name=Spring Security 초격차 패키지, category=null, authorId=null, deleted=false)
Book(super=BaseEntity(createdAt=2021-12-26T16:43:07.928287, updatedAt=2021-12-26T16:43:07.928287), id=3, name=SpringBoot AllInOne 패키지, category=null, authorId=null, deleted=true)
```

- 우리가 Book class에 @Where annotation 을 지정한 것이 native query 에서는 작동되지 않음을 알 수 있다.  
- Query문에 적은 쿼리를 그대로 활용한다.
- DB 종류가 변경이되었을때 코드변경없이 쿼리를 자동으로 변경해주는 JPA의 장점에서 조금 빗겨나간다.
- Native Query는 최소화하는것이 좋다.
- 그럼에도 불구하고 쓰는 이유는 성능에 대한 이점이있다.

## Native Query 의 이점 1

update query에는 `deleteAll` 처럼 한번에 실행하는 메소드가 없기때문에

```java
@Test
void nativeQueryTest() {
    List<Book> books = bookRepository.findAll();
    
    for(Book book : books) {
        book.setCategory("IT 전문서");
    }
    bookRepository.saveAll(books);
    System.out.println(bookRepository.findAll());
}
```

- 이와같이 테스트를 진행하면 현재는 DB에 두개밖에 없기때문에 update가 두번 쳐지지만, DB 가 천만개있다면 update를 천만번 이상 처야할수도있는 상황이 온다. 이는 심각한 성능저하를 불러올것이 뻔하다.

그러므로 아래와 같이 마이그래이션으로 배치하여 사용하는 경우가 많다.

```java
@Transactional
@Modifying
@Query(value = "update book set category = 'IT전문서'", nativeQuery = true)
int updateCategories();
```

그리고 아래와 같이 testing을 진행하면,

```java
@Test
void nativeQueryTest() {
    System.out.println("affected rows : " + bookRepository.updateCategories());
    bookRepository.findAllCustom().forEach(System.out::println);
}
```

아래와 같이 update를 한번만 때린다.

```text
Hibernate: 
    update
        book 
    set
        category = 'IT전문서'
affected rows : 3

Book(super=BaseEntity(createdAt=2021-12-26T17:19:05.210579, updatedAt=2021-12-26T17:19:05.210579), id=1, name=JPA 초격차 패키지, category=IT전문서, authorId=null, deleted=false)
Book(super=BaseEntity(createdAt=2021-12-26T17:19:05.232064, updatedAt=2021-12-26T17:19:05.232064), id=2, name=Spring Security 초격차 패키지, category=IT전문서, authorId=null, deleted=false)
Book(super=BaseEntity(createdAt=2021-12-26T17:19:05.240665, updatedAt=2021-12-26T17:19:05.240665), id=3, name=SpringBoot AllInOne 패키지, category=IT전문서, authorId=null, deleted=true)
```

하지만 여기에서는 위와같이 deleted 된 DB까지 함께 update 되는것을 알 수 있다.

## Native Query 의 이점 2

두번째 native query를 사용하는 경우는 JPA에서 기본적으로 지원하지않는 기능을 사용할때 사용한다.

```java
@Query(value = "show tables", nativeQuery = true)
List<String> showTables();
```

위와 같이 show tables라는 쿼리 기능을 사용할 수 있다.
