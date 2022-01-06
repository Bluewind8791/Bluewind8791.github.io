---
layout: post
title: Design Pattern
description: spring design pattern
summary: spring design pattern
tags: spring
---

## 생성 패턴

- 객체를 생성하는 것과 관련된 패턴
- 객체의 생성고 변경이 전체 시스템에 미치는 영향을 최소화
- 코드의 유연성 증가
  - Singleton
  - Builder
  - Chaining

## Singleton Pattern

- 클래스가 유일하게 1개만 존재할때 서로 자원을 공유할때 사용하는데 실물 세계에서는 프린터가 해당.

## 구조 패턴

큰 규모의 시스템에서는 많은 클래스들이 서로 의존성을 가지게되는데, 이런 복잡한 구조를 개발하기 쉽게 만들어주고, 유지 보수를 하기 쉽게 함.

- Adapter
- Decorator
- Facade
- Proxy

---

### Adapter pattern

- Adapter 는 실생활에서 100v 를 220v로 변환해주는 변환기를 예로 들 수 있음.
- 호환성이 없는 기존 클래스의 인터페이스를 변환하여 재사용할 수 있게함.
- SOLID 중에서 개방폐쇄원칙(OCP) 를 따름.

### Proxy Pattern

- 대리인이라는 뜻.
- 뭔가를 대신하여 처리.
- Proxy Class 를 통하여 대신 전달하는 형태로 설계. 실제 Client는 Proxy로 부터 결과를 받음.
- Cache의 기능으로도 활용 가능.
- SOLID 중에서 개방폐쇄원칙(OCP) 와 의존역전원칙(DIP)를 따름.

### Decorator Pattern

- 기존 뼈대(클래스)는 유지하되, 이후 필요한 형태로 꾸밀때 사용.
- 확장이 필요한 경우 상속의 대안으로도 활용.
- SOLID 중에서 OCP와 DIP를 따름.

### Facade Pattern

- Facade는 건물의 앞쪽이라는 뜻.
- 여러개의 객체와 실제 사용하는 서브 객체의 사이에 복잡 의존관계가 있을 때, 중간에 facade라는 객체를 두고, 여기에서 제공하는 interface 만을 활용하여 기능 사용하는 방식.
- Facade는 자신이 가지고있는 각 클래스의 기능을 명확히 알아야한다.

---

## 행위 패턴

- 상호작용하는 방법과 책임을 분산하는 방법.
- 행위 관련 패턴을 사용하여 독립적으로 일을 처리하고자 할때
  - Observer
  - Strategy

### Observer Pattern

- 관찰자 패턴은 변화가 일어났을때 미리 등록된 다른 클래스에 통보해주는 패턴을 구현.
- event listener에서 해당 패턴 사용

### Strategy Pattern

- 객체지향의 꽃
- 유사한 행위를 캡슐화하여 객체의 행위를 바꾸고 싶은 경우 직접 변경하는것이 아닌 전략만 변경하여 유연하게 확장하는 패턴.
- SOLID 중에서 개방폐쇄원칙과 의존역전원칙을 따른다.
