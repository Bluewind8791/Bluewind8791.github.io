---
layout: post
title: "[Spring Security] DaoAuthenticationManager 와 UserDetailsService"
description: spring security UserDetailsService
summary: spring security UserDetailsService
tags: spring_security
---

# DaoAuthenticationManager 와 UserDetailsService

이때까지 User 정보는 모두 DB를 쓰지않고 테스트 및 학습용으로 간단하게 InMemory 상태에서 처리를 했다. 하지만 대부분은 MySQL, Oracle과 같은 RDBMS[^1] 를 사용하거나, MongoDB와 같은 기타 데이터베이스를 사용하여 DB를 관리한다.

[^1]: 관계형 데이터베이스 관리 시스템

실제로 스프링 시큐리티를 써서 서비스를 만들라고 하면 대부분의 개발자들은 UserDetails 를 구현한 User 객체와 UserDetailsService 부터 만든다.

왜냐하면 UserDetailsService 와 UserDetails 구현체만 구현하면 스프링 시큐리티가 나머지는 쉽게 쓸 수 있도록 도움을 주기 때문이다. 그런 다음 나머지 부분은 하나하나 설정을 배워가면서 처리하면 된다.

그래서 UserDetailsService 와 UserDetails 를 구현해 보자.

## 실습하기

- login-basic 프로젝트의 리소스와 컨트롤러를 재사용한다.

![6-task-map](https://bluewind8791.github.io/assets/image/6-userdetails.png)

- SpUser 객체와 SpAuthority 대한 Entity 객체 정의는 아래와 같이 정의 합니다.

```java
@Entity
@Table(name="sp_user")
public class SpUser implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    private String email;

    private String password;

    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    @JoinColumn(name = "user_id", foreignKey = @ForeignKey(name="user_id"))
    private Set<SpAuthority> authorities;

    private boolean enabled;

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public boolean isAccountNonExpired() {
        return enabled;
    }

    @Override
    public boolean isAccountNonLocked() {
        return enabled;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return enabled;
    }
}
```

```java
@Entity
@Table(name="sp_user_authority")
@IdClass(SpAuthority.class)
public class SpAuthority implements GrantedAuthority {

    @Id
    @Column(name="user_id")
    private Long userId;

    @Id
    private String authority;
}
```

## h2 DB 설정하기

```gradle
dependencies{
    runtime("com.h2database:h2")
}
```

```yaml
spring:
  h2:
    console:
      enabled: true
      path: /h2-console

  datasource:
    url: jdbc:h2:mem:userdetails-test;
    driverClassName: org.h2.Driver
    username: sa
    password:

  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
```
