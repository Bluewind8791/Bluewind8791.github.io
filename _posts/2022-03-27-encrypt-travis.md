---
layout: post
title: "Travis CI로 설정파일 암호화 시키기"
description: aws travis application
summary: aws travis application
tags: aws
---


## before...

Spring Boot 프로젝트를 진행하면서 github에서도 public으로 올리다보니, 
나중에는 민감정보까지 `application.yml`에 들어가는 경우가 빈번했습니다.

이를 그냥 gitignore로 설정하여서 올리지않으면 상관없었지만, 
AWS를 다루면서 CI와 CD를 다루면서 깃헙과 연동이 되어 자동배포를 구현하니 이렇게 하는 방법은 비효율적이였습니다.

이를 jar의 외부 디렉토리로 빼서 설정하는 방법도 있었는데, 저는 개인적으로 현재 Travis CI를 사용중이라 이 방법이 편할 것 같아 공부해보았습니다.

또한, 이 암호화 방법은 윈도우에서는 `bad decrypt` 이라는 오류가 나기때문에 애초에 윈도우 사용자는 WSL(윈도우 서브시스템 리눅스)를 사용하여 진행하는것이 좋을 것 같습니다.


## 1. 로그인

- Travis CLI를 먼저 설치하고 Github의 repo와 연동이 되어있어야 합니다.
- 프로젝트의 디렉토리로 이동하여 아래의 명령어로 endpoint를 확인합니다.

```
travis endpoint
```

- 브라우저로 본인의 Travis CI 대쉬보드에 들어가서 URL를 확인하여 .com 으로 끝나있는 분들은 endpoint를 확인하여 org로 되어있다면 아래의 명령어로 com 으로 변경합니다.

```
travis endpoint --com --set-default
```

- https://github.com/settings/tokens 로 들어가서 personal access token을 만들어주세요.
- 'scope' 는 repo, read:org, gist를 선택합니다.
- 그리고 토큰이 복사되면 아래의 명령어로 로그인합니다.

```
travis login --pro --github-token <ghp_...로 시작하는 github token>
```

- 로그인에 성공하였다는 메세지가 뜨면 성공입니다.
- whoami 명령어로 제대로 로그인이 되었는지 확인할 수 있습니다.

```
travis whoami
```

## 2. 파일 암호화

- 저는 스프링 프로젝트의 application.yml 를 암호화 시키겠습니다.
- 아래의 명령어로 tar 파일로 압축합니다. 저는 아래와 같이 진행 하겠습니다.
- `No such file or directory`라는 오류때문에 tar 파일은 프로젝트의 루트 디렉토리에 두겠습니다.

```
// tar -cvf <tar파일 이름> <암호화할 파일 이름> <파일2>
$ tar -cvf setting.tar src/main/resources/application.yml
```

이제 Travis 로 파일을 암호화시킵니다.

```
// travis encrypt-file <암호화할파일명> --pro --add
$ travis encrypt-file setting.tar --pro --add
```

끝에 `--add` 명령어를 붙이면 .travis.yml 파일에 자동으로 명령어가 추가됩니다.

## 3. Github Push

암호화에 성공하였다면 아래와 같은 안내문이 뜨는것을 볼 수 있습니다.

```
Make sure to add setting.tar.enc to the git repository.
Make sure not to add setting.tar to the git repository.
Commit all changes to your .travis.yml.
```

`.gitignore` 에서 아래처럼 암호화시킨 파일 두개를 github에 올리지않도록 설정합니다.

```
application.yml
setting.tar
```

- `openssl...setting.tar -d` - 명령어는 travis로 암호화 할 때 --add 명령어로 자동으로 붙는 부분입니다.
- `tar xvf T.tar -C [PATH]` - tar 아카이브를 디렉토리에 풀때 -C 명령어를 주어 지정된 디렉토리에 풉니다.

```
...
before_install:
- openssl ...
  -in setting.tar.enc -out setting.tar -d
- tar xvf setting.tar -C src/main/resources
...
```

마지막으로 github에 push 하여 travis에서 build passing을 기도합니다.


## 참고

- [Travis CI] Travis CI application.yml 파일 암호화 복호화 하는 법 - [Gyun's 개발일지](https://devlog-wjdrbs96.tistory.com/319)
- 리눅스 tar 명령어 사용법 - [개발자를 위한 레시피](https://recipes4dev.tistory.com/146)
- Travis-CI openssl aes-256-cbc bad decrypt "Wrong final block" - [stackoverflow](https://stackoverflow.com/questions/47248236/travis-ci-openssl-aes-256-cbc-bad-decrypt-wrong-final-block)