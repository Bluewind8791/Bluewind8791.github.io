---
layout: post
title: "[JPA] JPA 소개와 간단한 명령어"
description: spring jpa
summary: spring jpa
tags: jpa spring
---

# JPA 소개


현대의 웹 어플리케이션에서 관계형 데이터베이스는 빠질 수 없는 요소가 되었습니다. 그러다 보니 객체를 관계형 데이터베이스에서 관리하는것이 중요하게 되었습니다.

그에 따라서 여러 문제가 생겼는데,
그 중 하나는 코드가 SQL 중심이 되어가면서 프로젝트 대부분이 어플리케이션 코드보다 반복적인 SQL로 가득하게 되었습니다.
또 다른 문제중 하나는 **패러다임 불일치** 문제입니다.
관계형 데이터베이스는 어떻게 데이터를 저장할지에 초점이 맞춰진 기술입니다. 하지만 자바를 비롯한 객체지향 프로그래밍 언어는 메세지를 기반으로 기능과 속성을 한 곳에서 관리하는 기술입니다. 이렇게 서로 패러다임이 다르다 보니 여러 문제가 발생하게 되는데 이를 패러다임 불일치라 합니다.

그래서 서로 지향하는 바가 다른 두개의 영역 중간에서 패러다임 일치를 시켜주기 위하여 JPA가 개발되었습니다.

## Spring Data JPA

JPA는 인터페이스로서, 사용하기 위해서는 구현체가 필요합니다. 대표적으로 Hibernate, Eclipse, Link 등이 있습니다. 하지만 Spring에서 JPA를 사용할 때는 이 구현체들을 직접 다루지 않고, 대신 이 구현체들을 쉽게 사용하기 위하여 추상화시킨 Spring Data JPA 라는 모듈을 사용합니다.

Spring Data JPA를 쓰면 어떤 용이성이 있을까요?

- 구현체 교체의 용이성 : Hibernate 외에 다른 구현체로 쉽게 교체하기 위함입니다.
  - 즉 Hibernate가 언젠가 수명을 다하여 새로운 JPA 구현체가 대세로 떠오를 때, Spring Data JPA를 사용하는 중이라면 아주 쉽게 교체할 수 있습니다.
- 저장소 교체의 용이성 : 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함입니다.
  - 만약 데이터베이스를 기존의 관계형 데이터베이스에서 Mongo DB로 교체해야 하는 시기가 온다면 개발자는 Spring Data JPA에서 Spring Data MongoDB로 의존성만 교체하면 됩니다.



- 객체와 데이터베이스 사이의 관계를 연결해주는 것을 **ORM**(Object Relational Mapping)이라고 합니다.
- Java 진영에서는 ORM의 표준 스펙으로 **JPA**(Java Persistence Api)를 인터페이스로 정의하여 제공합니다.
- JPA의 실제 클래스들을 모아놓은 것이 **Hibernate**인데, 그 중에서 자주쓰이는 기능들을 더 사용하기 쉽도록 Spring Framework에서 제공하는것이 **Spring Data JPA** 입니다.

- Hibernate란 JPA에 대한 실제 구현체, 흔히 말하는 Implemetation 입니다.
- Spring Data Jpa 는 Spring 에서 Hibernate를 조금 간편하게 사용할 수 있도록 추상객체를 한번 더 감싸서 만들어 놓은것입니다.

1
3
5
Financial Report