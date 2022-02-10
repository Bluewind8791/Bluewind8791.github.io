---
layout: post
title: "[JPA] JPA Cascade (영속성 전이)"
description: spring jpa cascade
summary: spring jpa cascade
tags: jpa spring
---

## Cascade (영속성 전이) 란?

- cascade : 작은 폭포
- 폭포가 위에서 아래로 물이 떨어지듯, 영속성도 하나의 객체에서 다음의 객체로 흘러보내주는 의미.
- `@OneToMany` 나 `@ManyToOne`에 옵션으로 줄 수 있는 값.
- 즉, Entity의 상태 변화를 전파시키는 옵션이다.
- 만약 Entity의 상태 변화가 있으면 연관되어 있는(ex. `@OneToMany`, `@ManyToOne`) Entity에도 상태 변화를 전이시키는 옵션이다.
- default 값은 `{}` 이기 때문에 어떠한 영속성 전이도 일어나지 않음.

## CascadeType

- ALL : 모든 경우에 영속성 전이.
- PERSIST : 엔티티를 영속화 할 때이 필드에 보유 된 엔티티도 유지.
- MERGE : 엔티티 상태를 병합 할 때, 이 필드에 보유 된 엔티티도 병합.
- REMOVE : 엔티티를 삭제할 때, 이 필드에 보유 된 엔티티도 삭제
- REFRESH : entity를 refresh했을 때, 연관관계가 있는 entity도 함께 refresh.
- DETACH : 영속성을 관리하지 않는 상태. 영속성 컨텐스트로부터 분리(detach).

## Cascade 적용

Parent - Child 관계에 있는 도메인에 적용할 수 있다.

Food 도메인과 Company 도메인을 만들어 적용해보자.

- Company 도메인에는 id, 회사 이름, 회사 로고 이미지와 만든 음식들이 존재한다.
- 한 회사에는 여러가지 음식이 있기 때문에 OneToMany의 관계로 지정한다.

```java
...
@Entity
public class Company extends BaseTimeEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long companyId;

    private String name;

    private String image;
    
    @OneToMany(fetch = FetchType.EAGER, mappedBy = "company")
    @Builder.Default
    @ToString.Exclude
    private List<Food> foods = new ArrayList<>();

    public void addFood(Food food) {
        this.getFoods().add(food);
        food.setCompany(this);
    }
}
```

- Food 도메인에는 id, 음식 이름, 이미지와 카테고리 그리고 회사가 존재한다.
- 한 회사에는 여러가지 음식이 있기 때문에 ManyToOne의 관계이다.

```java
...
@Entity
public class Food extends BaseTimeEntity {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long foodId;

    private String name;

    private String image;
    
    @Enumerated(EnumType.STRING)
    private Category category;

    @ManyToOne
    @JoinColumn(foreignKey = @ForeignKey(name = "foodId"))
    Company company;
}
```

그리고 간단하게 테스트 환경을 만든다.

```java
@Transactional
@SpringBootTest
public class CompanyTest {

    @Autowired private CompanyRepository companyRepository;
    @Autowired private FoodRepository foodRepository;

    @Test
    void testAddFood() {
        Food food = new Food();
        food.setName("testFood");

        Company company = new Company();
        company.setName("testCompany");
        company.setImage("logoImage");
        company.addFood(food);
        companyRepository.save(company);

        System.out.println(">>> " + companyRepository.findAll());
        System.out.println(">>> " + foodRepository.findAll());
    }
}
```

위와 같이 테스트를 실행하면 당연하게도 Company Repository 에만 데이터가 저장되어 있고, FoodRepository 에는 아무것도 없다.

```
>>> [Company(super=BaseTimeEntity(createdAt=2022-02-09T16:59:29.557061100, updatedAt=2022-02-09T16:59:29.557061100), companyId=1, name=testCompany, image=logoImage)]
>>> []
```

하지만 Company 도메인에 cascade 옵션을 준다면

```java
...
@Entity
public class Company extends BaseTimeEntity {
    ...

    @OneToMany(fetch = FetchType.EAGER, mappedBy = "company", cascade = CascadeType.ALL)
    @Builder.Default
    @ToString.Exclude
    private List<Food> foods = new ArrayList<>();
...
```

```
>>> [Company(super=BaseTimeEntity(createdAt=2022-02-09T17:02:26.854705300, updatedAt=2022-02-09T17:02:26.854705300), companyId=1, name=testCompany, image=logoImage)]
>>> [Food(super=BaseTimeEntity(createdAt=2022-02-09T17:02:27.010724200, updatedAt=2022-02-09T17:02:27.010724200), foodId=1, name=testFood, image=null, category=null, company=Company(super=BaseTimeEntity(createdAt=2022-02-09T17:02:26.854705300, updatedAt=2022-02-09T17:02:26.854705300), companyId=1, name=testCompany, image=logoImage))]
```

위의 결과값 처럼 자식객체(Food) 또한 함께 persist 되면서 저장된다.

---

## 다른 예제

아래와 같은 테스트가 있다.

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

---

## 참조

- 한번에 끝내는 Java/Spring 웹 개발 마스터 초격차 패키지 - 강현호 강사님의 강의
- Junseo Kim 님의 [블로그](https://velog.io/@max9106/JPA%EC%97%94%ED%8B%B0%ED%8B%B0-%EC%83%81%ED%83%9C-Cascade)