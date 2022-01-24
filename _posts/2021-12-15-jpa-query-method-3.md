---
layout: post
title: "[JPA] QueryMethod로 정렬시켜 보기"
description: spring jpa
summary: spring jpa
tags: jpa spring
---

## OrderBy Asc / Desc

- Asc : 정순
- Desc : 역순

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // 맨 위의 1개만 가져오는데, name을 통하여 검색할것이고, id의 역순으로 정렬한다.
    List<User> findTop1ByNameOrderByIdDesc(String name); 

    // find first one by name && order by id desc and email asc
    List<User> findFirstByNameOrderByIdDescEmailAsc(String name);
}
```
```java
@Test
void pagingAndSortingTest() {
    log.info("findFirstByNameOrderByIdDesc : {}", 
        userRepository.findFirstByNameOrderByIdDesc("ben"));
    log.info("findFirstByNameOrderByIdDescEmailAsc : {}", 
        userRepository.findFirstByNameOrderByIdDescEmailAsc("ben"));
}
```

---

## Sort Class를 활용하여 정렬하기

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findFirstByName(String name, Sort sort);
}
```
```java
@Test
void pagingAndSortingTest() {
    log.info("findFirstByName WithSortParams : {}",
        userRepository.findFirstByName("ben", Sort.by(Order.desc("id"))));

    log.info("findFirstByName WithSortParams : {}", 
        userRepository.findFirstByName("ben", Sort.by(Order.desc("id"), Order.asc("email"))));
}
```

- name을 찾는데 Sort class(org.springframework.data.domain.Sort)를 활용하여 id기준으로 역순 정렬.
- Order 조건을 여러개 줄 수도 있음.

```java
@Test
void pagingAndSortingTest() {
    log.info("findFirstByName WithSortParams : {}",
        userRepository.findFirstByName("ben", getSort()));
}
```
```java
private Sort getSort() {
    return Sort.by(
        Order.desc("id"),
        Order.asc("email"),
        Order.desc("createdAt")
    );
}
```

이런식으로 코드의 가독성을 높힐 수 있다.
