+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 실패와 지연 처리"
author = "lhj8390"
description = "스프링에서 실패와 지연을 처리하는 방법에 대해 설명한다."
date = "2022-10-30"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

> 스프링인액션 15장을 읽고 실패와 지연 처리에 대해 정리하였다.
> 
> - 서킷 브레이커 패턴 개요
> - Hystrix로 실패와 지연 처리
> - 서킷 브레이커 모니터링
> - 서킷 브레이커 메트릭 종합
> 
> 에 대해 작성하였다.
> 

## 서킷 브레이커란?

---

> 서킷 브레이커 패턴은 작성한 코드가 실행에 실패하는 경우 안전하게 처리되도록 도와준다. **마이크로서비스의 연쇄적인 실패를 막기 위해** 서킷 브레이커 패턴은 마이크로서비스 컨텍스트에서 훨씬 더 중요하다.
> 
<br/>

**서킷 브레이커 처리 흐름**

{{<figure src="/images/spring-circuit-breaker/1.png" class="large" caption="서킷 브레이커 패턴은 폴백을 사용하여 실패를 처리한다.">}}

메서드가 실행에 성공하면 서킷은 닫힘 상태를 유지하지만 실행에 실패하게 된다면 서킷은 열림 상태가 되고 <span class="red">실패한 메서드 대신 폴백 메서드가 호출된다.</span> 지정된 시간에 맞춰 절반-열림 상태로 전환되면 실패했던 메소드를 재실행하고 성공한다면 서킷은 닫힘 상태가 된다.

<br/>

**서킷 브레이커는 다음과 같은 유형의 메서드에 적용될 수 있다.**

- REST API : 사용할 수 없거나 HTTP 500 응답을 반환하는 서비스로 인해 실패할 수 있는 메서드
- 데이터베이스 쿼리: 데이터베이스가 무반응 상태이거나 스키마의 변경으로 인해 어플리케이션이 중단될 가능성이 있는 메서드
- 지연: 너무 오랫동안 실행되어 비정상적인 상태로 보이는 메서드

서킷 브레이커 패턴은 코드의 실패와 지연을 처리하는 강력한 수단이다.

## Hystrix

---

> Netflix Hystrix는 서킷 브레이커 패턴을 자바로 구현한 라이브러리이다. 대상 메서드가 실패할 때 폴백 메서드를 호출하는 aspect(어스펙트)로 구현된다. 대상 메서드의 실패율을 체크하고, 실패율이 한계값을 초과하면 모든 대상 메서드 호출을 폴백 메서드 호출로 전달한다.
> 

### 빌드 추가

```groovy
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
}

ext {
    set('cloud-version', "Hoxton.SR3")
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:{cloud-version}"
    }
}
```

서킷 브레이커를 선언하기 전 스프링 클라우드 Netflix Hystrix 스타터를 각 서비스의 빌드에 추가해야 한다. 또한 스프링 클라우드의 버전도 지정해주어야 한다.

### Hystrix 활성화

Hystrix를 활성화하기 위해서는 @EnableHystrix 어노테이션을 지정하면 된다.

```java
@SpringBootApplication
@EnableHystrix
public class IngredientServiceApplication {
    ...
}
```

Hystrix를 활성화한 다음에는 서킷 브레이커를 선언해야 한다.

```java
@HystrixCommand(fallbackMethod="getDefaultIngredients")
public Iterable<Ingredient> getAllIngredients() {
    ...
}
```

**@HystrixCommand** 어노테이션을 사용하여 getAllIngredients() 메서드에 대한 폴백 메서드를 제공해주었다. <span class="red">getAllIngredients() 메서드에서 예외 발생 시 서킷 브레이커가 해당 예외를 잡아서 폴백 메서드인 getDefaultIngredients() 메서드를 호출해 준다.</span>

→ *폴백 메서드는 원래의 메서드와 시그니처가 같아야 한다.*

### 지연시간 줄이기

기본적으로 @HystrixCommand가 지정된 메서드는 <span class="red">1초 후에 타임아웃되고</span> 해당 메서드에 대한 폴백 메서드가 호출된다. <br/>
서브 브레이커의 **타임아웃 시간을 변경**하려면 다음과 같이 사용한다.

```java
@HystrixCommand(fallbackMethod="getDefaultIngredients", 
                commandProperties={
                    @HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds", value="500")          
                })
public Iterable<Ingredient> getAllIngredients() {
    ...
}
```

@HystrixCommand 어노테이션의 commandProperties 속성을 통해 Hystrix 명령 속성인 `execution.isolation.thread.timeoutInMilliseconds` 을 500으로 설정하였다.

**타임아웃이 필요 없을 경우**에는 다음과 같이 사용한다.

```java
@HystrixCommand(fallbackMethod="getDefaultIngredients", 
                commandProperties={
                    @HystrixProperty(name="execution.timeout.enabled", value="false")           
                })
public Iterable<Ingredient> getAllIngredients() {
    ...
}
```

`execution.timeout.enabled` 을 false로 설정하면 타임아웃을 없앨 수 있다. 타임아웃이 없으면 지연 시간이 보호되지 않으므로 <span class="red">연쇄 지연 효과가 발생할 가능성이 있다.</span> 따라서 타임아웃 비활성화를 할 경우에는 조심해야 한다.

### 서킷 브레이커 한계값 관리

Hystrix 명령 속성을 통해 실패와 재시도 한계값을 설정할 수 있다.

**관련 명령 속성**

- `circuitBreaker.requestVolumeThreshold` : 지정된 시간 내에 메서드가 호출되어야 하는 횟수
- `circuitBreaker.errorThresholdPercentage` : 지정된 시간 내에 실패한 메서드 호출 비율
- `metrics.rollingStats.timeInMilliseconds` : 요청 횟수와 에러 비율이 고려되는 시간
- `circuitBreaker.sleepWindowInMilliseconds` : 절반-열림 상태가 되기 전 열림 상태의 서킷이 유지되는 시간

**한계값 적용 예시**

```java
@HystrixCommand(fallbackMethod="getDefaultIngredients", 
                commandProperties={
                    @HystrixProperty(name="circuitBreaker.requestVolumeThreshold", value="30"),
                    @HystrixProperty(name="circuitBreaker.errorThresholdPercentage",value="25"),    
                    @HystrixProperty(name="metrics.rollingStats.timeInMilliseconds", value="20000"),            
                    @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds", value="60000"),       
                })
public Iterable<Ingredient> getAllIngredients() {
    ...
}
```

20초 이내에 메서드가 30번 이상 호출 시 이중에서 25% 이상이 실패할 경우에 폴백 메서드가 실행되도록 하는 코드이다. 이때 서킷 브레이커는 절반-열림 상태가 되기까지 1분간 열림 상태를 유지해야 한다.

## 실패 모니터링

---

> 서킷 브레이커가 보호하는 메서드가 호출될 때마다 이에 관한 여러 데이터가 수집되어 Hystrix 스트림으로 발행된다. **이 Hystrix 스트림은 실행 중인 어플리케이션의 건강 상태를 실시간으로 모니터링하는 데에 사용할 수 있다.**
> 

Hystrix 스트림에 포함되는 데이터는 다음과 같다.

- 메서드 호출 횟수
- 성공 호출 횟수
- 폴백 메서드 호출 횟수
- 타임아웃 횟수

### 액추에이터 설정

이 Hystrix 스트림은 액추에이터 엔드포인트로 제공되며 모든 서비스들이 <span class="red">Hystrix 스트림을 활성화하려면 액추에이터 의존성을 추가해야 한다.</span>

```groovy
implementation 'org.springframework.boot:spring-cloud-starter-actuator'
```

Hystrix 스트림은 /actuator/hystrix.stream 경로에 노출되어 있다. 

다음은 **Hystrix 스트림을 액추에이터 엔드포인트로 활성화**하는 코드이다.

```yaml
management:
  endpoints:
    web:
      expsure:
        include: hystrix.stream
```

각 어플리케이션의 application.yml에 다음 구성을 추가하면 Hystrix 스트림 엔드포인트를 활성화할 수 있다.

### Hystrix 대시보드 설정

Hystrix 대시보드를 사용하기 위해서는 우선 Hystrix 대시보드 스타터 의존성을 추가해야 한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix-dashboard'
```

스프링 부트 Initializr를 사용할 경우 Hystrix Dashboard 체크박스를 선택하면 된다.

Hystrix 대시보드를 활성화하는 코드이다.

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardApplication {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class, args);
    }
}
```

메인 구성 클래스에 @EnableHystrixDashboard 어노테이션을 지정해야 한다.

### Hystrix 스레드 풀

어떤 메서드가 지연 상태일 때 그 메서드를 호출하는 메서드가 같은 스레드의 컨텍스트에서 실행 중이라면 호출하는 메서드는 <span class="red">지연 상태의 메서드를 기다릴 수 밖에 없다.</span>

이런 상황을 방지하기 위해 Hystrix는 각 의존성 모듈의 스레드 풀을 할당하여 **하나가 호출될 때 Hystrix가 관리하는 스레드 풀의 스레드에서 실행된다.** *- <span class="gray">잠재적인 스레드 포화를 Hystrix에서 관리할 수 있다.</span>*

Hystrix 대시보드에서 제공하는 스레드 풀 모니터다.


{{<figure src="/images/spring-circuit-breaker/2.png" class="large" caption="스레드 풀 모니터는 Hystrix가 관리하는 각 스레드 풀에 관한 중요한 통계 데이터를 보여준다.">}}



## 다수의 Hystrix 스트림 종합

---

> Hystrix 대시보드는 한 번에 하나의 스트림만 모니터링할 수 있다. 따라서 어플리케이션 전체의 건강 상태 정보를 얻는 것은 불가능하다. 이때 사용하는 것이 **Turbine**이다.<br/>
> Turbine은 모든 마이크로서비스로부터 모든 Hystrix 스트림을 Hystrix 대시보드가 모니터링 할 수 있는 <span class="red">하나의 스트림으로 종합하는 방법을 제공</span>한다.
> 

### Turbine 설정

Turbine 서비스를 생성하려면 Turbine 스타터 의존성을 추가해야 한다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-turbine'
```

Turbine 서비스를 활성화시키는 방법은 다음과 같다.

```java
@SpringBootApplication
@EnableTurbine
public class TurbineServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(TurbineServerApplication.class, args);
    }
}
```

어플리케이션의 메인 구성 클래스에 @EnableTurbine 어노테이션을 지정한다. 

Turbine은 유레카 클라이언트로 작동하므로 Hystrix 스트림을 종합할 서비스들을 유레카에서 찾는다. Hystrix 스트림을 종합할 서비스들을 알 수 있게 구성을 설정해준다.

```yaml
turbine:
  app-config: ingredient-service, taco-service, order-service, user-service
  cluster-name-expression: "'default'"
```

`turbine.app-config` 속성에 Hystrix 스트림을 종합하기 위해 유레카에서 찾을 서비스 이름을 설정한다.<br/>
`turbine.cluster-name-expression` 속성을 통해 이름이 default인 클러스터에 있는 모든 스트림들을 수집해야 함을 알린다. 해당 속성은 필수 속성으로, 설정하지 않으면 Turbine 스트림에 종합될 스트림을 포함할 수 없다.