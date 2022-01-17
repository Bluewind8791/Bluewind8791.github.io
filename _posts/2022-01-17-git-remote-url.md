---
layout: post
title: "[Git] Remote URL 변경하기"
description: git
summary: git
tags: git
---

GitHub 사이트에서 원격 URL 가 변경되었을 때, 로컬 저장소에서 바라보는 원격 URL 가 달라서 오류가 일어나는 경우가 있다.

로컬에서 remote url 을 확인하는 방법은 아래의 방법으로 확인 가능하다.

```bash
git remote -v
```

이를 변경하고 싶을 때는 아래의 명령어로 변경할 수 있다.

```bash
git remote set-url origin https://github.com/Bluewind8791/learning_programming.git
```

## 참조

- [DevAndy 님의 블로그](https://youngjinmo.github.io/2019/09/git-change-remote-branch-url/)