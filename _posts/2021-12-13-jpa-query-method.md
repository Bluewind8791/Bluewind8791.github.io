---
layout: post
title: "[JPA] QueryMethod의 정의 및 기본실습 1"
description: spring jpa
summary: spring jpa
tags: jpa spring
---


## Simple CRUD Jpa Repository

- CRUD : Create, Read, Update, Delete

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
