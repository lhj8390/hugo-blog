+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 스프링 관리"
author = "lhj8390"
description = "스프링 부트 Admin을 생성 및 관리하고 액추에이터 데이터를 수집하는 방법에 대해 설명한다."
date = "2022-12-09"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

> 스프링인액션 17장을 읽고 스프링 관리에 대해 정리하였다.
> 
> - 스프링 부트 Admin 설정
> - 클라이언트 어플리케이션 등록
> - 액추에이터 엔드포인트 사용하기
> 
> 에 대해 작성하였다.
> 

## 스프링부트 Admin

---

> 스프링 부트 Admin은 관리용 프론트엔드 웹 어플리케이션으로, 액추에이터 엔드포인트를 사용할 수 있도록 한다. Admin 서버는 스프링 부트 클라이언트라는 하나 이상의 스프링 부트 어플리케이션으로부터 제공되는 액추에이터 데이터를 수집하고 보여준다.
> 

### Admin 서버 생성

스프링 부트 어플리케이션에 Admin 서버 의존성을 빌드에 추가해야 한다.

```groovy
implementation 'de.codecentric:spring-boot-admin-starter-server'
```

스프링 부트 Initializr를 사용할 경우 ‘Spring Boot Admin (Server)’ 체크박스를 선택한다.

구성 클래스에서 @EnableAdminServer 어노테이션을 통해 Admin 서버를 활성화한다.

```java
@SpringBootApplication
@EnableAdminServer
public class BotAdminServerApplication {
  public static void main(String[] args) {
    SpringApplication.run(BootAdminServerApplication.class, args);
  }
}
```

마지막으로 application.yml 파일에서 포트를 설정한다.

```yaml
server:
  port: 9090
```

localhost:9090 포트로 접근하면 Admin 서버 UI를 확인할 수 있다.

### Admin 클라이언트 등록

Admin 서버가 다른 어플리케이션을 알 수 있도록 클라이언트를 등록해야 한다. 클라이언트를 Admin 서버에 등록하는 방법은 두가지가 있다.

1. 각 어플리케이션이 자신을 Admin 서버에 등록한다.
2. Admin 서버가 유레카 서비스 레지스트리를 통해 서비스를 찾는다.

<br/>

**1. 어플리케이션이 Admin 서버에 등록**

어플리케이션이 자신을 Admin 서버의 클라이언트로 등록하려면 Admin 클라이언트 스타터 의존성을 빌드에 추가해야 한다.

```groovy
implementation 'de.codecentric:spring-boot-admin-starter-client'
```

스프링 부트 Initializr를 사용할 경우 ‘Spring Boot Admin (Client)’ 체크박스를 선택한다.

application.yml 파일에 다음과 같이 구성해준다.

```yaml
spring:
  application:
    name: ingredient-service
  boot:
    admin:
      client:
        url: http://localhost:9090
```

Admin 서버의 위치와 클라이언트의 이름을 설정해준다.

<br/>

**2. 유레카 서비스 레지스트리 사용**

Admin 서버가 서비스들을 찾을 수 있게 활성화하려면 Admin 서버에 Netflix 유레카 클라이언트 스타터를 추가하면 된다. 

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
```

유레카에 등록된 모든 어플리케이션을 Admin 서버가 자동으로 찾기 때문에 따로 application.yml 파일 구성을 하지 않아도 된다.

다음은 Admin 서버 별도의 구성 예시이다.

```yaml
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka1.tacocloud.com:8761/eureka/
```

Admin 서버를 유레카 서비스로도 등록하려면 `register-with-eureka` 속성을 false로 설정한다. 또한 유레카 서버의 위치를 따로 구성하려면 `service-url.defaultZone` 속성을 이용한다.

## Admin 서버 보안

---

> Admin 서버는 다음과 같은 정보를 제공한다.
> 
> - 어플리케이션 건강 상태 정보 / 일반 정보
> - Micrometer를 통해 발행되는 메트릭과 /metrics 엔드포인트
> - 환경 속성
> - 패키지와 클래스 로깅 레벨
> - 스레드 추적 정보
> - HTTP 요청 추적 기록
> - 감사 로그
> 
> 이러한 정보는 관리자만이 봐야하는 정보로, 아무에게나 함부로 노출되거나 변경되면 안된다. 
> 

### Admin 서버 보안 활성화

Admin 서버에 보안을 추가하려면 스프링 시큐리티를 사용하여 처리할 수 있다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-security'
```

스프링 부트 Initializr를 사용할 경우 ‘Spring Security’ 체크박스를 선택한다.

관리자 이름과 비밀번호를 설정하지 않으면 무작위로 생성되기 때문에 application.yml 파일에서 이름과 비밀번호를 설정해준다.

```yaml
spring:
  security:
    user:
      name: admin
      password: 53cr3t
```

### 액추에이터로 인증

HTTP 기본 인증으로 액추에이터 엔드포인트의 보안을 처리할 수 있는데, 액추에이터 엔드포인트가 인증 정보를 제공하지 않으면 Admin 서버도 액추에이터 엔드포인트를 사용할 수 없다. Admin 서버가 인증정보를 얻으려면 <span class="red">클라이언트 어플리케이션이 자신의 인증 정보를 Admin 서버에 제공해야 한다.</span>

application.yml 파일에 Admin 서버가 액추에이터 엔드포인트에 접근하는 데 사용할 수 있는 인증 정보를 지정할 수 있다.

```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:9090
        instance:
          metadata:
            user.name: ${spring.security.user.name}
            user.password: ${spring.security.user.password}
```

인증 정보는 Admin 서버에 자신을 등록하는 각 클라이언트 어플리케이션에 설정되어 있어야 한다.

유레카를 통해 클라이언트 어플리케이션을 발견하도록 했다면 application.yml 파일에 다음과 같이 지정한다.

```yaml
eureka:
  instance:
    metadata-map:
      user.name: admin
      user.password: password
```

어플리케이션이 유레카에 등록할 때 인증 정보는 유레카 등록 레코드의 메타데이터에 포함된다. Admin 서버는 해당 인증 정보를 유레카로부터 가져온다.