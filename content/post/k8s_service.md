+++
author = "lhj8390"
title = "쿠버네티스 서비스"
date = "2022-06-04"
description = "쿠버네티스의 서비스 개념 및 종류에 대해 설명한다."
tags = ["DevOps", "server", "docker", "kubernetes", "k8s"]
series = ["쿠버네티스 개념"]
categories = [
    "Kubernetes"
]
+++

# 서비스

## 서비스의 개념

동일한 서비스를 제공하는 파드 그룹에 지속적인 단일 접점을 만들려고 할 때 생성하는 리소스
<br/><br/>

### ◼ **서비스의 필요성**

파드는 비영구적 리소스이기 때문에 파드 내의 IP 주소는 랜덤하게 지정되고, 리스타트 될 때마다 변경될 수 있다.

파드는 고정된 엔드포인트로 호출이 어렵다.

서비스를 사용하게 되면 파드가 클러스터내의 어디에 있는지에 상관없이 고정된 주소를 이용해서 접근이 가능하다.

→ “_서비스를 통해 고정 IP 주소를 설정하여 내, 외부 클라이언트가 파드에 접속할 수 있다.”_

{{< figure src="/images/ab861e53f3e043aa808be27b243103e0/1.png" alt="image" caption="외부 클라이언트가 frontend service에 접속, 내부 클라이언트(frontend pod)가 backend service에 접속" class="medium" >}}

<br/><br/>

## ServiceType 종류

- **ClusterIP**
  서비스를 클러스터 내부 IP에 노출시킨다. ServiceTypes의 기본 값.
  클러스터 내에서는 이 서비스에 접근이 가능하지만, 클러스터 외부에서는 외부 IP 를 할당 받지 못했기 때문에 접근이 불가능하다.
- **NodePort**
  고정 포트로 각 노드의 IP에 서비스를 노출시킨다.
  `<NodeIP>:<NodePort>` 를 요청하여, 클러스터 외부에서 NodePort 서비스에 접속할 수 있다.
- **LoadBalancer**
  클라우드 벤더에서 제공하는 설정 방식으로, 로드 밸런서를 사용하여 서비스를 외부에 노출시킨다.
  외부 로드 밸런서가 라우팅되는 NodePort와 ClusterIP 서비스가 자동으로 생성된다.
- **ExternalName**
  서비스를 externalName 필드의 콘텐츠 (예: foo.bar.example.com)에 매핑한다. 설정해둔 CNAME값이랑 연결된다.

<br/><br/>

## 서비스 생성 (ClusterIP)

> 레이블 셀렉터를 이용해 같은 레이블의 파드를 서비스에 연결한다.
> 기본 서비스 타입.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

◼️ **설명**

서비스가 80포트를 사용하여 `app=MyApp` 레이블을 가진 파드의 TCP 8080 포트를 대상으로 라우팅한다.

![Untitled](/images/ab861e53f3e043aa808be27b243103e0/2.png)

**Cluster IP:** 클러스터 안의 Pod끼리 통신하기 위한 Private IP

**External IP**: 클러스터 외부에 공개하는 IP 주소 (쿠버네티스 클러스터에서는 별도로 관리 X)

서비스에 할당된 주소는 10.109.128.30으로 클러스터 내부, 컨테이너 내부에서 접속을 시도할 경우 my-service와 연결된 파드의 8080포트가 실행되는 것을 확인할 수 있다.
<br/>
<br/>

### 멀티포트 지원

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8080
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9090
```

◼️ **설명**

서비스는 동시에 하나의 포트 뿐 아니라 여러개의 포트를 동시에 지원한다.

하나의 서비스를 이용해 포트 80과 443을 각각 8080, 9090 포트와 연결할 수 있다.

→ “단일 클러스터 IP로 모든 서비스 포트가 노출된다.”
<br/><br/>

💡 **주의점**

멀티포트 서비스를 만들 때는 각 포트의 이름을 지정해주어야 한다.

<br/><br/>

## NodePort

> 모든 노드에 특정 포트를 할당하고 서비스를 구성하는 파드로 들어오는 연결을 전달한다.
>
> Cluster IP와 유사하지만 서비스의 내부 클러스터IP 뿐만 아니라 모든 노드의 IP와 할당된 NodePort로 서비스에 액세스할 수 있다.
>
> <br/>
> → 서비스를 클러스터 밖에 위치한 외부 IP 주소에 노출하고 싶은 경우에 사용한다.
> <br/>
> <br/>

### NodePort 생성

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 31000
```

◼️ **설명**

nodePort는 기본적으로 생략이 가능하다.

생략 시, 쿠버네티스 컨트롤 플레인은 포트 범위에서 자동 할당해준다 (기본값: 30000-32767)

<br/>

💡 **주의점**

nodePort를 생략할 경우 서비스 재시작 시 nodePort는 계속 변경된다.
<br/><br/>

### 내부 POD와 통신하는 방법

![Untitled](/images/ab861e53f3e043aa808be27b243103e0/3.png)

◼️ **설명**

노드의 특정 IP 192.168.1.1 의 31000 포트에 접근한다면 트래픽이 가장 원활한 곳에 자동으로 연결을 시도한다.

해당 트래픽은 서비스의 80포트를 찾고, 컨테이너 내부 80포트를 읽는 구조로 진행된다.

<br/><br/>
💡 **주의점**

특정 노드에 장애가 발생 시, 클라이언트는 해당 서비스에 액세스할 수 없게 되고 자동으로 다른 노드를 통해 접근이 불가능하다.
이를 해결하기 위해서는 별도의 로드밸런서가 필요하다.

<br/><br/>

## **ExternalName**

> FQDN(정규화된 도메인 이름)으로 외부 서비스를 참조하는 방법
> 쿠버네티스 내부에서 외부 서비스를 호출하고자 할때 사용한다.

일반적인 셀렉터에 대한 서비스가 아닌, DNS 이름에 대한 서비스에 매핑한다

> <br/>

- FQDN(Fully Qualified Domain Name)이란?
  호스트 이름과 도메인 이름을 포함한 전체 도메인 이름을 일컫는 용어이다.

<br/><br/>

### **ExternalName 생성**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ExternalName
  externalName: my.database.example.com
```

◼️ **설명**

서비스를 `my.database.example.com`에 매핑한다.

서비스 프록시를 완전히 무시하고 외부 서비스에 직접 연결된다.

→ 따라서 ClusterIP를 얻지 못한다.

### 다른 네임스페이스의 서비스 참조

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
	namespace: a
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
	namespace: b
spec:
  type: ExternalName
  externalName: my-service.a.svc.cluster.local
```

◼️ **설명**

파드가 다른 namespace의 서비스에 접근해야 하는 경우에도 ExternalName을 이용할 수 있다.

b 네임스페이스의 external-service:8080 포트로 연결되는 모든 트래픽은

a 네임스페이스의 my-service로 라우팅된다. **(my-service.a.svc.cluster.local)**

<br/><br/>
💡 **특징**

서비스를 사용하는 파드에서 _실제 서비스 이름과 위치가 숨겨져_
나중에 externalName 속성을 변경하거나 서비스 스펙을 수정하면 다른 서비스를 가리킬 수 있다.

<br/><br/>
