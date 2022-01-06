---
layout: post
title: "[Spring Boot]"
description: spring boot ioc di aop
summary: spring boot ioc di aop
tags: spring
---

## IoC (Inversion of Control)

스프링에서는 일반적인 자바 객체를 new로 생성하여 개발자가 관리하는것이 아닌 Spring Container 에 모두 맡긴다.

즉, 개발자에서 -> 프레임워크로 제어의 객체 관리의 '권한이 넘어갔음'으로 "제어의 역전(IoC)"이라 한다.

## DI (Dependency Injection)

- DI 장점
  - 의존성으로 부터 격리시켜 코드 테스트에 용이하다.
  - DI 를 통하여, 불가능한 상황을 Mock 와 같은 기술을 통하여 안정적으로 테스트 가능
  - 코드를 확장하거나 변경할떄 영향 최소화 (추상화)
  - 순환 참조를 막을수있다.

## AOP (Aspect Oriented Programming : 관점지향 프로그램)

스프링 어플리케이션은 대부분 특별한 경우를 제외하고는 MVC 웹 어플리케이션에서는 Web Layer, Business Layer, Data Layer로 정의.

- Web Layer : REST API 제공, Client 중심의 로직 적용
- Business Layer : 내부 정책에 따른 logic 개발, 해당 부분을 개발
- Data Layer : 데이터베이스 및 외부와의 연동 처리

- Annotation
  - @Aspect        : 자바에서 널리 사용하는 AOP 프레임워크에 포함 / AOP를 정의하는 Class에 할당
  - @Pointcut      : 기능을 어디에 적용시킬지, 메소드? Annotation? 등 AOP를 적용시킬 지점 설정
  - @Before        : 메소드 실행하기 이전
  - @After         : 메소드 성공적으로 실행 후, exception이 발생하더라도 실행
  - @AfterReturing : 메소드 호출 성공 실행시 (Not Throws)
  - @AfterThrowing : 메소드 호출 실패 예외 발생 (Throws)
  - @Around        : Before / After 모두 제어

## Spring Boot Annotations

- @SpringBootApplication : Spring boot application 으로 설정
- @Controller            : View를 제공하는 controller로 설정
- @RestController        : REST API 를 제공하는 controller로 설정

- @RequestMapping : URL 주소를 mapping
- @GetMapping     : http GetMethod URL 주소 mapping
- @PostMapping
- @PutMapping
- @DeleteMapping

- @RequestParam : URL Query Parameter mapping
- @RequestBody  : http body를 Parsing mapping
- @Valid        : POJO Java class의 검증

- @Configuration : 1개 이상의 bean을 등록할때 설정
- @Component     : 1개의 class 단위의 bean을 등록
- @Bean          : 1개의 외부 라이브러리로부터 생성한 객체를 등록시 사용
- @Autowired     : DI를 위한곳에 사용
- @Qualifier     : @Autowired 사용시 bean이 2개 이상일때 명시적 사용
- @Resource      : @Autowired + @Qualifier 의 개념

- @Aspect        : AOP 적용시 사용
- @Before        : AOP 메소드 이전 호출 지정
- @After         : AOP 메소드 호출 이후 지정, 예외 발생 포함
- @Around        : AOP 이전/이후 모두 포함, 예외 발생 포함
- @AfterReturing : AOP 메소드의 호출이 정상일때 실행
- @AfterThrowing : AOP시 해당 메소드가 예외 발생할때 지정
