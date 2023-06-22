+++
author = "lhj8390"
title = "[AWS] NACL과 Security Group의 개념 및 구축 예제"
date = "2023-06-21"
description = "AWS의 NACL과 Security Group(sg) 에 대해 설명한다."
tags = ["DevOps", "server", "AWS", "NACL", "Security Group", "sg"]
categories = [
    "DevOps"
]
subcategories = ["server"]
+++

> AWS에는 VPC의 네트워크 트래픽을 제어할 수 있는 서비스인 **네트워크 ACL**과 **보안 그룹**, 2개의 서비스를 제공한다. <br/>
> VPC에 대한 네트워크 접근을 제어하는 기본 메커니즘으로 보안 그룹(Security Group)을 사용하고, 필요한 경우 네트워크 ACL(NACL)을 사용하여 stateless하게 네트워크 제어를 수행할 수 있다.

<br/>


{{< figure src="/images/server_nacl_and_security_group/1.png" alt="image" >}}



## 네트워크 ACL(NACL) 이란?

서브넷 수준에서 서브넷 내의 모든 인바운드 또는 아웃바운드 트래픽을 허용하거나 거부할 수 있는 가상 방화벽이다. NACL은 기본적으로 모든 트래픽을 허용하도록 설정되어 있다.
Stateless 하기 때문에 요청과 응답에 대한 트래픽 (인바운드, 아웃바운드 규칙)을 별도로 설정해주어야 한다.


### NACL의 특징

- 서브넷 기반 : 하나의 서브넷에 대해 하나의 NACL만 지정할 수 있다.
- 순서 기반 규칙 : 여러 규칙이 충돌할 경우 숫자가 가장 작은 규칙이 우선적으로 적용된다.
- Stateless : 상태를 유지하지 않기 때문에 인바운드 규칙에서 지정한 트래픽이 아웃바운드 규칙에서 자동으로 허용되지 않는다.



## 보안 그룹(Security Group) 이란?

인스턴스에 대한 인바운드 및 아웃바운드 트래픽을 제어하는 가상 방화벽이다. 보안 그룹은 기본적으로 모든 트래픽을 차단하도록 설정되어 있다.
Stateful 하기 때문에 요청에 대한 응답 트래픽을 자동으로 허용해준다.



### 보안 그룹의 특징

- 인스턴스 기반 : 인스턴스 단위로 적용되며 각 인스턴스는 하나 이상의 보안 그룹에 속할 수 있다.
- Stateful : 인바운드 규칙을 통해 들어온 트래픽은 별도의 아웃바운드 규칙 없이 허용된다.
- Allow 규칙만 적용 : 특정 규칙에 대해 Deny는 불가능하고 Allow 규칙만 가능하다.



## NACL과 보안 그룹 비교

| network ACL           | Security Group    |
|:----------------------|:------------------|
| 서브넷 레벨에서 운영           | 인스턴스 레벨에서 운영      |
| 서브넷에서 배포된 모든 인스턴스에 적용 | 인스턴스에 연결된 경우에만 적용 |
| 허용 및 거부 규칙 지원         | 허용 규칙만 지원         |
| 낮은 번호부터 우선순위 부여       | 우선순위 없음           |
| 상태 비저장        | 상태 저장             |  



## 실전 구축

### 인바운드 규칙과 아웃바운드 규칙

인바운드(Inbound) 규칙은 외부에서 내부로 들어오는 트래픽을 제어하는 규칙이며, 아웃바운드(Outbound) 규칙은 내부에서 외부로 나가는 트래픽을 제어하는 규칙이다. 즉 인바운드는 출발지라고 보면 되고, 아웃바운드는 도착지라고 이해하면 된다.


### NACL과 보안 그룹 설정 예제

다음의 그림과 같이 NACL과 보안 그룹을 설정하는 것이 목표이다.
1. 10.0.0.1/24 대역대의 서브넷과 10.0.1.1/24 대역대의 서브넷이 서로 통신 가능
2. 다른 서브넷 안에 있는 Server1 과 Server2가 서로 통신 가능
3. 같은 서브넷 안에 있는 Server2와 Server4가 서로 통신 가능
4. 같은 보안 그룹 안에 있는 Server2와 Server3이 서로 통신 가능

<br/>

{{< figure src="/images/server_nacl_and_security_group/2.png" alt="image" >}}

우선 <font color="#c0504d">Server1에서 Server2로 트래픽이 전달되는 과정</font>을 기술한다.
1. <font color="#d99694">Server1의 보안 그룹 정책 확인</font> : Server2와 양방향 통신이 가능하도록 Inbound 규칙으로 Server2의 대역대를 지정한다. Inbound 규칙을 지정했기 때문에 별도의 Outbound 규칙 없이 트래픽을 보낼 수 있다.
2. <font color="#d99694">Server1의 서브넷의 NACL 정책 확인</font> : Server2의 서브넷으로 트래픽을 보내기 위해 Outbound 규칙을 지정한다. Server2의 대역대인 10.0.1.1/24를 허용하는 Outbound 규칙을 설정한다.
3. <font color="#d99694">Server2의 서브넷의 NACL 정책 확인</font> : Server1의 서브넷으로부터 온 트래픽을 받기 위해 Inbound 규칙을 확인한다. Server1의 대역대인 10.0.0.1/24를 허용하는 Inbound 규칙을 설정한다.
4. <font color="#d99694">Server2의 보안 그룹 정책 확인</font> : Server1로부터 통신이 가능하도록 Server1 대역대를 Inbound 규칙으로 지정한다. Inbound 규칙을 지정했기 때문에 별도의 Outbound 규칙 없이 Server1로 트래픽을 다시 보낼 수 있다.

이와 같은 설정을 통해 10.0.0.1/24 대역대의 서브넷과 10.0.1.1/24 대역대의 서브넷끼리 서로 통신이 가능하며, 각각의 서브넷 안에 있는 Server1 과 Server2가 서로 통신이 가능해진다.

같은 서브넷 안에 존재하는 Server2와 Server4가 서로 통신이 가능하게 하려면 보안 그룹의 Inbound 규칙만 추가하면 된다. Server2와 Server4를 포괄하는 대역대인 10.0.1.1/24 대역대를 Inbound 규칙에 설정한다. <font color="#7f7f7f">Server2와 Server4는 다른 보안 그룹이지만 같은 규칙을 적용받기 때문에 그림에서는 하나로 묶어서 표시하였다.</font> 이제 Server2와 Server4가 서로 통신이 가능하다.

Server2와 Server3은 같은 보안 그룹 내에 존재하기 때문에 별도의 설정 없이 서로 통신이 가능하다.

<br/>
<br/><br/>
