---
layout: post
title: "[Docker] Permission denied 해결"
description: aws ec2 docker error
summary: aws ec2 docker error
tags: aws
---


새 인스턴스를 다루다보면 아래와 같은 permission 문구가 뜨면서 docker 관련 명령어가 제대로 먹지 않는 현상을 자주 목격한다.

```
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock:
Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/json: dial unix /var/run/docker.sock: connect: permission denied
```


## 해결방법

방법은 간단하다.


```
sudo groupadd docker
sudo usermod -aG docker $USER
```


exit로 SSH 로그아웃 후 다시 SSH 접속한다.