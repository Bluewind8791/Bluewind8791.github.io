---
layout: post
title: "[JPA] JPA 소개와 간단한 명령어"
description: spring jpa
summary: spring jpa
tags: jpa spring
---

# JPA 소개

- 객체와 데이터베이스 사이의 관계를 연결해주는 것을 **ORM**(Object Relational Mapping)이라고 합니다.
- Java 진영에서는 ORM의 표준 스펙으로 **JPA**(Java Persistence Api)를 인터페이스로 정의하여 제공합니다.
- JPA의 실제 클래스들을 모아놓은 것이 **Hibernate**인데, 그 중에서 자주쓰이는 기능들을 더 사용하기 쉽도록 Spring Framework에서 제공하는것이 **Spring Data JPA** 입니다.

- Hibernate란 JPA에 대한 실제 구현체, 흔히 말하는 Implemetation 입니다.
- Spring Data Jpa 는 Spring 에서 Hibernate를 조금 간편하게 사용할 수 있도록 추상객체를 한번 더 감싸서 만들어 놓은것입니다.

---

## Simple CRUD Jpa Repository

- CRUD - Create, Read, Update, Delete

### ■ findAll()

table의 전체값을 가져오는 메소드

```java
User user1 = new User("Jack", "jack@email");
User user2 = new User("Samuel", "samu@email");
userRepository.saveAll(Lists.newArrayList(user1, user2));
List<User> users = userRepository.findAll();
users.forEach(System.out::println);
```

### ■ findById()

- 의미 그대로 Id(PK)값을 통하여 DB에서 찾음.

```java
Optional<User> user = userRepository.findById(1L);
System.out.println(user);
```

### ■ flush()

- 수동으로 영속화를 시킴. 자세한 내용은 뒤의 영속화 부분에서...

```java
userRepository.save(new User("name", "email"));
userRepository.flush();
// == saveAndFlush(new User("name", "email"));
userRepository.findAll().forEach(System.out::println);
```

### ■ count()

- DB의 수를 return 한다.

```java
long count = userRepository.count();
System.out.println(count);
```

### ■ existsById()

- 해당 id가 존재하는지 boolean으로 return 한다.

```java
boolean exits = userRepository.existsById(1L);
System.out.println(exits);
```

### ■ delete

- entity must not be null 이여야 하며, 해당 id에 해당하는 db값을 삭제한다.

```java
userRepository.delete(userRepository.findById(1L).orElseThrow(RuntimeException::new));
```

### ■ deleteById(index)

- select query -> delete

```java
userRepository.deleteById(1L);
```

### ■ deleteAll()

- findAll()후에 각 element를 delete 한다.

```java
userRepository.deleteAll();
```

### ■ deleteAllInBatch()

- query를 만들고 delete 실행, 쿼리 한번만 실행

```java
userRepository.deleteAllInBatch(userRepository.findAllById(Lists.newArrayList(1L, 3L)));
```

---

## Example (searching)

```java
ExampleMatcher matcher = ExampleMatcher.matching()
    .withIgnorePaths("name") // name은 매칭안함
    .withMatcher("email", GenericPropertyMatchers.endsWith()); // email 항목의 끝부분만 매치

Example<User> example = Example.of(new User("ma", "fastcam.com"), matcher);
userRepository.findAll(example).forEach(System.out::println);
```

- withIgnorePaths()에 name을 넣었기때문에 "ma"는 무시되며, fastcam.com으로 끝나는 email db를 조회함.

``` java
User user = new User();
user.setEmail("slow");

ExampleMatcher matcher = ExampleMatcher.matching().withMatcher("email", GenericPropertyMatchers.contains());
Example<User> example = Example.of(user, matcher);

userRepository.findAll(example).forEach(System.out::println);
```

- email 항목에서 slow라는 문자열이 있는지 매칭함.

---

## Save

```java
userRepository.save(new User("Ben", "ben@email.com"));

User user = userRepository.findById(1L).orElse(null);
user.setEmail("updated@email.com");

userRepository.save(user);
```

- 영속화를 시킴. 해당 내용은 Persistence에 자세하게 다룰 예정.
