---
layout: post
title: "[JPA] QueryMethod의 정의 및 기본실습 1"
description: spring jpa
summary: spring jpa
tags: jpa spring
---

## find...By()

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByName(String name);
}
```
```java
@Autowired
private UserRepository userRepository;

@Test
void select() {
    System.out.println(userRepository.findByName("dennis"));
}
```

---

## ...By ...()

```java
// repository
User findByEmail(String email);
User getByEmail(String email);
User readByEmail(String email);
User queryByEmail(String email);
User searchByEmail(String email);
User streamByEmail(String email);
User findUserByEmail(String email);
User findSomethingByEmail(String email);
```
```java
// test
System.out.println("find by e-mail : " + userRepository.findByEmail("ben@email.com"));
System.out.println("read by e-mail : " + userRepository.readByEmail("ben@email.com"));
System.out.println("get by e-mail : " + userRepository.getByEmail("ben@email.com"));
System.out.println("query by e-mail : " + userRepository.queryByEmail("ben@email.com"));
System.out.println("search by e-mail : " + userRepository.searchByEmail("ben@email.com"));
System.out.println("stream by e-mail : " + userRepository.streamByEmail("ben@email.com"));
System.out.println("find user by e-mail : " + userRepository.findUserByEmail("ben@email.com"));
System.out.println("find something by e-mail : " + userRepository.findSomethingByEmail("ben@email.com"));
```

findSomethingByEmail() 로 동작시켜도 오류없이 돌아감.

---

## 접두사 뒤에 붙는 First, Top

```java
// repository
List<User> findFirst1ByName(String name);
List<User> findTop1ByName(String name);
```
```java
// test
System.out.println("find something by e-mail : " + userRepository.findFirst1ByName("Ben"));
System.out.println("find something by e-mail : " + userRepository.findTop1ByName("Ben"));
```
