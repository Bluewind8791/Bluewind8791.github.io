---
layout: post
title: "[JPA] Embedded 활용하기"
description: spring jpa embedded
summary: spring jpa embedded
tags: spring jpa
---

# JPA 에서 Embedded 활용하기

- 예를 들어서, 유저의 주소 필드를 추가해야하는 task가 생겼다.
- User 에도, UserHistory 에도 다 아래의 값을 복붙해서 생성할수도있지만,

```java
private String city; // 시
private String district; // 구
private String detail; // 상세주소
private String zipCode; // 우편번호
```

이는 DRY 법칙에 어긋나기에 새로운 객체를 생성한다.

*[DRY 법칙]: Don't Repeat Yourself: 즉 복붙을 지양하고 객체화 시키는것이 적절하다.

```java
@Embeddable // ★
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Address {
    
    private String city;        // 시
    private String district;    // 구
    private String detail;      // 상세주소
    private String zipCode;     // 우편번호
}
```

- `@Embeddable` annotation을 달아서 이것이 임베드를 할 수 있는 클래스임을 지정한다.

```java
@Embedded
private Address address;
```

- 그리고 User entity 에는 이 변수가 임베디드임을 지정하는 `@Embedded` annotation을 달아준다.

그리고 findAll.forEach로 테스트를 진행해보면

```sql
create table user (
        ...
        city varchar(255),
        detail varchar(255),
        district varchar(255),
        zip_code varchar(255),
        ...

User(...address=null, ...)
```

- 위와같이 create table 할때 4가지의 필드를 생성하며, 결과값도 null로 찍히는것을 볼 수 있다.

- 여기에서 detail 필드가 주소의 detail인지, 유저의 detail인지 애매모호 하기때문에

```java
@Column(name = "address_detail")
private String detail;      // 상세주소
```

- `@Column`을 이용하여 이름을 지정해준다.

```sql
   create table user (
        ...
        city varchar(255),
        address_detail varchar(255),
        ...
```

```sql
User(..., address=Address(city=서울시, district=노원구, detail=한글비석로, zipCode=11111), ...)
```

- 주소를 이제는 집 주소와 회사 주소로 분할해서 각자 기록하고자 한다.

```java
private String homeCity;
private String homeDistrict;
private String homeDetail;
private String homeZipCode;
```

- User와 UserHistory에 전처럼 이런식으로 추가하기 보다는
- 기존에 만들었던 Address entity를 재활용한다.

```java
@Embedded
private Address homeAddress;

@Embedded
private Address companyAddress;
```

```java
@Test
void embedTest() {
    ...
    user.setHomeAddress(new Address("서울시", "노원구", "한글비석로", "11111"));
    user.setCompanyAddress(new Address("서울시", "성동구", "성수2로", "22222"));
}
```

- 이렇게 기존의 메소드를 재활용하여 테스트해보니 에러가 뜬다.
- Address를 두 번씩이나 선언했기때문에 같은 column 또한 두 번씩 선언된것이다. 그러므로 중복을 허용하지 않는다.
- 이때 사용하는것이 `@AttributeOverrides` 이다.

```java
@Embedded
@AttributeOverrides({
    @AttributeOverride(name = "city", column = @Column(name = "home_city")),
    @AttributeOverride(name = "district", column = @Column(name = "home_district")),
    @AttributeOverride(name = "detail", column = @Column(name = "home_address_detail")),
    @AttributeOverride(name = "zipCode", column = @Column(name = "home_zip_code"))
})
private Address homeAddress;

@Embedded
@AttributeOverrides({
    @AttributeOverride(name = "city", column = @Column(name = "company_city")),
    @AttributeOverride(name = "district", column = @Column(name = "company_district")),
    @AttributeOverride(name = "detail", column = @Column(name = "company_address_detail")),
    @AttributeOverride(name = "zipCode", column = @Column(name = "company_zip_code"))
})
private Address companyAddress;
```

위와 같이 User class 와 UserHistory class에 함께 `@AttributeOverrides`를 지정해준다.

그리고 `UserEntityListener` class에는 ↓와 같이 수정해주도록 하자.

```java
userHistory.setHomeAddress(user.getHomeAddress());
userHistory.setCompanyAddress(user.getCompanyAddress());
```

이제 테스트를 돌려보면...

```text
create table user (
    ...
    company_city varchar(255),
    company_address_detail varchar(255),
    company_district varchar(255),
    company_zip_code varchar(255),
    ...
    home_city varchar(255),
    home_address_detail varchar(255),
    home_district varchar(255),
    home_zip_code varchar(255),
    ...
    )
...
User(...homeAddress=Address(city=서울시, district=노원구, detail=한글비석로, zipCode=11111), 
    companyAddress=Address(city=서울시, district=성동구, detail=성수2로, zipCode=22222), userHistories=[])
...
UserHistory(...homeAddress=Address(city=서울시, district=노원구, detail=한글비석로, zipCode=11111), 
    companyAddress=Address(city=서울시, district=성동구, detail=성수2로, zipCode=22222))
```

이렇게 테스트 결과값이 도출되어 잘 동작하는것을 볼 수 있다.

---

그렇다면 address 필드에 null 값이 오면 어떻게 될까?

```java
@Test
void embedTest() {
    ...
    User user1 = new User();
    user1.setName("josh");
    user1.setHomeAddress(null);
    user1.setCompanyAddress(null);

    userRepository.save(user1);

    User user2 = new User();
    user2.setName("jordan");
    user2.setHomeAddress(new Address());
    user2.setCompanyAddress(new Address());

    userRepository.save(user2);
    
    userRepository.findAll().forEach(System.out::println);
    userHistoryRepository.findAll().forEach(System.out::println);
    }
```

위와 같이 josh 에는 null 값을 집어넣고, jordan 에는 빈 객체를 넣어준다.

```
User(... name=josh,..., homeAddress=null,
companyAddress=null, userHistories=[])

User(... name=jordan,..., homeAddress=Address(city=null, district=null, detail=null, zipCode=null),
companyAddress=Address(city=null, district=null, detail=null, zipCode=null), userHistories=[])
```

결과값은 위와 같이 약간은 다르게 테스트 결과값이 도출되었다.

```
[6, 2021-12-27 19:50:10.861632, 2021-12-27 19:50:10.861632, null, null, null, null, null, null, null, null, null, null, josh> ]

[7, 2021-12-27 19:50:10.992431, 2021-12-27 19:50:10.992431, null, null, null, null, null, null, null, null, null, null, jordan>]
```

- 테스트 메소드에서 `entityManager.clear();` 를 통하여 영속성 컨텍스트 캐쉬를 제거하고,
- `findAllRawRecord`의 native query를 통하여 `select * from user` 쿼리를 실행해주면

```
User(super=BaseEntity(createdAt=2021-12-27T20:49:55.690597, updatedAt=2021-12-27T20:49:55.690597), id=6, name=josh, email=null, gender=null, homeAddress=null, companyAddress=null, userHistories=[])

User(super=BaseEntity(createdAt=2021-12-27T20:49:55.816963, updatedAt=2021-12-27T20:49:55.816963), id=7, name=jordan, email=null, gender=null, homeAddress=null, companyAddress=null, userHistories=[])
```

- 위의 결과가 도출되는데, 임베드된 객체가 null인 경우 내부의 모든 컬럼이 null인 것과 동일하게 처리된다.
- embed된 null 로 set하게되면 내부에 들어있는 city, district 등의 필드가 모두 null 상태로 저장이 된다.
