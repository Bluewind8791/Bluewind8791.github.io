---
layout: post
title: "[Spring Security] MethodSecurityInterceptor"
description: spring security interceptor
summary: spring security interceptor
tags: spring_security
---

# MethodSecurityInterceptor

- 필터를 통과한 request는 보안이 마킹된 메소드를 실행할 때 마다 MethodSecurityInterceptor 의 검사를 받는다. 
이를 설정하는 곳이 GlobalMethodSecurityConfiguration 이다.
  - 여기에 `securedEnabled=true`를 설정하면 `@Secured`로 설정된 권한을 체크하고, 
  - `prePostEnabled=true`로 설정하면 `@PreAuthorize`, `@PreFilter`, `@PostAuthorize`, `@PostFilter`로 설정된 권한을 체크한다.

MethodSecurityInterceptor 에서 중요한 멤버는 아래의 3가지이다

- AccessDecisionManager : `@Secured` 나 `@PreAuthorize`, `@PreFilter` 처리
- AfterInvocationManager : `@PostAutorize`, `@PostFilter` 처리
- RunAsManager : 임시 권한 부여

![adm](https://bluewind8791.github.io/assets/image/9-access-decision-manager.png)

## AfterInvocationManager

- Authentication 통행증만 가지고는 권한 체크를 충분히 했다고 볼 수가 없다. 보통, 어떤 객체의 값을 변경해야 하는 경우에는 메소드에 들어오기 전에, 값을 조회하려고 하는 경우에는 값을 가져온 이후에 각각 접근 권한을 체크해줘야 한다.

- 체크해야 할 대상이 한개라면 Pre/PostAuthorized 로 체크를 하면 되지만, 대상이 복수개라면 보통은 리스트로 묶이기 때문에 대상을 filtering 해서 들어가거나 넘겨야 한다.

- 물론, 메소드를 처리하는 중간에 권한을 검사해야하는 경우도 있다. 이 경우는 proxy bean 의 특징을 잘 파악해서 메서드간에 권한 검사가 충분히 이루어지도록 annotation 을 설계해서 처리할 수 있다.

### AfterInvocationProviderManager

```java
public Object decide(
  Authentication authentication,
  Object object,
  Collection<ConfigAttribute> config,
  Object returnedObject
) throws AccessDeniedException {

	Object result = returnedObject;
	for (AfterInvocationProvider provider : this.providers) {
		result = provider.decide(authentication, object, config, result);
	}
	return result;
}
```

### PostInvocationAdviceProvider

```java
public Object decide(
  Authentication authentication,
  Object object,
  Collection<ConfigAttribute> config,
  Object returnedObject
) throws AccessDeniedException {

	PostInvocationAttribute postInvocationAttribute = findPostInvocationAttribute(config);
	if (postInvocationAttribute == null) {
		return returnedObject;
	}
	return this.postAdvice.after(authentication, (MethodInvocation) object, postInvocationAttribute,
				returnedObject);
}
```

### ExpressionBasedPostInvocationAdvice

- GlobalMethodSecurityConfiguration 에서 expressHandler 를 ExpressionBasedPreInvocationAdvice 와 함께 공유한다.
- 이 말은 `@PreAuthorize` `@PreFilter` 와 같은 SpEL 루트 객체를 가지고 작업한다는 말이 된다.

```java
public Object after(
  Authentication authentication,
  MethodInvocation mi,
  PostInvocationAttribute postAttr,
  Object returnedObject
) throws AccessDeniedException {

	PostInvocationExpressionAttribute pia = (PostInvocationExpressionAttribute) postAttr;
	EvaluationContext ctx = this.expressionHandler.createEvaluationContext(authentication, mi);
	Expression postFilter = pia.getFilterExpression();
	Expression postAuthorize = pia.getAuthorizeExpression();
	if (postFilter != null) {
		if (returnedObject != null) {
			returnedObject = this.expressionHandler.filter(returnedObject, postFilter, ctx);
		}
	}
	this.expressionHandler.setReturnObject(returnedObject, ctx);
	if (postAuthorize != null && !ExpressionUtils.evaluateAsBoolean(postAuthorize, ctx)) {
		throw new AccessDeniedException("Access is denied");
	}
	return returnedObject;
}
```

## 출처

- 옥탑방개발자님의 강의자료 : [GitLab](https://gitlab.com/jongwons.choi/spring-boot-security-lecture/-/blob/master/part2/Lec-3%20%EB%A9%94%EC%8F%98%EB%93%9C%20%ED%9B%84%EC%B2%98%EB%A6%AC.md)