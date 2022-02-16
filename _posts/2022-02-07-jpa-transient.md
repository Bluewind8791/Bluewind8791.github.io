---
layout: post
title: "[JPA] Transient"
description: spring jpa transient
summary: spring jpa transient
tags: spring jpa
---

- transient : lasting for only a short time; temporary:
- `@Transient`는 JPA의 표준이라 할 수 있는 `javax.persistence` 패키지에 포함되어있는 column mapping reference annoation 이다.
- `@Transient`는 **영속 대상에서 제외** 하기 위해 사용한다.
- 필드를 DB에 저장하지 않도록 해주며, 엔티티에서만 볼 수 있다.

## 두가지 방식

- ElementType.METHOD
- ElementType.FIELD

### Field 방식

필드 방식은 영속 대상에서 제외하고 싶은 필드에 @Transient 애노테이션을 선언.

### Method 방식

이 외에도 `@Transient`는 메서드에도 선언시킬 수 있다.


## 참조

- https://juneyr.dev/transient
- https://gmoon92.github.io/jpa/2019/09/29/what-is-the-transient-annotation-used-for-in-jpa.html