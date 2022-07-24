+++
author = "lhj8390"
title = "docker 기본 경로 변경"
date = "2022-07-24"
description = "도커의 기본 경로를 변경하는 방법에 대해 설명한다."
tags = ["DevOps", "server", "docker"]
categories = [
    "DevOps"
]
subcategories = ["docker"]
+++

## docker 용량 문제
---
> docker의 overlay로 인해 물리디스크가 100% 차지하는 경우가 발생한다.<br/>
> 안쓰는 이미지와 컨테이너 제거 및 도커의 기본 경로를 바꿔야 한다.

## 1. 안쓰는 이미지와 컨테이너 삭제

```bash
$ docker system prune -a -f 
```

- 컨테이너가 실행되고 있지 않은 것까지 모두 삭제
- 도커 기본 설치 경로는 /var/lib/docker<br/>
    → 볼륨의 기본 설치 경로는 /var/lib/docker/volumes
    

## 2. 도커의 기본 경로 바꾸기

```bash
# docker 종료
$ systemctl stop docker

# 설정파일 수정
$ sudo vi /lib/systemd/system/docker.service
```

### docker.service 설정 파일

```bash
[Service]
# ExecStart=/usr/bin/dockerd
ExecStart=/usr/bin/dockerd --graph=[변경할 경로]
```

### 설정파일 변경 후

```bash
# 데몬 리로드
$ systemctl daemon-reload

# 도커 시작
$ systemctl start docker
```