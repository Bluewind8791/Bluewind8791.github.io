---
layout: post
title: "[AWS] EC2 표준시간 세팅하기"
description: aws ec2
summary: aws ec2
tags: aws
---


## 1. 시스템의 현재 표준 시간대 설정 확인

```
$ timedatectl
```

## 2. 사용 가능한 표준 시간대 확인

```
$ timedatectl list-timezones
```

여기에서 Asia/Seoul 을 확인.


## 3. 표준 시간대 설정

```
$ sudo timedatectl set-timezone Asia/Seoul
```



## 출처
- https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/set-time.html#change_time_zone