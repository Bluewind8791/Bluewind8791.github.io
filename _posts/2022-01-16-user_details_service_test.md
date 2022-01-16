---
layout: post
title: "[Spring Security] DaoAuthenticationManager 와 UserDetailsService Test"
description: spring security UserDetailsService
summary: spring security UserDetailsService
tags: spring_security
---

# DaoAuthenticationManager 와 UserDetailsService Test

※ 본 코드는 [GitHub 페이지](https://github.com/Bluewind8791/Learning-Programming/tree/main/Learning-Spring/Part3/6-userdetails)에서 확인할 수 있습니다.

## 실습 하기

- login-userdetails 폴더를 만든다.
- user-admin comp 모듈을 만들고, SpUser, SpAuthority Entity와 Repository 를 만든다.
- h2 DB 를 세팅한다.

---

## 기본 세팅

- web 폴더에는 user-admin 이라는 웹 모듈을 만든다.
- comp 폴더에는 user-admin 이라는 component를 만든다.
- 그리고 gradle update를 하면 기본세팅이 완료된다.

## web-user-admin

- login-basic 의 아래의 코드를 web-user-admin의 폴더로 옮겨서 재활용한다.
  - resource - static, templates 
  - controller - HomeController

- build.gradle을 아래와 같이 변경한다.

```gradle
apply from: "../web-common.gradle"

dependencies {
}
```

## server-login-basic

- application.yml 에서 thymeleaf 설정을 한다.

```yaml
...
spring:
  ...
  thymeleaf:
    prefix: classpath:/templates/
    cache: false
    check-template-location: true
```

- build.gradle 에서는 web-user-admin 의 프로젝트를 link 한다. (코드를 옮겨서 재활용 했기 때문)

```gradle
dependencies {
    ...
    compile project(":web-user-admin")
}
```

## server-login-userdetails

- application.yml 은 login-basic 의 코드를 그대로 가져다 쓴다.

- build.gradle 설정

```gradle
apply from: "../web-common.gradle"

dependencies {

    implementation("$boot:spring-boot-starter-data-jpa")

    runtime("com.h2database:h2")
    
    compile project(":comp-user-admin")
    compile project(":web-user-admin")
}
```

- web-user-admin 과 comp-user-admin 프로젝트를 link 하는데,
- comp-user-admin 에는 user 와 authority DB 객체를 정의할 것이기 때문에 link 한다.
- runtime 에는 h2 DB를 사용

## comp-user-admin

user-domain 에는 `UserDetails` 를 implement 한 SpUser 객체를 만든다.

```java
public class SpUser implements UserDetails {
    unimplement methods...
```

- SpUser에서 `GrantedAuthority` 를 Set 으로 가지고 있어야 하니 `SpAuthority` 객체를 생성한다.

```java
@Data
@Entity
@Builder
@NoArgsConstructor
@AllArgsConstructor
@IdClass(SpAuthority.class)
@Table(name = "sp_user_authority")
public class SpAuthority implements GrantedAuthority {

    @Id
    @Column(name = "user_id")
    private Long userId;

    @Id
    private String authority;
}
```

- SpAuthority 에는 GrantedAuthority 를 implement 하고 lombok 선언을 한다.
- 이 클래스에는 userId 와 authority 를 가지고있다.
- 이 객체는 DB에 저장해야 하기 때문에 @Entity 를 달고
- table name 을 지정해준다
- 그리고 이 두 필드가 unique 해야하기 때문에 @IdClass 선언을 하고, 두 필드에 각자 @Id 를 달아준다.


- build.gradle 에는 jpa module 을 import 한다.

```gradle
dependencies {
    implementation("$boot:spring-boot-starter-data-jpa")
}
```

다시 SpUser 로 돌아와서

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "sp_user")
public class SpUser implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    @JoinColumn(name = "user_id", foreignKey = @ForeignKey(name="user_id"))
    private Set<SpAuthority> authorities;

    private String email;

    private String password;

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

- 이 객체도 동일하게 table 에 저장할 것이기 때문에 @Entity 를 달고 @Table name 을 지정해준다
- userId 를 @Id 로 설정하고 @GeneratedValue 는 IDENTITY 로 설정한다.
- `Set<SpAuthority> authorities` 는 
  - 한명이 여러가지 권한을 가지고 있을 수 있기 때문에 `@OneToMany` 로 join 한다. 
  - `fetch` 는 메모리에 올릴때 언제나 따라오도록 `EAGER` 로 지정하고, `cascade` 는 user 와 life cycle 이 같도록 `ALL` 로 지정한다.
  - `JoinColumn` 로 forign key 를 mapping 하는데 `SpAuthority` 의 PK 인 `user_id` 를 FK 로 지정한다.
- 여기에서는 email 를 username 으로 사용할 것이기 때문에 `getUsername` 메소드를 email 로 return 한다.
- `enabled` 로 `isAccountNonExpired` / `isAccountNonLocked` / `isCredentialsNonExpired` 를 대체한다.

DB 와 연동하도록 sevice 와 repository directory 를 만들고 객체를 생성한다.

```java
public interface SpUserRepository extends JpaRepository<SpUser, Long> {
    Optional<SpUser> findSpUserByEmail(String email);
}
```

- SpUserRepository 에는 Optional 로 email 로 유저를 찾아오는 메서드도 추가한다.

```java
@Service
@Transactional
public class SpUserService implements UserDetailsService {

    private final SpUserRepository userRepository; // (1)
    public SpUserService(SpUserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findSpUserByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException(username));
    }

    public Optional<SpUser> findUser(String email) {
        return userRepository.findSpUserByEmail(email);
    }

    public SpUser save(SpUser user) {
        return userRepository.save(user);
    }

    public void addAuthority(Long userId, String authority) {
        userRepository.findById(userId).ifPresent(user -> { // find by userId, if not null ->
            SpAuthority newRole = new SpAuthority(user.getUserId(), authority); // make new role
            if (user.getAuthorities() == null) { // if user has no authority
                HashSet<SpAuthority> authorities = new HashSet<>(); // make hash type authorities
                authorities.add(newRole); // add new role
                user.setAuthorities(authorities); // set authority
                save(user);
            } else if (!user.getAuthorities().contains(newRole)) { // else if user doesn't contain newRole
                HashSet<SpAuthority> authorities = new HashSet<>();
                authorities.addAll(user.getAuthorities());
                authorities.add(newRole);
                user.setAuthorities(authorities);
                save(user);
            }
        });
    }
    
    public void removeAuthority(Long userId, String authority) {
        userRepository.findById(userId).ifPresent(user -> { // find by userId, if not null ->
            if (user.getAuthorities() == null) { // if user has no authority -> return
                return;
            }
            SpAuthority targetRole = new SpAuthority(user.getUserId(), authority); // set target role
            if (user.getAuthorities().contains(targetRole)) { // if user has target role
                user.setAuthorities(                          // user get authority which isn't target
                    user.getAuthorities().stream().filter(auth -> !auth.equals(targetRole)).collect(Collectors.toSet())
                );
            }
        });
    }
}
```

- SpUser 가 UserDetails 를 implement 받기때문에 SpUserService 는 UserDetailsService 를 implement 받을 수 있다.
- 이 Service 는 SpUserRepository 를 참조하니 (1) 과 같이 선언한다.
- `UserDetailsService` 의 메소드인 `loadUserByUsername` 메소드는 email을 통하여 username 을 찾는 repository 메소드를 사용하고, 찾지 못했을 때는 `UsernameNotFoundException` 를 던져준다.
- 추가적으로 편의상 `addAuthority`, `removeAuthority` method 를 추가해준다.

## server-login-userdetails

- 이렇게 구현한 것들을 이제 login-userdetails 에서 사용할 수 있어야 한다.
- security 구현을 위하여 config directory 를 만들고 login-basic 의 SecurityConfig 를 복사해온다.

```java
@EnableWebSecurity(debug = true)
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final SpUserService userService; // (1)
    public SecurityConfig(SpUserService userService) {
        this.userService = userService;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService); // (2)
    }

    @Bean
    PasswordEncoder passwordEncoder() { // (3)
        return NoOpPasswordEncoder.getInstance();
    }
    ...
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()
                .requestMatchers(
                        PathRequest.toStaticResources().atCommonLocations(),
                        PathRequest.toH2Console() // (4)
                )
        ;
    }
}
```

- (1) `SpUserService` 를 가져온다.
  - `userService` 가 스캔이 안되어 있기 때문에 아래와 같이 repository, entity 스캔을 설정해준다.
  ```java
  @SpringBootApplication(scanBasePackages = {
      "com.sp.fc.user",
      "com.sp.fc.web"
  })
  @EntityScan(basePackages = {
      "com.sp.fc.user.domain"
  })
  @EnableJpaRepositories(basePackages = {
      "com.sp.fc.user.repository"
  })
  public class UserDetailsTestApplication {
      public static void main(String[] args) {
          SpringApplication.run(UserDetailsTestApplication.class, args);
      }
  }
  ```

- (2) AuthenticationManagerBuilder configure 은 기존의 inMemory 방식을 service를 등록하여 변경해준다.

- (3) H2 DB에서 user 정보를 넣었다가 뺏다 하기 위하여 편의상 `NoOpPasswordEncoder` 를 사용한다 (No Operation Password Encoder)
  - 실제 서비스에서는 사용해서는 안된다.

- (4) Test 시에 H2 Console 을 사용할 것이기 때문에 H2Console path 를 열어준다.

H2 DB를 사용할 것이기 때문에 application.yml 의 설정을 아래와 같이 변경한다.

```yml
...
spring:
  ...

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

## H2 Console

- <http://localhost:9055/h2-console> 의 도메인으로 들어가게 되면 login 창이 뜨는데
- URL 를 yaml 에서 설정하였듯이 `jdbc:h2:mem:userdetails-test` 로 변경하고 접속한다.
- 이 창에서 INSERT query를 통하여 SP_USER 과 'SP_USER_AUTHORITY' 테이블에 user 정보를 넣어서 접속한다.
- ADMIN 권한까지 주려고 하는 경우 'SP_USER_AUTHORITY' 테이블에서 'ROLE_ADMIN' 권한을 추가적으로 부여한다.
