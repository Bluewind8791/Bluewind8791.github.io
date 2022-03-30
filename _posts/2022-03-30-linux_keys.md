---
layout: post
title: "[Linux] 리눅스 자주쓰는 명령어 모음"
description: aws ec2 linux
summary: aws ec2 linux
tags: aws linux
---


## nohup

- nohup은 no hang up 의 약자.
- 로그아웃으로 세션과의 연결이 종료되더라도 nohup 으로 돌린 프로세스는 데몬 형태로 실행되게 되어 종료되지 않고 계속 실행되게 됩니다.
- nohup으로 실행시킬 파일은 반드시 755 permission을 가지고 있어야 합니다.
- & 를 명령어 맨 끝에 붙여주면 해당 프로그램이 백그라운드로 실행됩니다.

### 자주쓰는 명령어

nohup으로 해당 jar 파일을 백그라운드에서 실행 (끝에 & 주의)

```
nohup java -jar -Dspring.profiles.active=set1 *.jar &
```

파일의 마지막 10라인을 실시간으로 계속 출력한다

```
tail -f nohup.out
```

tail -n 10 nohup.out (마지막 부터 10줄을 출력한다)

---

## 상태 확인

현재 실행중인 포트 확인

```
sudo netstat -tnlp
```

해당 서비스 상태 확인

```
service nginx status
```
```
ps ef | grep java
```

---

## Nginx

Nginx error log 확인

```
tail -f /var/log/nginx/error.log
```

---

## 부팅 시 자동 실행

리눅스에서 /etc/init.d/에 스크립트 파일이 있으면 부팅시 자동으로 실행

```
sudo vim /etc/init.d/codedeploy-startup.sh
```

스크립트 파일에 아래의 내용을 추가합니다

```
#!/bin/bash

echo 'Starting codedeploy-agent'
sudo service codedeploy-agent start
```

---

## Java 11

EC2

```
# aws coreetto 다운로드
sudo curl -L https://corretto.aws/downloads/latest/amazon-corretto-11-x64-linux-jdk.rpm -o jdk11.rpm

# jdk11 설치
sudo yum localinstall jdk11.rpm

# jdk version 선택
sudo /usr/sbin/alternatives --config java

# java 버전 확인
java --version

# 다운받은 설치키트 제거
rm -rf jdk11.rpm
```

---


## tar



## 참고

- 쉽게 설명한 nohup 과 &(백그라운드) 명령어 사용법 - [바로가기](https://joonyon.tistory.com/98)