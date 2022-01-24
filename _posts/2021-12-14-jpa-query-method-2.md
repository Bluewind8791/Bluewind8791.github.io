---
layout: post
title: "[JPA] QueryMethod의 정의 및 기본실습 2"
description: spring jpa
summary: spring jpa
tags: jpa spring
---

## And

- 두가지 조건 둘다 만족해야 결과값으로 리턴.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmailAndName(String email, String name);
}
```
```java
@Test
void select() {
    System.out.println("findByEmailAndName : " + userRepository.findByEmailAndName("ben@email.com","Ben"));
}
```

---

## Or

- 조건중 둘중 하나라도 맞다면 결과값으로 리턴.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmailOrName(String email, String name);
}
```
```java
@Test
void select() {
    System.out.println("findByEmailOrName : " + userRepository.findByEmailOrName("ben@email.com","Ben"));
}

```

---

## After

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByCreatedAtAfter(LocalDateTime yesterday);
    List<User> findByIDAfter(Long id);
}
```
```java
@Test
void select() {
    System.out.println("findByCreatedAtAfter : " + userRepository.findByCreatedAtAfter(LocalDateTime.now().minusDays(1L)));
    System.out.println("findByCreatedAtAfter : " + userRepository.findByIDAfter(4L));
}
```

---

## GreaterThan (>)

```java
public interface UserRepository extends JpaRepository<User, Long> {    
    List<User> findByCreatedAtGraterThan(LocalDateTime yesterday);
}
```
```java
@Test
void select() {
    log.info("findByCreatedAtGraterThan : {}", userRepository.findByCreatedAtGraterThan(LocalDateTime.now().minusDays(1L)));
}
```

---

## GreaterThanEqual (>=)

- … where x.age >= ?

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByCreatedAtGraterThanEqual(LocalDateTime yesterday);
}
```
```java
@Test
void select() {
    log.info("findByCreatedAtGraterThanEqual : {}", userRepository.findByCreatedAtGraterThanEqual(LocalDateTime.now().minusDays(1L)));
}
```

---

## Between

- between 조건은 양단의 조건을 포함하고 있음.  
  - (findByIdBetween 조건에서 결과값은 id 1,2,3 이 결과값으로 나옴)  
  - 즉, findByIdGraterThanEqualAndLessThanEqual() 과 같은 조건.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByCreatedAtBetween(LocalDateTime yesterday, LocalDateTime tomorrow);
    List<User> findByIdBetween(Long id1, Long id2);
}
```
```java
@Test
void select() {
    log.info("findByCreatedAtBetween : {}", 
        userRepository.findByCreatedAtBetween(LocalDateTime.now().minusDays(1L), LocalDateTime.now().plusDays(1L)));
    log.info("findByIdBetween : {}", 
        userRepository.findByIdBetween(1L, 3L));
}
```

---

## IsNotNull / IsNotEmpty

- IsNotNull : where user0_.id is not null
- IsNotEmpty : 평소의 not empty 의 의미는 "" 이나, JPA에서의 not empty 뜻은 collection type의 not empty이다. 잘 쓰이지 않는 구문.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByIdIsNotNull();
    List<User> findByIdIsNotEmpty();
}
```
```java
@Test
void select() {
    log.info("findByIdIsNotNull : {}", 
        userRepository.findByIdIsNotNull());
    log.info("findByIdIsNotEmpty : {}", 
        userRepository.findByIdIsNotEmpty()); // error
}
```

---

## In / NotIn

- In : or 과 비슷한 성능을 내는 쿼리.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameIn(List<String> names);
}
```
```java
@Test
void select() {
    log.info("findByNameIn : {}", 
        userRepository.findByNameIn(Lists.newArrayList("Ben", "Dennis")));
}
```

---

## StartingWith / EndingWith / Containing

- StartingWith : ~로 시작하는
- EndingWith : ~로 끝나는
- Containing : ~를 포함하는

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameStartingWith(String name);
    List<User> findByNameEndingWith(String name);
    List<User> findByNameContains(String name);
}
```
```java
@Test
void select() {
    log.info("findByStartingWith : {}", userRepository.findByNameStartingWith("Be"));
    log.info("findByEndingWith : {}", userRepository.findByNameEndingWith("en"));
    log.info("findByContains : {}", userRepository.findByNameContains("e"));
}
```

---

## Like

- findByNameLike("%e%") : contains
- findByNameLike("%en") : end with
- findByNameLike("Be%") : start with

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByNameLike(String name);
}
```
```java
@Test
void select() {
    log.info("findByNameLike : {}", userRepository.findByNameLike("%e%"));
    log.info("findByNameLike : {}", userRepository.findByNameLike("%en"));
    log.info("findByNameLike : {}", userRepository.findByNameLike("Be%"));
}
```

---

## Is

- Is 키워드는 큰 역할은 없지만 코드의 가독성을 위하여 생긴 쿼리 메서드.
