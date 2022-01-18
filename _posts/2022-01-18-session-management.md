---
layout: post
title: "[Spring Security] Session Management"
description: spring security session
summary: spring security session
tags: spring_security
---

# 세션 관리

서버는 기본적으로 사용자를 보면서 판단할 수 없다. 서버는 로그인을 통해 요청을 보낸 사용자를 구분한다. 하지만 모든 요청에 아이디/패스워드를 물어볼 수는 없는 노릇이다. 

그래서 토큰을 발급하고, 세션에는 토큰을 저장해놓고 세션이 유지되는 동안, 혹은 remember-me 토큰이 있다면, 해당 토큰이 살아있는 동안은 로그인 없이 해당 토큰 만으로 사용자를 인증하고 요청을 처리해 준다.

그래서 악의적으로 정보를 취하고자 하는 해커는 세션을 탈취하기 위한 시도를 한다. 따라서 세션 관리에 헛점이 없도록 구성의 기본 내용을 잘 알아야한다.

# ConcurrentSessionFilter

- SessionRegistry 를 사용한다. 이 bean 을 이용하여 세션사용자(SessionInformation)를 모니터링 할 수 있다.

![ConcurrentSessionFilter](https://bluewind8791.github.io/assets/image/8-concurrent-session.png)

- 만료된 세션에 대한 요청인 경우, 세션 즉시 종료.
- 세션 만료에 대한 판단은 SessionManagementFilter 의 ConcurrentSessionControlAuthenticationStrategy 에서 처리한다.
- SessionInformation 의 expired 된 토큰이 들어오지 못하도록 관리하는 필터 -> 그것이 마치 동시접속 제어를 하는것과 같은 효과를 냄

- **문제점**
  - 서블릿 컨테이너(tomcat) 으로 부터 HttpSessionEventPublisher 를 리스닝 하도록 listener 로 등록해야 한다.
  - tomcat 의 session 과는 별도로 session 을 SessionRegistry 의 SessionInformation 에서 관리한다.
  - SessionRegistry 의 session 정보를 expire 시키면 tomcat 에서 session 을 아무리 허용하더라도 app 내로 들어와서 활동할 수 없다.
  - SessionRegistry 에는 Authentication 으로 등록된 사용자만 모니터링 한다.

# SessionManagementFilter

- SessionAuthenticationStrategy 라는 interface를 갖는데, 여기에서 여러가지 세션 인증 정책을 관리하도록 설정할 수 있다.

  - 세션 생성 정책
  - 세션 아이디 고정 설정
  - 동시 접근 허용 문제
  - 세션 타임아웃 문제

```java
http
  .sessionManagement(session->session
      .sessionFixation(fix->fix.changeSessionId())
      .maximumSessions(1) // 한 유저당 한 세션만 허용
      .key("remember-me")
      .tokenValiditySeconds(30*24*60*60)
      .maxSessionsPreventsLogin(false) // 새로들어온 세션은 인정 구 세션은 expire
      .expiredUrl("/session-expired") // expired session 사용자가 이동할 url
  )
```

![SessionManagementFilter](https://bluewind8791.github.io/assets/image/8-session-management.png)

- SessionFixationProtectionStrategy : 세션 고정 문제 해결
  1. 브라우저로 서버에 접속
  2. 정상 사용자의 브라우저에 자신의 브라우저 세선값을 주입한다.
  3. 정상 사용자가 해당 사이트에 로그인하기만 하면 내 브라우저로 마치 인증된 사용자인것처럼 리소스에 접근 가능.

  - 해결방법: 서버에서 로그인을 시도하면 이전 세션을 invalidate 시키고 새로운 새션을 만들어서 인증해준다.

- ConcurrentSessionControlAuthenticationStrategy
  - 동시 세션의 개수 제한.
  - RegisterSessionAuthenticationStrategy 와 함께 SessionRegistry 를 참조해 작업한다.

- 세션 정책
  - Always : 항상 세션을 생성.
  - Stateless : 생성하지 않음. 존재해도 사용하지 않음.
  - If_Required : 필요 시 생성.
  - Never : 생성하지 않음. 존재하면 사용.
- sessionManagement 는 스프링 필터에 따라서 달리 적용될 수 있다.
- 보통은 ALWAYS 나 STATELESS 로 극단적이게 사용한다.

```java
.sessionManagement(s -> s
  .sessionCreationPolicy(SessionCreationPolicy.ALWAYS)
  ...
)
```

---

## 세션 관리 실습

- 실시간으로 로그인한 사용자들의 세션 개수와 로그인한 URL을 모니터링 한다.
- 3명의 사용자를 만든다.
- 동시접속 제어를 테스트 한다.
- RememberMe 설정.
- RememberMe 를 설정했을때

## Session Monitoring Page

세션을 모니터링 할 수 있는 페이지를 만든다.

```java
package com.sp.fc.web.controller;
...
@Controller
public class SessionController {

    @Autowired
    private SessionRegistry sessionRegistry
```
- SessionRegistry 는 SecurityConfig 에 SessionRegistryImpl 을 구현한다.
    ```java
    @Bean
    SessionRegistry sessionRegistry() {
        SessionRegistryImpl registryImpl = new SessionRegistryImpl();
        return registryImpl;
    }
    ```

```java
    @GetMapping("/sessions")
    public String sessions(Model model) { // (1)
        model.addAttribute("sessionList", // (3)
            sessionRegistry.getAllPrincipals().stream().map(p -> UserSession.builder() // (2)
                .username(((SpUser)p).getUsername())
                .sessions(sessionRegistry.getAllSessions(p, false).stream().map(s ->
                    SessionInfo.builder()
                        .sessionId(s.getSessionId())
                        .time(s.getLastRequest())
                        .build())
                    .collect(Collectors.toList()))
        .build()).collect(Collectors.toList()));
        return "/sessionList";
    }
```

- (1) 이 session 들을 페이지에 넘겨줄 Model이 필요하다.
- (2) sessionRegistry 에서 먼저 principal 을 리스트로 가져오면, 이것을 UserSession 을 build 하여 mapping 한다.
- (3) 그리고 `sessionList` 의 값으로 이 값들을 넘겨준다.
- UserSession 에는 아래에서 생성한 username, sessions 가 있다.
  - username 은 p(principal)에서 username 을 가져오고
  - sessions 는 sessionRegistry 에서 principal 로 `getAllSessions` 를 하는데,
  - 만료된 세션은 포함하지 않고 (false) 살아있는 세션만 가져온다.
  - 그리고 steam map 을 진행하는데 `SessionInfo` 를 build 하여 만들어준다.
    - `SessionInfo` 에는 sessionId 와 time 이 존재한다.
    - sessionId 는 s 의 `getSessionId` 를 하고,
    - time 은 `getLastRequest` 로 마지막 요청 시간을 가져온다.
    - build 한 후에 이 값들을 `collect` 로 리스트 형식으로 가져온다

```java
    @PostMapping("/session/expire")
    public String expireSession(@RequestParam String sessionId) {
        SessionInformation sessionInformation = sessionRegistry.getSessionInformation(sessionId);
        if (!sessionInformation.isExpired()) {
            sessionInformation.expireNow();
        }
        return "redirect:/sessions";
    }
```

- 위의 메서드는 session 을 강제로 만료시키는 Post mapping 메서드이다.
- expire 할 session Id 를 가져오고
- sessionRegistry 에서 sessionId 로 SessionInformation 를 가져오고
- 만약 sessionInformation 가 expired 되지 않았다면 바로 expire 시켜버린다.

```java
    @GetMapping("/session-expired")
    public String sessionExpired() {
        return "/sessionExpired";
    }
}
```

- 만료된 세션이 갈 page 를 설정해준다.

---

- `sessions` 메서드에서 넘겨줄 `UserSession` 이 필요하다

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserSession {
    private String username;
    private List<SessionInfo> sessions;
    // 세션의 사이즈를 return 해줄 getCount 메서드
    public int getCount() {
        return sessions.size();
    }
}
```

- 여러개의 세션 정보를 담을 sessions 를 만드는데 SessionInfo 객체가 필요하다.
- SessionInfo 에는 세션의 아이디와 마지막으로 세션이 들어온 시간을 만든다.

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class SessionInfo {
    private String sessionId;
    private Date time;
}
```

