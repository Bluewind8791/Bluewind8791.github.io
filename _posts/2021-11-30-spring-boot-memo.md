---
layout: post
title: "[Spring Boot] Validation"
description: spring boot validation filter interceptor
summary: spring boot validation filter interceptor
tags: spring
---

## Validation

- Validation 이란 프로그래밍에 있어서 가장 필요한 부분.
- 특히 Java에서는 null 값에 대해서 접근하려고 할때, null pointer exception이 발생함으로, 이러한 부분을 방지하기 위하여 미리 검증하는 과정을 Validation이라고 함.

ex) 단순한 예시 코드

```java
public void run(String account, String pw, int age) {
    if (account == null || pw == null) {
        return;
    }

    if (age == 0) {
        return
    }
    // logics
}
```

1. 검증해야할 값이 많은경우 코드의 길이가 길어진다
2. 구현에 따라서 달라질 수 있지만 service logic과의 분리가 필요
3. 흩어져있는 경우 어디에서 검증하는지 알기 어려우며, 재사용의 한계가 있음
4. 구현에 따라 달라질 수 있지만, 검증 logic이 변경되는 경우 테스트코드 등 참조하는 클래스에서 logic이 변경되어야 하는 부분이 발생할 수 있음.

## Annotation

- @Size     : 문자 길이 측정, int type 불가
- @NotNull  : null 불가
- @NotEmpty : null, "" 불가
- @NotBlank : null, "", " " 불가
- @Past     : 과거 날짜
- @PastOrPresent
- @Future
- @FutureOrPresent
- @Pattern : 정규식 적용
- @Max
- @Min
- @AssertTrue / False : 별도 Logic 적용
- @Valid              : 해당 object validation 실행

## Custom Validation

1. AssertTrue / False 와 같은 method 지정을 통하여 Custom Logic 적용 가능
2. ConstraintValiator를 적용하여 재사용이 가능한 Custom Logic 적용 가능

## Spring Boot Exception

### Exception 처리

Web Application의 입장에서 바라보았을때, 에러가 났을때 내려줄 수 있는 방법은 많지 않음.

1. 에러페이지
2. 4XX error or 5XX error
3. Client가 200외에 처리를 하지 못할때는 200을 내려주고 별도의 에러 메세지 전달

- @ControllerAdvice : Global 예외처리 및 특정 package/controller 처리
- @ExceptionHandler : 특정 controller의 예외처리

## Filter

Filter란, Web Application에서 관리되는 영역으로써, Spring Boot Framework에서 클라이언트로부터 오는 요청/응답에 대해서 최초/최종 단계 위치에 존재하며, 이를 통하여 요청/응답의 정보를 변경하거나, Spring에 의해서 데이터가 변환되기 전의 순수한 Client의 요청/응답 값을 확인할 수 있다.

**유일하게 SerletRequest, ServletResponse의 객체를 변환**할수 있다.

- 주로 Spring Framework에서는 request / response의 Logging 용도로 활용하거나 인증과 관련된 Logic들을 해당 필터에서 처리.

- 이를 선/후 처리함으로써 Service business logic과 분리시킨다.

## Interceptor

- Interceptor란 Filter와 매우 유사한 형태로 존재하지만, 차이점은 Spring Context에 등록됨.

- AOP와 유사한 기능을 제공할수있으며, 주로 **인증 단계**를 처리하거나, Logging을 하는데에 사용.

- 이를 선/후 처리함으로써, Service business logic과 분리시킨다.
