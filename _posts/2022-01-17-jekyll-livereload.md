---
layout: post
title: "Jekyll LiveReload 기능 적용하기"
description: jekyll livereload
summary: jekyll livereload
tags: jekyll
---

# Jekyll LiveReload

Jekyll 테마를 수정할 때, 변경점이 생길 때 마다 브라우저에서 새로고침을 해줘야하지만, 이 기능을 사용하면 변경점이 발생할 때 마다 알아서 새로고침을 해준다!

## 적용법

### 1. Browser

- 브라우저에서 LiveReload 확장 프로그램을 설치한다.
- [Chrome 확장프로그램](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei)

### 2. Gemfile

- Gemfile 에서 아래의 코드를 추가해준다.

```
gem 'eventmachine', '1.2.7', git: 'https://github.com/eventmachine/eventmachine', tag: 'v1.2.7'
```

### 3. Terminal

- terminal 에서 아래와 같이 install 한 뒤
- 서버를 실행할 때 마다 끝에 --livereload 를 붙여서 실행한다.

```bash
$ bundle install
$ bundle exec jekyll serve --livereload
```

## 참고

- [chanoyoung 님의 블로그](https://chanoyoung.github.io/jekyll-livereload/)