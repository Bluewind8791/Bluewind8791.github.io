---
layout: post
title: "[Spring Security] Foam Login"
description: spring security foam login
summary: spring security foam login
tags: spring_security
---

## **DefaultLoginPageGeneratingFilter**

- GET /login 을 처리
- 별도의 로그인 페이지 설정을 하지 않으면 제공되는 default 필터
- 기본 로그인 폼을 제공
- OAuth2 / OpenID / Saml2 로그인과도 같이 사용 가능

## **UsernamePasswordAuthenticationFilter**

- POST /login 을 처리. processingUrl을 변경하면 주소를 바꿀 수 있다.
- form 인증을 처리해주는 필터로 스프링 시큐리티에서 가장 일반적임.
- 주요 설정 정보
  - filterProcessingUrl : 로그인을 처리해줄 URL (POST)
  - username parameter : POST에 username에 대한 값을 넘겨줄 인자의 이름
  - password parameter : POST에 password에 대한 값을 넘겨울 인자의 이름
  - 로그인 성공시 처리 방법
    - defaultSuccessUrl : alwaysUse 옵션 설정이 중요
    - successHandler
  - 로그인 실패시 처리 방법
    - failureUrl
    - failureHandler
  - authenticationDetailSource : Authentication 객체의 details에 들어갈 정보를 직접 만들어줌.

```java
@Override
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
        throws AuthenticationException {
    if (this.postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    }
    String username = obtainUsername(request);
    username = (username != null) ? username : "";
    username = username.trim();

    String password = obtainPassword(request);
    password = (password != null) ? password : "";

    UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password); // token을 먼저 만들고
    // Allow subclasses to set the "details" property
    setDetails(request, authRequest); // detail 정보가 필요할수있기때문에 set details를 하고
    return this.getAuthenticationManager().authenticate(authRequest); // auth manager에게 처리해달라고 위임함
}
```

## **DefaultLogoutPageGeneratingFilter**

- GET /logout 을 처리.
- POST /logout 을 요청할수있는 UI를 제공
- csrf token이 처리됨

## **LogoutFilter**

- POST /logout 을 처리한다. processingUrl을 번경하면 바꿀 수 있다.
- 로그아웃을 처리한다.

  - session, SecurityContext, csrf, cookie, remember-me cookie 등을 삭제한다.
  - 기본 로그인 페이지로 redirect 한다.

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    if (requiresLogout(request, response)) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication(); // SecurityContextHolder가 auth를 가져다가
        if (this.logger.isDebugEnabled()) {
            this.logger.debug(LogMessage.format("Logging out [%s]", auth));
        }
        this.handler.logout(request, response, auth); // 핸들러에게 넘겨서 로그아웃 처리
        this.logoutSuccessHandler.onLogoutSuccess(request, response, auth); // 처리가 되면 success 처리
        return;
    }
    chain.doFilter(request, response);
}
```

- LogoutHandler
  - void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication); 구현하는 유일한 메소드
  - SecurityContextLogoutHandler : 세션과 SecurityContext 를 clear 함.
  - CookieClearingLogoutHandler : clear 대상이 된 쿠키들을 삭제함.
  - CsrfLogoutHandler : csrfTokenRepository 에서 csrf 토큰을 clear 함.
  - HeaderWriterLogoutHandler
  - RememberMeServices : remember-me 쿠키를 삭제함.
  - LogoutSuccessEventPublishingLogoutHandler : 로그아웃이 성공하면 이벤트를 발행함.

- LogoutSuccessHandler
  - void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication)
  - throws IOException, ServletException;
  - SimpleUrlLogoutSuccessHandler
