---
layout: post
title: "[AWS] EC2 스와핑으로 RAM 늘리기"
description: aws ec2
summary: aws ec2
tags: aws
---


## 1. 스왑 파일 생성

프리티어의 기본 RAM은 1G 이니 권장사항인 두배인 2G를 늘린다.

```
$ sudo dd if=/dev/zero of=/swapfile bs=128M count=16
```

## 2. 스왑 파일에 대한 읽기 및 쓰기 권한을 업데이트

```
$ sudo chmod 600 /swapfile
```

## 3. Linux 스왑 영역을 설정

```
$ sudo mkswap /swapfile
```

## 4. 스왑 공간에 스왑 파일을 추가하여 스왑 파일을 즉시 사용할 수 있도록 만든다


```
$ sudo swapon /swapfile
```

## 5. 절차가 성공했는지 확인

```
$ sudo swapon -s
```

## 6. /etc/fstab 파일을 편집하여 부팅 시 스왑 파일을 활성화

```
$ sudo vi /etc/fstab
```

vim 편집기로 실행하여 파일 끝에 다음과 같은 줄을 추가하고 :wq

`/swapfile swap swap defaults 0 0`


## 7. 메모리 확인

`free`



## 출처

- https://okky.kr/article/884329
- https://aws.amazon.com/ko/premiumsupport/knowledge-center/ec2-memory-swap-file/