+++
author = "lhj8390"
title = "ubuntu 포트 사용 확인하는 법"
date = "2022-07-28"
description = "ubuntu에서 포트가 사용중인지 확인하는 방법을 설명한다."
tags = ["DevOps", "server", "linux", "ubuntu"]
categories = [
    "DevOps"
]
subcategories = ["server"]
+++

## linux에서 포트가 열려 있는지 확인하는 명령어

- **nc** : netcat command
- **nmap** : network mapper tool
- **telnet**: telnet command
- **echo > /dev/tcp/..**
- **netstat - tuplen**

## netcat 사용법

**netcat 설치**

```
$ sudo apt-get install netcat
```
<br/>

**netcat 기본 명령어**

```
$ nc [-options] <host-ip-adress> <port-number>
```
<br/>

**주요 option**

- **z** : 스캐닝에 사용되는 제로 I/O 모드 
- **v** : 자세한 출력용 
- **w10** : 시간 초과 대기 초 

<br/>

**example**
    
```
$ nc -zvw10 192.168.0.1 22
```
    
성공하면 `Connection to <ip 주소><port> succeeded!` 가 표시된다.

## nmap 사용법

**nmap 기본 명령어**

```
$ nmap [-options] <IP 또는 hostname> -p <port>
```
<br/>

**주요 옵션**
- **-v** : 자세한 정보

<br/>

**example**
    
```
$ nmap - v 192.168.0.1 -p 22
```
- **open** : 스캔된 포트가 listen 상태
- **filtered** : 방화벽이나 필터에 막혀 해당 포트의 open, close 여부를 알 수 없을 때
- **closed** : 포트스캔을 한 시점에는 listen 상태가 아님
- **unfiltered** : nmap의 스캔에 응답은 하지만 해당 포트의 open, close 여부는 알 수 없을 때
    

## telnet 사용법

**telnet 기본 명령어**

```
$ telnet <IP 또는 Hostname> <Port>
```
- <span class="red">telnet: Unable to connect to remote host: Connection refused</span><br/>
    : 방화벽 오픈은 되었으나 listen 상태가 아니다.
    
- <span class="red">Trying <ip주소>....</span><br/>
    : 방화벽 오픈이 안되어 있다.
    
- <span class="red">Connected to <ip주소></span><br/>
    : 방화벽 오픈되어 있고 listen 상태