---
layout: post
title: "[JPA] QueryMethod로 Paging 처리하기"
description: spring jpa
summary: spring jpa
tags: jpa spring
---

```java
import org.springframework.data.domain.Page;  
import org.springframework.data.domain.Pageable;
```

- pageable은 paging에 대한 요청값
- page는 응답값.

## Paging

```java
Page<User> users= userRepository.findAll(PageRequest.of(1, 3));

System.out.println("page : " + users);
System.out.println("total elements : " + users.getTotalElements());
System.out.println("total pages : " + users.getTotalPages());
System.out.println("number of elements : " + users.getNumberOfElements()); // 현재 가져온 elements 수
System.out.println("sort : " + users.getSort());
System.out.println("size : " + users.getSize());

users.getContent().forEach(System.out::println);
```

---

## Find with Paging

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByName(String name, Pageable pageable);
}

@Test
void pagingTest() {
    log.info("findByName with paging : {}",
            userRepository.findByName("ben", PageRequest.of(0, 1, Sort.by(Order.desc("id")))));
}
```

첫번째 페이지를 id를 기준으로 역순으로 정렬하여 값을 가져온다.
