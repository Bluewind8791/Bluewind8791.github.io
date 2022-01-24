---
layout: post
title: "[JPA] JPA Cascade (영속성 전이)"
description: spring jpa cascade
summary: spring jpa cascade
tags: jpa spring
---

# JPA Cascade (영속성 전이)

- cascade : 작은 폭포
- 폭포가 위에서 아래로 물이 떨어지듯, 영속성도 하나의 객체에서 다음의 객체로 흘러보내주는 의미.

## CascadeType

- ALL : 모든 경우에 영속성 전이.
- PERSIST : 엔티티를 영속화 할 때이 필드에 보유 된 엔티티도 유지.
- MERGE : 엔티티 상태를 병합 할 때, 이 필드에 보유 된 엔티티도 병합.
- REMOVE : 엔티티를 삭제할 때, 이 필드에 보유 된 엔티티도 삭제
- REFRESH : entity를 refresh했을 때, 연관관계가 있는 entity도 함께 refresh.
- DETACH : 영속성을 관리하지 않는 상태. 영속성 컨텐스트로부터 분리(detach).
- default 값은 {} 이기 때문에 어떠한 영속성 전이도 일어나지 않음.

```java
@Test
@Transactional
void bookCascadeTest() {
    Book book = new Book();
    book.setName("JPA 초격차 패키지");
    bookRepository.save(book); // 영속화

    Publisher publisher = new Publisher();
    publisher.setName("FastCampus");
    publisherRepository.save(publisher); // 영속화

    // 연관관계 만들기
    book.setPublisher(publisher); 
    bookRepository.save(book); // 영속화

    publisher.addBook(book);
    publisherRepository.save(publisher); // 영속화

    System.out.println("books : " + bookRepository.findAll());
    System.out.println("publishers : " + publisherRepository.findAll());
}
```

위와 같은 경우에 영속화를 4번이나 하는것은 비효율적이기 때문에 이때 cascade를 사용한다.

```java
public class Book extends BaseEntity {
    ...
    @ManyToOne(cascade = CascadeType.PERSIST)
    @ToString.Exclude
    private Publisher publisher;
    ...
}
```

- Book이라는 entity가 persist될 때 publisher도 또한 같이 persist 전이가 일어나도록 설정한다.

- 그리고 test를 아래와같이 변경하면 테스트가 성공한다.

```java
@Test
@Transactional // lazy initialization exception
void bookCascadeTest() {
    Book book = new Book();
    book.setName("JPA 초격차 패키지");

    Publisher publisher = new Publisher();
    publisher.setName("FastCampus");

    book.setPublisher(publisher); // 연관관계 만들기
    bookRepository.save(book);
    
    System.out.println("books : " + bookRepository.findAll());
    System.out.println("publishers : " + publisherRepository.findAll());
    }
```
