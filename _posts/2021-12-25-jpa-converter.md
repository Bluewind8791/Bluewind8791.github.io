---
layout: post
title: "[JPA] Converter 사용하기"
description: spring jpa converter
summary: spring jpa converter
tags: jpa spring
---

# Converter 사용하기

- 책의 판매상태를 나타내는 필드를 추가하려고 한다.
- 일단 Book 클래스에 판매상태를 나타내는 변수를 하나 추가하고,
- 이 값을 활용하여 이 상품을 진열할것인지 결정하는 메소드를 추가한다.

```java
private BookStatus status; // 판매 상태

public boolean isDisplayed() {
    return status == 200;
}
```

DB에 status 필드를 추가하고

```java
@Test
void converterTest() {
    bookRepository.findAll().forEach(System.out::println);
}
```

- 테스트 코드를 작성해서 코드를 돌려보면, `status = 200` 이렇게 값이 잘 나온다.

- 하지만 이런 코드로 이루어진 데이터를 직접 활용하는것은 ORM 측면에서는 어긋났다고 볼 수 있다. 그러므로 우리에게 의미있는 객체로 바꿔보자.

- 먼저 repository.dto 에 `BookStatus` 클래스를 하나 생성한다.

```java
@Data
public class BookStatus {
    private int code;
    private String description;

    public BookStatus(int code) {
        this.code = code;
        this.description = parseDescription(code);
    }

    public boolean isDisplayed() {
        return code == 200;
    }

    private String parseDescription(int code) {
        switch (code) {
            case 100:
                return "판매종료";
            case 200:
                return "판매중";
            case 300:
                return "판매보류";
            default:
                return "미지원";
        }
    }
}
```

- 안에는 int 형의 코드가 있고 그것을 설명하는 description이 있다.
- 그리고 `parseDescription` 메소드를 만들어서 code 와 description의 관계를 설명한다.

- `BookStatus` 라는 객체를 만들었음으로 Book class의 status는 `BookStatus` 의 클래스를 가지게 되며
- `isDisplayed` 메소드는 Book에서 `BookStatus` 로 이관되어왔기 때문에 제거한다.

```java
private BookStatus status; // 판매 상태
```

그리고 domain.converter의 디렉토리에 `BookStatusConverter` 라는 Converter를 만든다.

```java
@Converter
public class BookStatusConverter implements AttributeConverter<BookStatus, Integer> {

    @Override
    public Integer convertToDatabaseColumn(BookStatus attribute) { // BookStatus 객체를 받아서 DB에 저장할때 어떻게 할것인지
        return attribute.getCode();
    }

    @Override
    public BookStatus convertToEntityAttribute(Integer dbData) { // db에서 int값을 받아서 bookstatus를 만듦
        return dbData != null ? new BookStatus(dbData) : null; // null에 대한 exception 처리
    }
}
```

그리고 Book class에 지정해놓았던 변수에는 `@Convert` annotation을 추가해주고, converter 값은 converter class의 값을 지정해준다.

```java
@Convert(converter = BookStatusConverter.class)
private BookStatus status; // 판매 상태
```

그리고 테스트를 돌려보면 아래와같이 status의 값이 잘 나온다.

```java
@Test
void converterTest() {
    Book book = new Book();
    book.setName("또다른 IT 전문서적");
    book.setStatus(new BookStatus(200));

    bookRepository.save(book);
    
    System.out.println(">>> " + bookRepository.findAll()); // error
}
```
```text
Book(super=BaseEntity(createdAt=2021-12-26T23:29:49.062238, updatedAt=2021-12-26T23:29:49.062238), id=1, name=JPA 초격차 패키지, category=null, authorId=null, deleted=false, status=BookStatus(code=100, description=판매종료))
```

setStatus 에 new를 통한 객체를 넣어 테스트를 해보려고했는데, `bookRepository.findAll` 메소드를 사용할 수 없어, 저번에 배운 `@Query` 만드는법을 활용하여 BookRepository에 native query 메소드를 하나 만들어서 테스팅 해본다.

```java
@Query(value = "select * from book order by id desc limit 1", nativeQuery = true)
Map<String, Object> findRawRecord();
```

```java
@Test
void converterTest() {
    ...
    System.out.println(">>> " + bookRepository.findRawRecord().values());
}
```

아래와 같이 200이라는 값이 객체를 통하여 setStatus를 했음에도 불구하고 code 값으로 DB에 들어가있음을 알 수 있다.

```sql
Hibernate: 
    select
        * 
    from
        book 
    order by
        id desc limit 1
>>> [4, 2021-12-27 15:45:16.817252, 2021-12-27 15:45:16.817252, null, null, false, 또다른 IT 전문서적, 200, null]
```
