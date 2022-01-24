---
layout: post
title: "[JPA] @Entity의 기본속성"
description: spring jpa entity
summary: spring jpa entity
tags: jpa spring
---

class annotation으로 **@Entity**를 추가하면, 메소드에 **@Id** annotation을 추가할 수 있으며, 이는 PK를 지정한다.

## @GeneratedValue

- Method Annotation
- `@Id`만 사용하거나 `@GeneratedValue`를 함께 사용하는 방법이 있다.
- GenerationType 4가지 전략이있다.

### GenerationType

- IDENTITY : @GeneratedValue(strategy = GenerationType.IDENTITY)
  - 일반적으로는 MYSQL에서 자주 사용함.
  - 기본키 생성을 데이터베이스에 위임.
- SEQUENCE : @GeneratedValue(strategy = GenerationType.SEQUENCE)
  - DB Sequence Object 사용.
  - Oracle 등에서 자주 사용.
  - H2 DB 또한 Sequence 사용함.
- TABLE : @GeneratedValue(strategy = GenerationType.TABLE)  
  - DB의 종류에 상관없이 ID값을 관리하는 별도의 테이블을 만들어두고, 그 테이블에서 아이디값을 추출하여 사용할수있다.
- AUTO : default 값

---

## @Table

- Class Annotation
- 알아서 Table을 생성함.
- name : (테이블 이름)은 자동으로 클래스 네임과 동일하게 설정되며, 동일하게 설정하는것이 좋기때문에 잘 설정하지 않는다.
- indexes : 인덱스 지정
- uniqueConstraints : unique 제약 조건

```java
@Table(name = "user", indexes = {@Index(columnList = "name")}, uniqueConstraints = {@UniqueConstraint(columnNames = {"email"})})
public class User {
    ...
}
```

---

## @Column

필드의 annotation.

- name : 해당 column의 이름을 지정한다.
- nullable : null type을 허용하지 않음.
- unique : true로 지정시 unique의 속성을 지니게 된다.
- length : 길이를 지정 (default : 255)
- updateable : false 지정시, update할 때 해당값을 저장하지 않음
- insertable : false 지정시, insert할 때 해당값을 저장하지 않음

```java
@Column(name = "createdAt", nullable = false)
private LocalDateTime createAt;
```

---

## @Transient

```java
@Transient
private String testData;
```

필드 annotation이며, DB 데이터에 반영되지않고 해당 데이터와 생명주기가 같아짐.

---

## @Enumerated

```java
public enum Gender {
    MALE,
    FEMALE
}

// repository
@Enumerated(value = EnumType.STRING)
private Gender gender;
```

enum 타입으로 Gender type을 하나 만들고, repository에 추가했을 때, `@Enumerated(value = EnumType.STRING)` 을 붙이지 않으면 0 과 1 등으로 추가되기때문에 예상치 못한 결과값이 나올 수 있다. 그러므로 enum을 사용할땐 enumerated를 활용.
