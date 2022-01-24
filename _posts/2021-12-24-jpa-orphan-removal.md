---
layout: post
title: "[JPA] JPA에서 OrphanRemoval 활용하기"
description: spring jpa orphan removal
summary: spring jpa orphan removal
tags: jpa spring
---

# JPA에서 OrphanRemoval 활용하기

아래와 같이 delete test를 실행해보자.

```java
void bookCascadeTest() {
    ...
    Book book2 = bookRepository.findById(1L).get();
    bookRepository.delete(book2); // delete

    System.out.println("books : " + bookRepository.findAll());
    System.out.println("publishers : " + publisherRepository.findAll());
    }
```

그러면 결과값이 books 만 삭제되고 `publishers` 는 여전히 결과값이 나오는데, 연관된 `publishers` 까지 삭제하기 위해서는

```java
@ManyToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REMOVE})
@ToString.Exclude
private Publisher publisher;
```

- Book 클래스에서 위와 같이 *cascade remove type* 를 지정해준다.

이렇게 하면 결과값이 아래와 같이 나옴을 알 수 있다.

```
books : []  
publishers : []
```

- cascade는 한 객체가 remove 액션을 취하게 되면, 포함하고있는 객체에 해당 영속성 이벤트를 전파하여 하위 entity까지 remove 시킨다.
- 하지만 setter and getter로 연관관계를 지워버리면 remove event가 발생하지 않는다.  

```java
public class Publisher extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(orphanRemoval = true)
    @JoinColumn(name = "publisher_id") // 중간 column을 없애기 위하여
    @ToString.Exclude
    private List<Book> books = new ArrayList<>();

    public void addBook(Book book) {
        this.books.add(book);
    }
}
```

- 이때 연관관계가 끊어진 데이터를 살려두기위해서는 `orphanRemoval = false`,
- 삭제하기 위해서는 `orphanRemoval = true` 로 설정한다.

---

## Soft Delete

하지만 현업에서는 대부분의 경우에는 delete query를 자주 사용하지않고 flag를 사용해서 지웠다라고 표시만 하는 경우가 많다.

```java
private boolean deleted;
```

```sql
insert into book(`id`, `name`, `publisher_id`, `deleted`) values (2, 'Spring Security 초격차 패키지', 1, false);
```

예를 들면, Book class에 위와 같이 boolean type의 delete flag를 추가하고, 마찬가지로 DB에도 추가해준다.

그리고 BookRepository class에 아래와 같은 method를 추가해주면 false 처리된 즉, 삭제되지 않은 결과값만 출력된다.

```java
List<Book> findAllByDeletedFalse();
List<Book> findByCategoryIsNullAndDeletedFalse();
```

---

## @Where

하지만 이는 개발자의 실수나 여러가지 요인으로 인해 불안정한 요소가 있기 때문에 아래와 같은 `@Where` annotation을 활용한다.

```java
@Where(clause = "deleted = false")
public class Book extends BaseEntity {
```

- Where annotation에는 `(clause:절(주어와 동사가 갖춰진 문장 구성 성분))`를 지정하며,
- 이곳에는 query clause를 적어준다. (위의 SQL 구절 참고)
