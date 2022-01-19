---
layout: post
title: "[Spring Security] 권한 체크와 오류 처리"
description: spring security athority authentication
summary: spring security athority authentication
tags: spring_security
---

# 권한 체크와 오류 처리

- 스프링의 핵심은 bean 을 담고있는 container 라고 할 수 있다.
- 컨테이너 안에 bean 들을 담은 이유는 컨테이너가 처리 로직에 직접 관여할 수 있도록 하기 위함이다.
- 개발자는 비즈니스 코드와 처리 방식을 컨테이너에게 알려주고 나머지는 컨테이너가 다 알아서 해준다.

- 스프링은 코드가 스파게티가 되지 않도록 *관심사 분리 원칙*을 강조한다.
- AOP(Aspect Oriented Programming)의 철학이 그대로 여기저기에 녹아들어 있다.

---

## 스프링의 인증과 권한

![authAuth](https://bluewind8791.github.io/assets/image/9-authentication-authorization.png)

---

## **FilterSecurityInterceptor**

![FilterSecurityInterceptor](https://bluewind8791.github.io/assets/image/9-filtersecurityinterceptor.png)

- SecurityInterceptor :
  - `AccessDecisionManager` 를 통하여 권한 여부를 판단하고 통과시켜주거나 Deny 한다.

- 주로 처리 로직이 아래와 같다

```java
public void invoke(FilterInvocation filterInvocation) throws IOException, ServletException {
	if (isApplied(filterInvocation) && this.observeOncePerRequest) {
		// filter already applied to this request and user wants us to observe
		// once-per-request handling, so don't re-do security checking
		filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
		return;
	}
	// first time this request being called, so perform security checking
	if (filterInvocation.getRequest() != null && this.observeOncePerRequest) {
		filterInvocation.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
	}
	InterceptorStatusToken token = super.beforeInvocation(filterInvocation); // (1)
	try {
		filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
	}
	finally {
		super.finallyInvocation(token); // (2)
	}
	super.afterInvocation(token, null); // (3)
}
```

  - (1) beforeInvodation : Security Config 에서 설정한 접근 제한을 체크한다.
  - (2) finallyInvocation : RunAs(임시 권한을 부여받는 로직) 권한을 제거한다.
  - (3) afterInvocation : `AfterInvocationManager` 를 통해 체크가 필요한 사항을 체크한다. 특별히 설정하지 않으면 `AfterInvocationManager` 는 null 이다.

---

## **ExceptionTranslationFilter**

이 필터는 FilterSecurityInterceptor 나 app 에서 올라오는 오류를 가로채 처리하는 작업을 한다.

- `AuthenticationException` 과 `AccessDeniedException` 만 처리한다.
- 그 밖의 오류는 보통 `ControllerAdvice` 를 이용하여 처리하는것을 권장한다.

![ExceptionTranslationFilter](https://bluewind8791.github.io/assets/image/9-exception-translation-filter.png)

---

## **401 에러와 403 에러**

- 401: 인증 실패
  - AuthenticationException
  - 다시 로그인 해야 하므로 AuthenticationEntryPoint 로 처리를 넘긴다.
  - AuthenticationException이 발생했다고 서버가 반드시 401 에러를 내려보내는 것은 아니다.
  - 해당 에러를 401 오류로 처리하는 코드를 넣어야 한다. 필요에 따라서는 403 오류코드로 처리하기도 한다.

- 403: 권한 없음
  - AccessDeniedException
  - Anonymous 유저이거나 RememberMe 유저이면 다시 로그인 하도록 AuthenticationEntryPoint 로 처리를 넘긴다.
  - 그 밖의 유저는 권한없음 페이지로 이동하거나 권한없음 메시지를 받는다.

---

## **AuthenticationEntryPoint**

```java
void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException;
```

- AuthenticationEntryPoint 는 인증과정에서 실패하거나 Authentication 헤더를 보내지 않게 되는 경우 일어나는 401 에러를 처리해 주는 로직이다.
- 웹페이지라면 로그인 페이지로 이동하고, 클라이언트 애플리케이션이라면 재로그인하라는 메시지를 보낸다.

### 실습

*(본 코드는 [GitHub](https://github.com/Bluewind8791/learning_programming/tree/main/Learning-Spring/part3_spring-security/8-session-management) 페이지에서 확인할 수 있습니다.)*

```java
http
    ...
    .exceptionHandling(error->
        error
            .authenticationEntryPoint(authenticationEntryPoint)
    ...
```

- 로그인이 필요하다는 에러 창을 간단하게 만들고 (LoginRequired.html)
- HomeController 에 로그인이 필요한 경우에 /login-required 로 이동하는 메서드를 만든다.

```java
@GetMapping("/login-required")
public String loginRequired(){
    return "loginRequired";
}
```

그리고 config 폴더에 AuthenticationEntryPoint 를 implement 한 CustomEntryPoint 를 구현한다.

```java
public class CustomEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(
            HttpServletRequest request, 
            HttpServletResponse response,
            AuthenticationException authException) throws IOException, ServletException {
        
        request
            .getRequestDispatcher("/login-required")
            .forward(request, response);
    }
}
```

그렇게 구현한 CustomEntryPoint 클래스를 authenticationEntryPoint 에 넣어 설정한다.

```java
http
    ...
    .exceptionHandling(error->
        error
            .authenticationEntryPoint(new CustomEntryPoint())
    ...
```

---

## AccessDeniedHandler

- 만약 사용자가 access 에 실패했을 때, 어떻게 처리할 것인지 설정을 할 수 있다.

```java
http
    .exceptionHandling(error->
        error
            // .accessDeniedPage("/access-denied")
            .accessDeniedHandler("")
```

- 그런데 만약 기존에 설정해 놓았던 `.accessDeniedPage("/access-denied")`에 `accessDeniedHandler` 를 추가해서 함께 쓴다면 두번의 redirecting 이 일어나기 때문에 문제가 될 수 있다.
- 그래서 `accessDeniedPage` 는 무시되며, 대신에 `accessDeniedHandler` 를 직접 커스텀하여 사용할 수 있다.

```java
public class CustomDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(
            HttpServletRequest request, 
            HttpServletResponse response,
            AccessDeniedException accessDeniedException) throws IOException, ServletException 
    {
        // do something...
    }
}
```

- config 에 custom 할 Handler 를 생성하는데 `AccessDeniedHandler` 를 implement 한다.
- 그러면 `handle` 을 구현하는데 request, response, accessDeniedException 를 가져다 준다.
- 이 메서드로 엑세스 거부 상황에 따라서 다른 오류 창을 보여지게 할 수 있다.
- 어떤 근거로 다른 엑세스 오류창을 보여줄 것인가를 설정하기 위해서는 `AccessDeniedException` 을 상속한 Handler를 보낼 수 있다.

- 테스트로 userPage 에 들어오면 `YouCannotAccessUserPage` 라는 exception 을 던지도록 해보자.

```java
@PreAuthorize("hasAnyAuthority('ROLE_USER')")
@GetMapping("/user-page")
public String userPage() throws YouCannotAccessUserPage{
    if (true) {
        throw new YouCannotAccessUserPage();
    }
    return "UserPage";
}
```

```java
public class YouCannotAccessUserPage extends AccessDeniedException {
    public YouCannotAccessUserPage() {
        super("유저페이지 접근 거부");
    }
}
```

- `YouCannotAccessUserPage` 는 `AccessDeniedException` 를 상속받으며
- "유저페이지 접근 거부" 라는 메세지를 던진다.


다시 config의 CustomDeniedHandler 로 돌아와서,

```java
public class CustomDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(
            HttpServletRequest request, 
            HttpServletResponse response,
            AccessDeniedException accessDeniedException) throws IOException, ServletException 
    {
        if (accessDeniedException instanceof YouCannotAccessUserPage) {
            request.getRequestDispatcher("/access-denied").forward(request, response);
        } else {
            request.getRequestDispatcher("/access-denied2").forward(request, response);
        }
    }
}
```

만약에 accessDeniedException 이 YouCannotAccessUserPage 클래스 라면 "/access-denied" 로 보내주고, 아니라면 "/access-denied2" 로 보내도록 설정한다.

```java
protected void configure(HttpSecurity http) throws Exception {
    http
        ...
        .exceptionHandling(error->
            error
                .accessDeniedHandler(new CustomDeniedHandler())
        ...
```

이런식으로 에러페이지라고 하더라도 상황에 따라서 다른 에러페이지를 사용자에게 보여주려 한다면 이렇게 AccessDeniedHandler 를 핸들링 해야 할 수 있다.

---

## **AuthenticationException**

- InsufficientAuthenticationException : credential 값이 부족할 때

- AccountStatusException :
  - LockedException
  - CredentialsExpiredException
  - DisabledException
  - LockedException

- AuthenticationCredentialsNotFoundException : 서블릿 컨텍스트에서 Authentication이 없을 때

- AuthenticationServiceException : 서버 문제로 인증 서비스를 처리해 줄 수 없을 때. (500)

- BadCredentialsException : credentials 값이 invalid 할 때

- RememberMeAuthenticationException
  - CookieTheftException
  - InvalidCookieException

- InsufficientAuthenticationException

- AuthenticationServiceException
  - InternalAuthenticationServiceException

- NonceExpiredException

- ProviderNotFoundException

- PreAuthenticatedCredentialsNotFoundException

- SessionAuthenticationException

- UsernameNotFoundException
