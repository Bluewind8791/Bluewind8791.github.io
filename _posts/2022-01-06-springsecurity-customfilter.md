---
layout: post
title: "[Spring Security] Custom Token and custom Filter"
description: spring security authentication login custom_filter custom_token
summary: spring security authentication login custom_filter custom_token
tags: spring_security
---

## **학생과 선생님 로그인 구현**

---

![map]("https://user-images.githubusercontent.com/85560758/148631055-ee78bc30-eea7-4a91-b161-a05a6331aac6.png")

- Student는 로그인을 통하여 StudentAuthenticationProvider을 통하여 StudentAuthenticationToken을 발급받는다.
- Teacher은 로그인을 통하여 TeacherAuthenticationProvider을 통하여 TeacherAuthenticationToken을 발급받는다.

- StudentAuthenticationToken 과 TeacherAuthenticationToken 을 각각 Authentication 으로 한다.
- StudentManager와 TeacherManager 를 각각 AuthenticationProvider 로 구현한다.

## 문제점

- UsernamePasswordAuthenticationFilter 가 해주던 일을 직접 구현해야 한다.

  - CustomLoginFilter 를 쓸 경우 successHandler와 failureHandler 를 별도로 구현해 주어야 한다.
  - default 페이지와 caching 된 request 페이지로 redirect 하는 기능도 직접 구현해야 한다.
  - 현재로서는 CustomLoginFilter 와 UsernamePasswordAuthenticationFilter 를 동시에 사용하는 것이 가장 현명한 대안이다.

- 가능한 UsernamePasswordAuthenticationFilter 를 재활용하는 방안을 강구해야 한다.

※ 본 코드는 [깃헙 페이지](https://github.com/Bluewind8791/Learning-Programming/tree/main/Learning-Spring/Part3/4-1-login-custom-filter)에서 확인할 수 있습니다.

---

## **Custom Token**

### * Principle

Student에게 토큰을 발행하기 위하여 principle를 만든다.

`login-custom-filter.src.main.java`

```java
package com.sp.fc.web.student;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    private String id;
    private String username;
    private Set<GrantedAuthority> role; // GrantedAuthority 구현
}
```

### * AuthenticationToken

그리고 같은 디렉토리에 학생들이 로그인하여 받을 통행증인 StudentAuthenticationToken 객체를 구현한다.

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class StudentAuthenticationToken implements Authentication {

    private Student principal;
    private String credentials;
    private String details;
    private boolean authenticated; // 인증도장

    @Override
    public String getName() {
        return principal == null ? "" : principal.getUsername();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return principal == null ? new HashSet<>() : principal.getRole();
    }
}
```

- Authentication 을 implements 하면 많은 메서드들을 재정의 하게되는데
- principal을 Student로 정의하고, credentials와 details 정의, 인증 도장을 찍어줄 boolean type의 authenticated도 정의해주면 @Data로 인하여 대부분의 메서드들이 상쇄된다.
- getAuthorities 메서드 - Student 객체가 GrantedAuthority를 가지고 있기때문에 principal이 null이라면 새로운 빈 객체를 생성하고, 있다면 getRole로 넘겨준다.
- getName 메서드 - principal 이 null이라면 ""를 name으로 넣고, 있다면 getUsername으로 넘겨준다.

### * Authentication Provider

```java
@Component
public class StudentManager implements AuthenticationProvider, InitializingBean {

    private HashMap<String, Student> studentDB = new HashMap<>();

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        UsernamePasswordAuthenticationToken token = (UsernamePasswordAuthenticationToken) authentication;

        if (studentDB.containsKey(token.getName())) {
            Student student = studentDB.get(token.getName());
            return StudentAuthenticationToken.builder()
                .principal(student)
                .details(student.getUsername())
                .authenticated(true)
                .build();
        }
        return null;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication == UsernamePasswordAuthenticationToken.class;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Set.of(
            new Student("hong", "홍길동", Set.of(new SimpleGrantedAuthority("ROLE_STUDENT"))),
            new Student("puppy", "강아지", Set.of(new SimpleGrantedAuthority("ROLE_STUDENT"))),
            new Student("kitty", "고양이", Set.of(new SimpleGrantedAuthority("ROLE_STUDENT")))
        ).forEach(s ->
            studentDB.put(s.getId(), s)
        );
    }
}
```

- AuthenticationProvider로 사용할것이기 때문에 implements를 해주면 설정해야할 메소드가 supports와 authenticate가 있다.

- supports 메소드는 이 매니저가 관심을 갖고있는 대상은 UsernamePasswordAuthenticationFilter을 통하여 Token을 받을것이기 때문에, 이 클래스의 토큰을 받으면 검증을 해주는 provider로 동작을 하겠다고 선언한다.

- authenticate 메소드가 중요한데, 우리는 StudentAuthenticationToken으로 발행을 하고자 한다. 하지만 UsernamePasswordAuthenticationToken을 대상으로 발행하기로 했기 때문에, 현재 토큰은 UsernamePasswordAuthenticationToken이 된다.

- 원래는 Student DB를 핸들링하여 가져와야 하지만 테스트라서 간단하게 구현한다.  
  - `private HashMap<String, Student> studentDB = new HashMap<>();`
  - InitializingBean을 implements 하여 Bean이 초기화되었을 때 afterPropertiesSet 메서드를 통하여 학생의 정보를 put하여 DB를 구성한다.
  - 그러면 token에서 `studentDB.put(s.getId(), s)` id값이 return 될 것이다.

- if문을 통하여 token에서 name을 가져와서 db와 일치하면, db에서 student name을 가져오고, StudentAuthenticationToken으로 만들어서 전달해준다.  
이때 principal을 student로 주고, credentials와 details는 여기에서는 굳이 필요없으며, authenticated를 true값으로 줘서 빌드하여 return한다.

- 여기에서 매칭되지 않아서 authenticated를 false로 주게되면, 어쨋든 핸들링이 되었기 때문에 문제가 된다. 그래서 처리할 수 없는 auth는 null로 넘겨준다.

### * AuthenticationProvider를 AuthenticationManager에 등록하기

SecurityConfig로 넘어가서.

```java
@EnableWebSecurity(debug = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final StudentManager studentManager;

    public SecurityConfig(StudentManager studentManager) {
        this.studentManager = studentManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(studentManager);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests(request->
                request.antMatchers("/").permitAll() // root page permit all
                .anyRequest().authenticated() // any request authenticated
            )
        .formLogin(login -> 
            login.loginPage("/login") // 로그인 페이지 설정
            .permitAll() // redirection error
        )
            ;
    }
    ...
}
```

- 먼저 AuthenticationManagerBuilder configure에서 authenticationProvider를 studentManager로 등록한다.
- studentManager은 final로 지정하고 constructor로 주입한다.

- 이 studentManager가 동작하기 위해서는 usernamePasswordAuthenticationFilter를 동작시켜야 하기때문에 formLogin을 등록해준다.
- login page를 설정하게되면 usernamePasswordAuthenticationFilter가 동작하게되고 usernamePasswordAuthenticationToken을 studentManager이 발행을 해준다.

### * Auth page 확인

```json
{
   "principal":{
      "id":"hong",
      "username":"홍길동",
      "role":[
         {
            "authority":"ROLE_STUDENT"
         }
      ]
   },
   "credentials":null,
   "details":"홍길동",
   "authenticated":true,
   "authorities":[
      {
         "authority":"ROLE_STUDENT"
      }
   ]
}
```

principal로 만들었던 student가 동작하고있으며 authorities로 ROLE_USER가 동작하고 있음을 알 수 있다.

> 결과적으로, UsernamePasswordAuthFilter가 UsernamePasswordAuthToken 발행 -> AuthManager는 토큰을 처리 할 Provider를 찾는다 -> StudentManager의 supports 메소드를 보고 StudentManager 에게 토큰을 넘겨서 처리하게 된다.

## * Teacher Authentication

앞의 Student의 Auth를 만들었듯, Teahcer, TeacherAuthenticationToken, TeacherManager 객체를 생성한다.

SecurityConfig 에서 teacherManager를 Provider로 추가한다.

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.authenticationProvider(studentManager);
    auth.authenticationProvider(teacherManager); // *
}
```

그러면 teacherManger을 통하여 토큰이 처리되서 로그인이 완료되는것을 알 수 있다.

---

## **Custom Filter**

config directory에 CustomLoginFilter 객체를 생성한다.

```java
public class CustomLoginFilter extends UsernamePasswordAuthenticationFilter {

    public CustomLoginFilter(AuthenticationManager authenticationManager) {
        super(authenticationManager);
    }
    
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException {
        String username = obtainUsername(request);
        username = (username != null) ? username : "";
        username = username.trim();
        String password = obtainPassword(request);
        password = (password != null) ? password : "";
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
```

- 이 커스텀필터는 UsernamePasswordAuthenticationFilter를 상속받는다.
- UsernamePasswordAuthenticationFilter 에서 보통 attemptAuthentication 이 토큰을 만드는 메서드이기 때문에 override하여 재정의를 해준다.
- AuthenticationManager 가 authenticationManager 를 줘야 UsernamePasswordAuthenticationFilter 가 동작하기 때문에 생성자에서 CustomLoginFilter 를 받아서 넘기도록 한다.

이제 SecurityConfig로 넘어가서

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    CustomLoginFilter filter = new CustomLoginFilter(authenticationManager());
    http
        .authorizeRequests(request->
            request.antMatchers("/", "/login").permitAll()
            .anyRequest().authenticated()
        )
    // .formLogin(login -> 
    //     login.loginPage("/login")
    //     .permitAll()
    //     .defaultSuccessUrl("/", false)
    //     .failureUrl("/login-error")
    // )
    .addFilterAt(filter, UsernamePasswordAuthenticationFilter.class)
    .logout(logout -> logout.logoutSuccessUrl("/"))
    .exceptionHandling(e -> e.accessDeniedPage("/access-denied"))
    ;
}
```

- 기존의 formlogin은 제외하고 필터를 만들어서 쓰는데, authenticationManager를 직접 bean 에서 받아다가 CustomLoginFilter에 넣도록 한다.
- 그리고 addFilterAt를 하는데, 커스텀 필터를 UsernamePasswordAuthenticationFilter가 있던 자리에 그대로 꽂아준다.
