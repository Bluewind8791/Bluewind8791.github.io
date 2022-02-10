---
layout: post
title: "[Java] IntStream"
description: java intstream
summary: java intstream
tags: java
---

## 1. 생성하기

```java
IntStream.of(1,2,3,4,5); // 1
IntStream.range(1,5); // 2
IntStream.rangeClosed(1,5); // 3
```

1. 1,2,3,4,5 에 대한 int stream 생성, of 의 인자는 배열도 가능
2. 1,2,3,4 에 대한 int stream 생성
3. 1,2,3,4,5에 대한 int stream 생성

## itorator

```java
IntStream.iterator(0, i -> i + 2).limit(4);
```

0,2,4,6 에 대한 스트림 생성, `limit(4)`는 4개의 의미.


## anyMatch

```java
IntStream.range(1,5).anyMatch(i -> i % 2 == 0);
```

`IntStream.range(1,5)` -> 1,2,3,4 중 조건에 맞는것이 하나라도 있다면 true를 return.

## allMatch

```java
IntStream.range(1,5).allMatch(i -> i % 2 == 0);
```

`IntStream.range(1,5)` -> 1,2,3,4 중 조건에 모두 맞다면 true를 return.

## noneMatch

```java
IntStream.range(1,5).noneMatch(i -> i % 2 == 0);
```

`IntStream.range(1,5)` -> 1,2,3,4 중 조건에 모두 맞지 않다면 true를 return.

## filter

```java
IntStream.range(1,5).filter(i -> i % 2 == 1); // 1
IntStream.range(1,5).filter(i -> i % 2 == 1).allMatch(i -> i % 2 == 1); // 2
IntStream.range(1,5).filter(i -> i % 2 == 1).noneMatch(i -> i % 2 == 1); // 3
```

1. `IntStream.range(1,5)` -> 1,2,3,4 중 `(i -> i % 2 == 1)` 조건에 맞는 1, 3 을 return
2. 위의 1에서 `(i -> i % 2 == 1)` 조건에 모두 맞다면 true 를 return
3. 위의 1에서 `(i -> i % 2 == 1)` 조건에 하나도 맞지않다면 true 를 return

## max, min

```java
IntStream.range(1,5).max().getAsInt(); // 1
IntStream.range(1,5).min().getAsInt(); // 2
```

1. `IntStream.range(1,5)` -> 1,2,3,4 중 가장 큰 수를 int로 return -> 4
2. `IntStream.range(1,5)` -> 1,2,3,4 중 가장 작은 수를 int로 return -> 1

## 참고

- 오라클자바님의 [개발자를 위하여 블로그](https://blog.daum.net/oraclejava/15874666)
