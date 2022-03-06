---
layout: post
title: "[Spring Security] 스프링 시큐리티의 권한"
description: spring security
summary: spring security
tags: spring_security
---

# 스프링 시큐리티의 권한

## AOP : Aspect Oriented Programming

- 스프링의 철학 중에 DI(의존성 주입)이 있다. DI 를 통해 약한 상관성(Lossely coupled)을 설계할 수 있고 모듈화를 완성할 수 있다.
- DI 와 함께 매우 중요한 철학이 '관심사의 분리' 이다. 코드를 스파게티로 묶지말고 관심이 있는 코드들을 따로 관리하는 것.
- 관심사 분리가 가능한 이유는 AOP의 기술이 있기에 가능하다.

## AOP 란

- Aspect: 공통의 관심사
  - 권한처리
  - 로그
  - 트랜잭션 관리
  - 세션 관리
  - 기타...

- Weaving
  - bean 과 bean 을 Proxy 로 감싸서 연결해주는 작업
  - bean 과 bean 의 호출사이에 PointCut 을 적용해서 JoinPont 를 판별한 다음 PointCut 을 요청한 advice 를 JoinPoint 에 적용해 준다.

- PointCut : JoinPoint 를 만들어내는 칼
  - JoinPoint 를 지적해주는 expression
  - annotation 마킹

- JoinPoint: 관절. bean과 bean 사이에서 호출이 일어나는 곳
  - method 호출
  - setter 를 통한 멤버 주입
  - 생성자 호출

- Advice : 호출 시 추가로 처리해줄 로직
  - intercept 해서 주입해줄 code

## Authorization (권한)

![authauth2](https://bluewind8791.github.io/assets/image/9-authentication-authorization-2.png)


- SecurityFilterChain 당 한개의 FilterSecurityInterceptor 를 둘 수 있고, 각 SecurityIntercepter 당 한개의 AccessDecisionManager 를 둘 수 있다. 

- 반면 Method 권한 판정은 global 한 권한 위원회를 둔다. 그래서 GlobalMethodSecurityConfiguration 를 통해 AccessDecisionManager 를 설정한다.

- 인증이 모든 요청에 대해 공통적으로 처리해야 하는 것인데 반해, 권한은 상황에 맞게 처리해야 하는 특징이 있다. 그래서 인증을 처리하는 코드는 필터와 어울리고, 권한은 interceptor 와 어울려 동작한다. 
  - 필터는 servlet container 가 제공하는 구조를 스프링이 자체 filter chain 을 만들어서 관리하는 방식으로 처리하고 있고, 
  - interceptor 는 스프링이 bean 을 등록하고 proxy 객체를 가지고 엮어주는 과정에서 각 PointCut 에 의해 구분된 JoinPoint 에 interceptor 가 advice 하는 메커니즘으로 작동한다.

- 필터 위에 상주하는 interceptor 를 FilterSecurityInterceptor 라 하고, method 위에 annotation 의 형태로 상주하는 interceptor 를 MethodSecurityInterceptor 라고 한다.
  - FilterInterceptor 는 필터 설정에서 설정하고 MethodInterceptor 는 annotation 으로 설정한다.
  - `@EnableGlobalMethodSecurity` 를 설정해줘야 MethodSecurityInterceptor 가 동작한다.

## 권한 처리에 관여하는 것들

그런데 권한을 체크하려면 다음과 같은 고민을 해봐야한다.

- 접근하려고 하는 사람이 어떤 접근 권한을 가지고 있는가?
  - GrantedAuthority
    - Role Based
    - Scope Based
    - User Defined

- 접근하려고 하는 상황에서 체크해야 할 내용은 무엇인가?
  - SecurityMetadataSource, ConfigAuttribute
  - 정적인 경우와 동적인 경우
  - AccessDecisionVoter 가 vote 해준다.

- 여러가지 판단 결과가 나왔을 때, 취합은 어떤 방식으로 할 것인가?
  - AccessDecisionManager : 권한 의원회
    - AffirmativeBased : 긍정 위원회
    - ConsensusBased : 다수결 의원회
    - UnanimouseBased : 만장일치 의원회

## 인증과 권한의 구조

![authorization](https://bluewind8791.github.io/assets/image/9-authorization-2.png)

- 인증이 AuthenticationFilter 를 가지고 Authentication 을 발급해주는 관계였다면, 권한은 SecurityInterceptor 에서 Access Granted 와 Denied 를 판정하는 결과를 만들어내는 대응 관계를 가지고 있다.
- 인증이 제공해주는 권한과 각 Interceptor 가 위치한 포인트의 조건들(ConfigAttribute)을 가지고 판정을 내려주는 voter 들에 따라 granted / denied 가 구분된다. 그렇지만 권한은 인증보다 훨씬 상황이 다양하다.
- AccessDecisionManager 는 인터페이스이다. 반드시 voter 를 구현하여 처리해야할 필요는 없다. application 을 구현한다면 voter 없이 구현하는것이 간단 할 수 있다.

## 권한 처리 클래스

![fsi](https://bluewind8791.github.io/assets/image/9-filtersecurityinterceptor.png)

---

## 참고자료

- 옥탑방개발자님의 강의자료 - [GitLab](https://gitlab.com/jongwons.choi/spring-boot-security-lecture/-/blob/master/part2/Lec-1%20%EA%B6%8C%ED%95%9C%EC%9D%98%20%ED%81%B0%20%EA%B7%B8%EB%A6%BC.md)