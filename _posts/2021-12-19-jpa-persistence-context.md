---
layout: post
title: "[JPA] 영속성 컨텍스트"
description: spring jpa persistence
summary: spring jpa persistence
tags: jpa spring
---

# 영속성 컨텍스트

- Persistence 영속화 : 사라지지않고 지속적으로 접근할 수 있다.
- JPA를 이해하는데 가장 중요한 용어
- "엔티티를 영구 저장하는 환경"이라는 뜻

---

## MYSQL DB 설정하기

application.yml setting

```yml
spring:
  h2:
    console:
      enabled: true
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true # 압축된 쿼리를 읽기 좋게 포멧
    generate-ddl: true # 자동으로 entity에서 활용하고있는것을 테이블을 만듦
    hibernate:
      ddl-auto: create-drop # schema.sql이 존재할시 무시됨
    # defer-datasource-initialization: true # for h2 database
  datasource:
    url: jdbc:mysql://localhost:3306/book_manager
    username: root
    password: 1234
server:
  port: 8080
```

이 후에 contextload test를 해보면 MySQL8Dialect 결과값이 나옴.

# Entity Cache

영속성 컨텍스트 내부에는 캐시가 있는데 이를 1차 캐시라고 한다.
영속 상태의 엔티티를 이곳에 저장한다.
1차 캐시의 키는 식별자 값(데이터베이스의 기본 키)이고 값은 엔티티 인스턴스이다.

```java
@SpringBootTest
public class EntityManagerTest {
    
    @Autowired
    private EntityManager entityManager;

    @Test
    void entityManagerTest() {
        System.out.println(entityManager.createQuery("select u from User u").getResultList());
    }
}
```

아래와 같은 쿼리를 동작시키는 내용.

```java
userRepository.findAll();
```

- Entity Manager은 Cache를 사용하고 있다.
- 이는 실제로 save method를 실행시키는 시점에 db에 반영되지 않으며, 영속성 컨텍스트와 실제 db 사이에 db 갭이 발생된다.

---

### **쓰기지연**

엔티티 매니저는 트랜잭션을 커밋하기 직전까지 내부 쿼리 저장소에 INSERT SQL을 모아둔다. 그리고 트랜잭션을 커밋할 때 모아둔 쿼리를 DB에 보낸다. 이것을 트랜잭션을 지원하는 쓰기 지연이라 한다.

---

### **flush()**

- Entity manager이 자체적으로 db의 영속화를 해주지만, `flush()`는 개발자가 원할때 db의 영속화를 시켜주는 메소드.

- 영속성 캐쉬에 쌓여서 아직 반영되지 않은 entity의 변경을 해당 메서드 실행시점에 모두 db에 반영해주는 역할.

- flush로 인해서 예상치못한 일이 발생할 수도 있음.

- ※ DB에 반영되는 시점
  - flush method
  - transaction이 종료되는 시점 :
    - 전체 로직에 대해서 `@Transactional`을 달지않으면 해당 라인이 transaction을 가지게 되며 자동으로 flush 됨.
    - 전체 로직에 `@Transactional`을 달면, 전체 로직이 실행되고, transaction이 끝나는 시점에 커밋이 일어나면서 auto flush가 실행됨.
    - 복잡한 조회의 조건의 jpql 실행되는 조건

---

# Entity Life Cycle

Entity 생애주기 알아보기

## 비영속 상태

- 비영속 상태: 영속성 컨텍스트가 해당 entity 객체를 관리하지 않는 상태.

```java
@Service
public class UserService {

    @Transactional
    public void put() {
        User user = new User();
        user.setName("new user");
        user.setEmail("newuser@email.com");
    }
}
```

위와 같이 service를 하나 만들고 아래와 같이 testing을 진행한다면, 영속화되지 않음.

```java
@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Autowired
    private UserRepository userRepository;

    @Test
    void testPut() {
        userService.put();

        System.out.println(">>> " + userRepository.findByEmail("newuser@email.com"));
    }
}
```

---

### 영속화 방법 1. Service에 userRepository 추가하고 save 진행

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void put() {
        User user = new User();
        user.setName("new user");
        user.setEmail("newuser@email.com");
        
        userRepository.save(user);
    }
}
```

---

### 영속화 방법 2. EntityManager를 활용하여 persist 진행

```java
@Service
public class UserService {
    @Autowired
    private EntityManager entityManager;

    @Transactional
    public void put() {
        User user = new User();
        user.setName("new user");
        user.setEmail("newuser@email.com");        

        entityManager.persist(user);
    }
}
```

---

### 영속화 방법 3. EntityManager가 알아서 영속화 진행

```java
@Service
public class UserService {

    @Autowired
    private EntityManager entityManager;

    @Transactional
    public void put() {
        User user = new User();
        user.setName("new user");
        user.setEmail("newuser@email.com");        

        entityManager.persist(user);

        user.setName("newUserAfterPersist");
    }
}
```

한번의 persist 후에 `user.setName("newUserAfterPersist")` 은 영속화가 되지않아 DB에 반영되지 않을것같지만, transaction이 종료되는 시점에 별도로 persist 메서드를 호출하지 않더라도 entity manager이 알아서 update 쿼리를 해서 영속화 시킴.

---

## 준영속상태 : detached

원래 영속화되었던 객체를 **분리(detach)**해서 영속성 컨텍스트 밖으로 꺼내는 동작.

```java
@Service
public class UserService {

    @Autowired
    private EntityManager entityManager;

    @Transactional
    public void put() {
        User user = new User();
        user.setName("new user");
        user.setEmail("newuser@email.com");        

        entityManager.persist(user);
        entityManager.detach(user);

        user.setName("newUserAfterPersist");
        entityManager.merge(user); // (1)
        
        entityManager.clear(); // (2)
    }
}
```

- 위와같이 detach한다면 아래의 `user.setName("newUserAfterPersist")`은 DB에 반영되지 않음.

- 그러나 (1) merge method를 진행한다면 DB에 반영됨.

- clear method (2) 를 진행한다면 persist까지 반영되며, merge 예약이 clear 되어버림 그래서 앞의 내용이 반영되기를 원한다면 앞에 `flush()` 메소드를 써야함.

- remove는 해당 entity를 더이상 사용하지 못함.

```java
@Service
public class UserService {
    @Autowired
    private EntityManager entityManager;

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void put() {
        ...
        User user1 = userRepository.findById(1L).get();
        entityManager.remove(user1);
    }
}
```
```java
    @Test
    void removeTest() {
        userService.put();
        userRepository.findAll().forEach(System.out::println);
    }
```

예를 들어 위와 같이 Id가 1번인 user1을 repo에서 꺼내와 remove를 실행하여 test에서 findAll을 실행하면 id=1의 값이 지워져서 나오지않음.
