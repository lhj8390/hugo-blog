+++
author = "lhj8390"
title = "docker 개념"
date = "2022-07-23"
description = "도커의 개념 및 VM과의 차이점을 설명한다."
tags = ["DevOps", "server", "docker"]
categories = [
    "DevOps"
]
subcategories = ["docker"]
+++
## Docker란?

---

> linux 기반의 Container RunTime 오픈소스.
> 
> 
> 소프트웨어를 컨테이너라는 표준화된 유닛으로 패키징하며, 소프트웨어를 실행하는 데 필요한 모든 것이 포함되어 있다.<br/>
> `Virtual Machine`과 유사한 기능을 가지면서, 가벼운 형태로 배포가 가능하다.
> 

## VM과 Docker의 차이점

---

### VM

{{< figure src="/images/docker-basic/1.png" alt="image" class="left medium" >}}

<br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/>
Host OS 위에 Hypervisor 가 깔린 후에 Virtual Machine이 만들어진다.<br/>
- x86 하드웨어를 가상화한 것 → 다양한 종류의 OS 설치 가능하다.

### Docker

{{< figure src="/images/docker-basic/2.png" alt="image" class="left" >}}

<br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/>

Kernel은 Host OS 를 그대로 사용하고, Host OS 와 Container OS 의 차이가 되는 부분만 패키징한다.<br/>
→ Container 내에서 명령어를 수행하면 실제로는 Host OS 에서 명령어 수행

Container의 OS는 기본적으로 Linux 만 지원

## Docker 특징

---
### Repository 연계

Application들을 Container로 패키징해서 <span class="red">다른 환경으로 쉽게 옮길 수 있다.</span><br/>
→ local pc에서 mysql, apache, tomcat등을 각 Container에 넣어서 개발을 한 후에 테스트 환경으로 옮길때, Container를 repository에 저장했다가 똑같은 테스트환경을 꾸밀 수 있다.
<br/><br/>

- **Base Image** : 기본 install 이미지<br/>
    ex) Ubuntu OS
    
- **Docker file :** 기본 install 이미지와 그 위에 추가로 설치되는 스크립트<br/>
    ex) Ubuntu OS + Apache, MySQL