+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 마이크로서비스 이해"
author = "lhj8390"
description = "스프링에서 마이크로서비스의 개념에 대해 설명한다."
date = "2022-10-10"
tags = ["spring", "web", "java", "spring boot", "microservice"]
categories = ["Framework"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

> 스프링인액션 13장을 읽고 마이크로서비스에 대해 공부하였다.
> 
> - 마이크로서비스란?
> - 서비스 레지스트리 생성하기
> - 서비스 등록, 발견
> 
> 에 대해 작성하였다.
> 

## 마이크로서비스

---

> 단일 어플리케이션은 **다음과 같은 한계**가 존재한다.
> 
> 1. 전체를 파악하기 어렵다.
> 2. 테스트가 더 어렵다.
> 3. 라이브러리 간의 충돌이 생기기 쉽다.
> 4. 확장 시에 비효율적이다.
> 5. 적용할 테크놀로지를 결정할 때 전체 어플리케이션을 고려해야 한다.
> 6. 프로덕션으로 이양 시 많은 노력이 필요하다.
> 
> 이러한 문제를 해결하기 위해 마이크로 서비스 아키텍처가 개발되었다. 
> 

### 마이크로서비스 아키텍처

마이크로서비스 아키텍처란 <span class="red">애플리케이션이 독립적인 구성 요소로 구축되어 각 애플리케이션 프로세스가 서비스로 실행되는 아키텍처</span>를 뜻한다.

단일 어플리케이션 아키텍처와 대조적으로 다음과 같은 특징이 있다.

1. 마이크로 서비스는 쉽게 이해할 수 있다.
2.  테스트가 쉽다.
3. 라이브러리 비호환성의 문제가 생기지 않는다.
4. 독자적으로 규모를 조정할 수 있다.
5. 각 마이크로서비스에 적용할 테크놀로지를 다르게 선택할 수 있다.
6. 언제든 프로덕션으로 이양할 수 있다.

하지만 마이크로서비스의 <span class="ul">원격 호출이 많을수록 어플리케이션의 실행이 느려질 수 있다</span>는 점과, 어플리케이션의 <span class="ul">규모가 상대적으로 작다면 단일 어플리케이션이 더 적합하다</span>는 사실을 고려하여 마이크로서비스 아키텍처를 선택해야 한다.

## 서비스 레지스트리 설정

---

### 유레카란?

유레카는 마이크로서비스 어플리케이션에 있는 모든 서비스의 **중앙 집중 레지스트리**로 작동한다. 서로 다른 서비스들이 서로를 찾을 수 있도록 각 서비스들은 유레카 서비스 레지스트리에 자신을 등록한다.

{{<figure src="/images/spring-microservice/1.png" class="large">}}

그림에서 other-service는 some-service의 어떤 인스턴스를 사용할 것인지 결정해야 하는데, 인스턴스를 매번 선택하는 대신 클라이언트 측에서 동작하는 <span class="red">로드 밸런싱 알고리즘</span>인 “**리본**”을 적용한다.

<aside>
💡 <strong>왜 리본을 사용할까?</strong>

리본은 **각 클라이언트에서 실행되는 클라이언트 측의 로드밸런서**이다.
각 클라이언트에 하나의 로컬 로드밸런서가 있기 때문에 클라이언트 수에 비례하여 로드밸런서의 크기가 조정된다. 또한 각 클라이언트에 적합한 로드 밸런싱 알고리즘을 적용할 수 있다.

</aside>

### 유레카 서버 의존성 추가

유레카 서버 스타터 의존성을 추가하는 코드이다.

```groovy
dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
}
```

스프링 클라우드 버전 의존성도 추가한다.

```groovy
ext {
  set('cloud-version', "Hoxton.SR3")
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-dependencies:{cloud-version}"
  }
}
```

### 유레카 서버 활성화

```java
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {

  public static void main(String[] args) {
    SpringApplication.run(ServiceRegistryApplication.class, args);
  }
}
```

@EnableEurekaServer 어노테이션을 추가하여 유레카 서버를 활성화시킬 수 있다.

### 유레카 서버 구성

```yaml
server:
  port: 8761
  
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server:
    enable-self-preservation: false
```

- server.port : 유레카가 실행되는 포트
- eureka.instance.hostname : 유레카가 실행되는 호스트 이름
- fetchRegistry, registerWithEureka : 유레카와 상호작용하는 방법을 명시 <span class="gray">(기본값 - true)</span>
  - `fetchRegistry` : 서비스 검색 시 레지스트리에 있는 정보 받아올지 여부
  - `registerWithEureka` : 서비스 인스턴스 등록과 함께 유레카에 등록할 것인지 여부
- serviceUrl : 하나 이상의 유레카 서버 URL 포함. Map 형식이다.
  - defaultZone : 클라이언트가 자신이 원하는 영역을 지정하지 않았을 때 사용
- enable-self-preservation : 자체 보존 모드
  - **자체 보존 모드** : 서비스 인스턴스로부터 등록 갱신 요청을 받지 못하면 서비스 인스턴스 등록이 취소되고, 추가적인 서비스 인스턴스의 등록 취소가 방지된다.
    

### 유레카 확장

프로덕션 환경에서는 고가용성을 위해 최소한 두 개의 유레카 인스턴스를 가져야 한다.

두 개의 유레카를 구성하는 yaml 파일이다.

```yaml
spring:
  profiles: eureka-1
  application:
    name: eureka-1
server:
  port: 8761
eureka:
  instance:
    hostname: eureka1.tacocloud.com
other:
  eureka:
    host: eureka2.tacocloud.com
    port: 8762
---
spring:
  profiles: eureka-2
  application:
    name: eureka-2
server:
  port: 8762
eureka:
  instance:
    hostname: eureka2.tacocloud.com
other:
  eureka:
    host: eureka1.tacocloud.com
    port: 8762
```

두 개의 프로파일인 eureka-1과 eureka-2가 구성되어 있다. 각 프로파일에 설정된 다른 유레카 인스턴스를 참조하기 위해 other.eureka.host와 other.eureka.port 속성을 설정한다.

fetchRegistry나 registerWithEureka를 설정하지 않으면 기본값인 true로 지정되어 각 유레카 서버가 다른 유레카 서버에 자신을 등록하고 레지스트리 등록 정보를 받아올 수 있다.

## 서비스 등록하고 찾기

---

> 서비스를 다른 서비스에서 찾아 사용하게 하려면, 유레카 서비스 레지스트리의 클라이언트로 활성화시켜야 한다.
> 

### 유레카 클라이언트 의존성 추가

```groovy
dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
}
```

서비스 레지스트리 클라이언트로 활성화하기 위해 유레카 클라이언트 스타터 의존성을 추가해준다.</br>
<span class="gray">※ 스프링 클라우드 버전 의존성도 추가해주어야 한다.</span>

### 유레카 클라이언트 속성 구성

```yaml
server:
  port: 0
spring:
  application:
    name: ingredient-service
eureka:
  client:
    service-url: 
      defaultZone: http://eureka1.com:8761/eureka/, http://eureka2.com:8761/eureka/
```

- spring.application.name : 서비스 이름이 중복으로 되는 것을 방지하기 위해 서비스 이름을 지정한다.
- server.port : 서비스의 포트 충돌을 막기 위해 랜덤 포트인 0을 지정한다.
    - 서비스는 유레카를 통해서만 찾으므로, 리스닝하는 포트는 상관이 없다.
- service-url : 프로덕션 시 **유레카 서버의 위치를 지정**할 경우 사용한다.
    - 첫번째 URL에서 등록이 실패하면 두번째 URL의 레지스트리에 등록을 시도한다. 만약 첫번째 유레카 서버가 온라인 상태가 되면 해당 서비스의 서버 레지스트리가 복제된다.

### 서비스 사용

유레카 서버에서 서비스를 찾을 때 여러 개의 서비스 인스턴스가 반환될 경우가 있다. 이 때, 컨슈머 어플리케이션은 서비스 인스턴스를 선택하지 않아도 되며 **리본 클라이언트 로드 밸런서를 사용하여 서비스 인스턴스를 쉽게 찾아 사용할 수 있다.**

**서비스를 선택 및 사용하는 방법**

- 로드 밸런싱된 RestTemplate
- Feign에서 생성된 클라이언트 인터페이스

### RestTemplate 사용하여 서비스 사용

RestTemplate 빈을 선언하는 코드이다.

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
  return new RestTemplate();
}
```

@LoadBalanced 어노테이션을 통해 해당 RestTemplate이 (1) <span class="ul">리본을 통해서만 서비스를 찾는다는 것을 스프링 클라우드에게 알려준다.</span> 또, (2) <span class="ul">주입 식별자로 동작</span>할 수 있게 한다.</br>
→ *getForObject() 메서드의 HTTP 요청에서 호스트와 포트 대신 주입 식별자를 사용할 수 있다.*
<br/><br/>
주입 식별자를 사용한 예시이다.

```java
public Ingredient getIngredientById(String ingredientId) {
  return rest.getForObject("http://ingredient-service/ingredients/{id}", Ingredient.class, ingredientId);
}
```
RestTemplate는 ingredient-service라는 이름의 서비스를 찾아 인스턴스를 선택하도록 리본에 요청한다. <br/><br/>

RestTemplate 대신 WebClient를 사용한 예시이다.

```java
@Bean
@LoadBalanced
public WebClient.Builder webClientBuilder() {
  return new WebClient.Builder();
}
```

RestTemplate을 사용했던 것과 동일한 방법으로 수행할 수 있다.

### Feign 클라이언트 인터페이스 정의

Feign은 REST 클라이언트 라이브러리이며, 인터페이스를 기반으로 하는 방법을 사용해서 REST 클라이언트를 정의한다.<br/><br/>

**Feign 활성화**

Feign 의존성을 추가하는 코드이다.

```groovy
dependencies {
  implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
}
```
Feign 의존성을 추가한 후 구성 클래스에서 Feign을 활성화시키는 코드이다.

```java
@Configuration
@EnableFeignClients
public class FeignClientConfig {
}
```

@EnableFeignClients 어노테이션을 통해 Feign을 활성화시킬 수 있다.
<br/><br/>

**Feign 인터페이스 생성**

```java
@FeignClient("ingredient-service")
public interface IngredientClient {

  @GetMapping("/ingredients/{id}")
  Ingredient getIngredient(@PathVariable("id") String id);

  @GetMapping("/ingredients")
  Iterable<Ingredient> getAllIngredients();

}
```

ingredient-service라는 이름으로 유레카에 등록된 서비스를 사용하는 클라이언트를 작성하였다. 해당 인터페이스에 선언된 메서드를 사용하면 ingredient-service 이름의 서비스에 대한 요청이 이루어지게 된다.