+++
author = "lhj8390"
title = "[AWS] Elastic Container Service (ECS)"
date = "2023-07-09"
description = "AWS의 Elastic Container Service, ECS의 개념에 대해 설명한다."
tags = ["DevOps", "server", "AWS", "ECS"]
categories = [
    "DevOps"
]
subcategories = ["server"]
+++

> Amazon ECS (Elastic Container Service) 는 AWS에서 컨테이너화된 애플리케이션을 쉽게 배포, 관리, 스케일링할 수 있도록 도와주는 완전 관리형 컨테이너 오케스트레이션 서비스이다.

{{< figure src="/images/aws-ecs/1.png" alt="image" >}}

<br/>

## ECS 의 특징

1. 기존의 AWS 서비스를 잘 결합해서 컨테이너 배포에 맞도록 서비스를 구축하였기 때문에 손쉽게 배포 및 운영이 가능하다.
2. AWS 서드 파티 도구에 기본적으로 통합되기 때문에 인프라가 아닌 어플리케이션을 구축하는 데에 집중할 수 있다.

<br/>

## ECS 구성 요소

{{< figure src="/images/aws-ecs/2.png" alt="image" >}}

- Cluster : 컨테이너 인스턴스들의 논리적인 그룹
- Container Instance : ECS 에이전트가 실행되고 클러스터에 등록되는 EC2 인스턴스
- Task Definition : 컨테이너를 실행하기 위해 정의한 JSON 파일
- Task : Task Definition에서 정의된 설정으로 실제 배포된 컨테이너들
- Service : Task Definition에 따라 Task 실행 및 Task 유지 관리 (스케쥴러 역할)
- Container : 작업 인스턴스화 중에 생성된 도커 컨테이너

Kubernetes 와 비교하자면 Container instance는 k8s의 Node, Service는 k8s의 Service 와 Controller 를 결합한 개념이고, Task 는 k8s 의 Pod를 의미한다.

<br/>

## ECS 동작 방식

{{< figure src="/images/aws-ecs/3.png" alt="image" >}}

- **Agent Communication** : ECS Agent는 실행할 작업에 대해 CPU, 메모리, GPU 용량을 사용할 수 있도록 ECS control plain에 보고하고 control plain은 Agent에게 응답으로 프로그램 작업을 시작하도록 지시한다. ECS Agent가 예기치 않게 종료되거나 호스트의 종료 알림을 지속적으로 모니터링한다.
- **Task monitoring** : 작업 수명 주기 및 작업 상태 점검 모니터링
- **API** : 개발자가 의도에 따라 ECS 를 구성할 수 있도록 API 제공
- **Task Scheduling** : ECS가 수행하는 작업의 핵심. 사용자가 설정한 구성을 실질적으로 수행하는 엔진. 클러스터 상태 정보를 활용하여 적절한 배치 결정을 내린다. 

<br/>

## ECS Launch Types
ECS는 EC2, Fargate(서버리스), external 총 3가지의 Launch Type으로 인스턴스를 생성할 수 있다.

### EC2 Cluster
- 명시적으로 EC2 인스턴스 프로비저닝
- EC2 인스턴스를 직접 관리해야 하는 필요성 존재
- EFS 및 EBS 지원
- 직접 클러스터 최적화 처리 필요
- 과금 정책 : 실행 중인 EC2 인스턴스 마다 부여

즉, 호스트 수준의 제어가 가능해 사용자가 직접 커스터마이징이 가능하다.

### Fargate Cluster
- 리소스를 자동으로 프로비저닝
- 제한된 제어 및 인프라 자동화
- EFS 및 EBS 지원 없음
- Fargate가 클러스터 최적화
- 과금 정책 : 실행 중인 Task 마다 부여

awsvpc networking mode 라는 aws 자체적으로 제공하는 네트워킹 모드만 지원한다.

<br/>

## Capacity Provider
Capacity Provider 란 자동 스케일링 그룹의 스케일링을 관리하는 ECS의 새로운 개념이다.
Fargate Capacity Provider 과 AutoScaling Capacity Provicer 가 존재한다.


### Capacity Provider 등장 배경
{{< figure src="/images/aws-ecs/4.png" alt="image" >}}

기존의 **Infrastructure-First 환경**의 경우 AutoScaling Group 설정에 따라 먼저 ECS 서버를 Scale Out 한 후 Task를 실행할 수 있었다. 따라서 EC2 서버가 미처 생성되지 않았다면 EC2가 프로비저닝될 때 까지 Task 를 실행할 수 없어 고가용성을 보장할 수 없었다.
<br/>
하지만 **Application-First 환경** 으로 넘어감에 따라 어플리케이션이 주도권을 가지게 되고 Capacity Provider 개념이 등장하게 되었다. Capacity Provider 가 직접 어플리케이션의 요구사항에 대응하고 Scale in, Scale out 여부를 관리함으로써 <span class="red">비용이 절감하고 고가용성을 보장</span>할 수 있게 되었다.

<br/>

## Task Definition
컨테이너를 실행하기 위해 정의한 JSON 파일
JSON 형식으로 구성되며 최대 10개의 컨테이너를 정의할 수 있다. 정의한 모든 컨테이너는 동일한 호스트에 위치한다.

``` json
{
  "family": "nginx-demo",
  "containerDefinitions": [
    {
      "name": "nginx",
      "image": "nginx",
    }
  ]
}
```

컨테이너 이름, 컨테이너 이미지, CPU, Memory, 네트워킹 모드, 로깅, Port Mapping, volume 등을 정의할 수 있다.

### 네트워킹 모드
- Awsvpc : EC2 인스턴스와 동일한 네트워크 속성 적용
- Bridge : EC2 인스턴스에서 실행하는 Docker의 기본 가상 네트워크 이용
- Host : Docker의 가상 네트워크를 우회하여 컨테이너 포트를 호스팅하는 EC2 인스턴스의 ENI에 직접 매핑
- None : 외부 네트워크 연결 無

<br/><br/><br/>