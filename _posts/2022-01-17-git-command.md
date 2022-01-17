---
layout: post
title: "[Git] Git Command 모음"
description: git command
summary: git command
tags: git
---

*몇번을 해봐도 자꾸 까먹는 Git 명령어. 역시 "기억보단 기록을" 이라는 향로 개발자님의 블로그 명이 떠오른다.*

## git config

- git 을 설치하고 최초로 설정한다.
- `--global` 옵션으로 설정하는것은 처음의 딱 한번만 하면 되며, 프로젝트마다 다른 이름과 이메일 주소를 사용하고싶다면 `--global` 옵션을 빼고 실행한다.

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

- 설정한 값 확인은 `git config --global --list` 명령어로 확인 가능하다.

## 편집기 설정

- git 에서 사용할 텍스트 편집기를 지정할 수 있다.
- 개인적으로 git bash 에서 default 값으로 사용되는 편집기보다는 notepad++ 가 편리하기 때문에 변경해서 사용한다.
  - (notepad++ 설치 경로를 잘 확인하자)

```bash
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -nosession"
```

## config 확인

```
git config --list
```

---

## git repository

- 먼저 터미널에서 cd 명령어를 사용하여 원하는 로컬 디렉토리로 이동한다.
- 두가지의 방법이 있으니 택일 하여 진행한다.

1. 기존 디렉토리를 git 저장소로 등록하기

```
git init
```

- 이렇게하면 로컬 디렉토리에 .git 이라는 숨김폴더가 생성된다.

2. github 의 directory 를 clone 받아오기

```
git clone https://github.com/Bluewind8791/...
```

---

## git stages

git 에서 관리되는 파일의 라이프 사이클에는 아래와 같이 4단계가 있다.

- Untracked : git의 관리대상이 아님
- Unmodified : 말 그대로 파일이 tracked 되었으나, commit 이후 수정되지 않은 파일이다.
- Modified : Unmodified 파일이 수정된 상태이다.
- Staged: modified 된 파일을 git에 올리기 위해서 staged 상태로 만들고 commit 을 진행하게 된다.

![file-lifecycle](https://git-scm.com/book/en/v2/images/lifecycle.png)

## git status

- 아래의 명령어를 사용하게 되면 어떤 파일이 어떤 상태인지를 알려준다. 위의 4단계를 참조하자.

```
git status
```

## git add

- add 명령어로 Untracked 상태인 파일을 Tracked 상태로 변경할 수 있으며,
- Modified 파일을 Staged 상태로 변경할 수 있다.

```bash
git add README
git add *.c
git add .
```

- README 파일만 add
- 확장자가 c 인 파일만 add
- 디렉토리 내의 모든 파일 add

## git commit

- Modified 된 파일을 Staged 상태로 커밋하기 위하여 아래의 명령어를 실행한다.

```bash
git commit
```

---

## git push

- commit 된 프로젝트를 저장소에 업로드하고 싶을 때 push 명령어를 사용한다.
- 이 명령은 git push <리모트 저장소 이름> <브랜치 이름> 의 구성으로 되어 있다.

```
git push origin master
```

---

## git pull / fetch

- 리모트 저장소에서 데이터를 가져오려면 pull 과 fetch 의 방법이 있다.

```
git fetch origin
```

- git fetch {원격 저장소 이름}
- git fetch 명령은 리모트 저장소의 데이터를 모두 로컬로 가져오지만, 자동으로 Merge 하지 않는다. 
- 그래서 직접 로컬에서 하던 작업을 정리하고 나서 수동으로 Merge 해야 한다.

```
git pull
```

- git pull 명령으로 리모트 저장소 브랜치에서 데이터를 가져올 뿐만 아니라 자동으로 로컬 브랜치와 Merge 시킬 수 있다

---

## git remote

- 리모트 저장소는 인터넷이나 네트워크 어딘가에 있는 저장소를 말한다.
- git remote 명령으로 현재 프로젝트에 등록된 리모트 저장소를 확인할 수 있다. 

```
$ git remote
origin
```

- v 옵션을 주어 단축이름과 URL을 함께 볼 수 있다

```
$ git remote -v
origin	https://github.com/Bluewind8791/project (fetch)
origin	https://github.com/Bluewind8791/project (push)
```

- remote 저장소를 아래의 명령어로 추가할 수 있다.
- git remote add {저장소 이름} {url}

```
git remote add pb https://github.com/Bluewind8791/project
```

- 리모트 저장소 이름 바꾸기
- git remote rename {origin 저장소 이름} {rename 저장소 이름}

```
git remote rename pb paul
```

- 리모트 저장소 삭제하기

```
git remote remove paul
```

---

## git branch

branch(나뭇가지) 는 말 그대로 기존의 줄기에서 다른 줄기로 뻗어 나가는것을 의미한다. 예를 들어, 새로운 기능을 추가해서 테스팅 할 때 유용하게 사용할 수 있다. 물론 이 기능이 마음에 들어서  master 브랜치로 병합할 수도 있다.

- 브랜치 생성
- git branch 명령은 브랜치를 만들기만 하고 브랜치를 옮기지 않는다.

```
git branch {branch name}
```

- git checkout 명령으로 다른 브랜치로 이동할 수 있다

```
git checkout {branch name}
```

---

## git merge

- 만일 프로젝트 중에 해결해야할 hotfix 가 생겼다면, hotfix 라는 브랜치를 만들고 이슈를 해결할 때 까지 사용한다.
- 브랜치를 만들면서 Checkout까지 한 번에 하려면 git checkout 명령에 -b 라는 옵션을 추가한다.

```
git checkout -b hotfix
```

- 이슈를 해결하였다면 최종적으로 운영환경에 배포하기 위해 hotfix 브랜치를 master 브랜치에 합쳐야 한다.
- git merge 명령으로 아래와 같이 한다.

```
$ git checkout master
$ git merge hotfix
```

- 급한 문제를 해결하고 master 브랜치에 적용하고 나면 다시 일하던 브랜치로 돌아가야 한다. 
- 이제 더 이상 필요없는 hotfix 브랜치는 삭제한다. 
- git branch 명령에 -d 옵션을 주고 브랜치를 삭제한다

```
git branch -d hotfix
```

---

## 참조

- [Git-Documentation](https://git-scm.com/book/ko/v2)