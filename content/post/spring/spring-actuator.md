+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 스프링 액추에이터 사용"
author = "lhj8390"
description = "스프링 액추에이터를 사용하는 방법에 대해 설명한다."
date = "2022-11-20"
tags = ["spring", "web", "java", "spring boot", 'spring actuator']
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
> 스프링인액션 16장을 읽고 스프링 액추에이터 사용법에 대해 정리하였다.
> 
> - 스프링 액추에이터 활성화
> - 액추에이터 엔드포인트
> - 액추에이터 커스터마이징
> - 액추에이터 보안 처리
> 
> 에 대해 작성하였다.
> 

## 액추에이터란?

---

> 엑추에이터는 **스프링 부트 어플리케이션의 모니터링이나 메트릭**(metric)과 같은 기능을 HTTP와 JMX 엔드포인트를 통해 제공한다. 즉, 실행 중인 어플리케이션의 내부를 볼 수 있고 어플리케이션의 작동 방법을 제어할 수 있다.
> 

액추에이터가 노출하는 엔드포인트를 사용하여 얻을 수 있는 **정보**는 다음과 같다.

- 어플리케이션 환경에서 사용할 수 있는 구성 속성
- 패키지의 로깅 레벨
- 어플리케이션이 사용 중인 메모리
- 엔드포인트가 받은 요청 횟수
- 어플리케이션 건강 상태 정보

### 액추에이터 활성화

액추에이터를 활성화하려면 액추에이터의 스타터 의존성을 추가해야 한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

액추에이터를 활성화하면 어플리케이션에서 액추에이터 엔드포인트를 사용할 수 있다.<br/><br/>
액추에이터 엔드포인트 정보는 아래의 링크에서 확인할 수 있다.

[https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/production-ready-endpoints.html](https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/production-ready-endpoints.html)

### 액추에이터 설정

**액추에이터 기본 경로 구성**

모든 엔드포인트 경로에는 /actuator가 앞에 붙는데, 기본 경로를 변경한 예제이다.

```yaml
management:
  endpoints:
    web:
      base-path: /management
```

기본 경로는 application.yml 파일의 `management.endpoint.web.base-path` 속성에서 변경할 수 있다. 이후 건강 상태 정보를 얻고 싶다면 /management/health GET 요청을 보내면 된다.

**액추에이터 엔드포인트 활성화/비활성화**

엔드포인트 노출 여부를 제어하는 코드이다.

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, beans, conditions
```

 application.yml 파일의 `management.endpoint.web.exclude` 와  `management.endpoint.web.include` 속성을 통해 노출을 원하는 엔드포인트를 지정할 수 있다. exclude는 제외할 엔드포인트, include는 노출할 엔드포인트를 의미한다. 

<aside>
include 속성에는 <span class="red">와일드카드(*)</span>를 통해 모든 엔드포인트를 노출하도록 설정할 수 있다.<br/>
</aside>

## 액추에이터 엔드포인트 사용

---

> 엑추에이터의 HTTP 엔드포인트를 제공하는데, REST API 혹은 curl 명령어를 통해 엔드포인트를 사용할 수 있다. **/actuator GET 요청**을 하면 엔드포인트의 HATEOAS 링크를 응답으로 받을 수 있다.
> 

### 어플리케이션 기본 정보 받기

/info 엔드포인트를 통해 어플리케이션의 기본 정보를 알 수 있다.

```yaml
info:
  contact:
    email: email@email.com
    phone: 010-0000-0000
```

/info 엔드포인트가 반환하는 정보를 설정하는 방법은 여러가지가 있는데, 위에서 설정한 바와 같이 application.yml 에서 <span class="red">**info**로 시작하는 구성 속성을 설정</span>하는 방법도 있다.

/health 엔드포인트를 통해 어플리케이션의 건강 상태 정보를 얻을 수 있다.

```bash
$ curl localhost:8080/actuator/health
{"status": "UP"}
```

기본적으로 종합된 건강 상태 정보를 가지는 JSON 응답을 받는다.

**건강 상태를 나타내는 각 지표**

- 모든 건강 지표가 UP → 건강 상태 UP
- 하나의 건강 지표가 DOWN → 건강 상태 DOWN
- 하나의 건강 지표가 OUT_OF_SERVICE → 건강 상태 OUT_OF_SERVICE
- 하나의 건강 지표가 UNKNOWN → 종합 건강 상태에서 고려되지 않음

<br/>
건강 지표를 자세히 보려면 다음과 같이 구성한다.

```yaml
management:
  endpoint:
    health:
      show-details: always
```

`management.endpoint.health.show-details` 속성을 **always**로 설정하면 모든 건강 지표를 자세히 볼 수 있다. 기본값은 never이고, when-authorized로 설정 시 클라이언트가 인가된 경우에 한해 상세한 내역을 보여준다.

### 구성 상세 정보 보기

**빈 연결 정보**를 얻으려면 /beans 엔드포인트를 사용한다.

```json
{
  "contexts" : {
    "application-1": {
      "beans": {
        "ingredientsController": {
          "aliases": [],
          "scope": "singleton",
          "type": "tacos.ingredients.IngredientsController",
          "resource": "file [...ingredients/IngredientsController.class]",
          "dependencies": ["ingredientRepository"]
        },
      },
      "parentId": null
    }
  }
}
```

해당 어플리케이션 컨텍스트에 있는 모든 빈의 상세 정보를 포함한다. 

**자동-구성 내역**을 알기 위해서는 /conditions 엔드포인트를 사용한다.

```json
{
  "contexts" : {
    "application-1": {
      "positiveMatches": {
        "MongoDataAutoConfiguration#mongoTemplate": [
          {
            "condition": "OnBeanCondition",
            "message": "..."
          }
        ],
      },
      "negativeMatches": {
        "DispatcherServletAutoConfiguration": {
          "notMatched": [],
          "matched": []
        },
      },
      "unconditionalClasses": []  
    }
  }
}
```

**positiveMatches**에서는 자동 구성이 된 빈들을 보여주고, **negativeMatches**에서는 자동 구성이 어떠한 이유때문에 실패했다는 것을 보여준다. **unconditionalClassess**는 조건 없이 자동 구성된 빈을 보여준다.

**환경 속성과 구성 속성**을 보기 위해서는 /env 엔드포인트를 사용한다.

```json
{
  "activeProfiles": [
    "development"
  ],
  "propertySources": [
    {
      "name": "systemEnvironment",
      "properties": {
        "PATH": {
          "value": "/usr/bin:/bin:/usr/sbin:/sbin",
          "origin": "System Environment Property \"PATH\""
        }
      }
    },
    {
      "name": "applicationConfig: [classpath:/application.yml]",
      "properties": {
        "spring.application.name": {
          "value": "ingredient-service",
          "origin": "class path resource [application.yml]:3:11"
        },
        "server.port": {
          "value": 8081,
          "origin": "class path resource [application.yml]:9:9"
        },
      }
    },
  ]
}  
```

activeProfiles 필드를 통해 활성화된 프로파일을 알 수 있고, propertySources 필드에서는 스프링 어플리케이션 환경의 속성 근원들을 포함한다. 

<aside>
/env 엔드포인트에 POST 요청을 하면 실행 중인 어플리케이션의 속성을 설정할 수 있다.<br/>

</aside>
<br/>

**모든 HTTP 요청 핸들러 내역을 확인**하려면 /mappings 엔드포인트를 사용한다.

```json
{
  "contexts" : {
    "application-1": {
      "mappings": {
        "dispatcherHandlers": {
          "webHandler": [
            {
              "predicate": "{[/ingredients],methods=[GET]}",
              "handler": "...",
              "details": {
                "handlerMethod": {
                  "className": "tacos.ingredients.IngredientsController",
                  "name": "allIngredients",
                  "descriptor": "()Lreactor/core/publisher/Flux;"
                },
                "handlerFunction": null,
                "requestMappingConditions": {
                  "consumes": [],
                  "headers": [],
                  "methods": ["GET"],
                  "params": [],
                  "patterns": ["/ingredients"],
                  "produces": []
                }
              }  
            },
          ]
        }
      },
      "parentId": "application-1"
    }
  }
}
```

특정 API의 요청이 어떤 메서드에서 처리되는지 확인할 수 있다.

**로깅 레벨을 관리**하려면 /loggers 엔드포인트를 사용한다.

```json
{
  "levels": [ "OFF", "ERROR", "WARN", "INFO", "DEBUG", "TRACE" ],
  "loggers": {
    "ROOT": {
      "configuredLevel": "INFO", "effectiveLevel": "INFO"
    },
    "org.springframework.web": {
      "configuredLevel": null, "effectiveLevel": "INFO"
    }
  }
}
```

응답의 맨 앞에는 모든 로깅 레벨 내역이 노출되고, 각 패키지에 대한 로깅 상세 내역을 볼 수 있다. configuredLevel 속성은 명시적으로 구성된 로깅 레벨이고, effectiveLevel 속성은 상속받을 수 있는 유효 로깅 레벨이다.

<aside>
/loggers 엔드포인트에 POST 요청을 하면 로깅 레벨을 변경할 수 있다.<br/>

</aside>

### 어플리케이션 모니터링

**메모리나 스레드 문제를 찾는 gzip 형태의 HPROF 힙 덤프 파일을 다운로드**하려면 /heapdump 엔드포인트를 사용한다.<br/>

**어플리케이션이 처리한 최근 100개의 요청**을 알기 위해서는 /httptrace 엔드포인트를 사용한다.

```json
{
  "traces": [
    {
      "timestamp": "2018-06-04T23:41:24.494Z",
      "principal": null,
      "session": null,
      "request": {
        "method": "GET",
        "uri": "http://localhost:8081/ingredients",
        "headers": {
          "Host": ["localhost:8081"],
          "User-Agent": ["curl/7.54.0"],
          "Accept": ["*/*"]
        },
        "remoteAddress": null
      },
      "response": {
        "status": 200,
        "headers": {
          "Content-Type": ["application/json;charset=UTF-8"]
        }
      },
      "timeTaken": 4
    },
  ]
} 
```

스프링 부트 Admin에서는 이런 HTTP 추적 정보를 실시간으로 캡쳐하여 그래프로 보여준다.

**현재 실행 중인 스레드에 관한 스냅샷을 확인**하려면 /threaddump 엔드포인트를 사용한다.

```json
{
  "threadName": "reactor-http-nio-8",
  "threadId": 338,
  "blockedTime": -1,
  "blockedCount": 0,
  //...
  "stackTrace": [
    {
      "methodName": "kevent0",
      "fileName": "KQueueArrayWrapper.java",
      "lineNumber": -2,
      "className": "sun.nio.ch.KQueueArrayWrapper",
      "nativeMethod": true
     },
   ],
   "lockedMonitors": [],
   "lockedSynchronizers": [],
   "lockInfo": null
}
```

요청 시점의 스레드 활동에 대한 스냅샷을 확인할 수 있다. 스레드 덤프에는 스레드의 블로킹과 록킹 상태의 관련 상세 정보와 스택 기록 등을 보여준다. 

**메모리, 프로세스, 가비지 컬렉션, HTTP 요청 관련 메트릭**을 확인하려면 /metrics 엔드포인트를 사용한다.<br/>
/metrics/{메트릭 종류} 형식으로 사용한다면 해당 종류의 메트릭에 대한 상세한 정보를 알 수 있다.

```bash
$ curl localhost:8081/actuator/metrics/http.server.requests?tag=status:404&tag=uri:/**
{ 
  "name": "http.server.requests",
  "measurements": [
    { "statistic": "COUNT", "value": 30 },
    { "statistic": "TOTAL_TIME", "value": 0.519791548 },
    { "statistic": "MAX", "value": 0 }, 
  ],
  "availableTags": [
    { "tag": "exception", "values": [ "ResponseStatusException" ] },
    { "tag": "method", "values": [ "GET" ] }
  ]
}
```

tag에 stauts, url 태그 이름과 값을 지정하면 HTTP 404 응답을 받은 /** 경로에 대한 메트릭을 볼 수 있다. 

## 액추에이터 커스터마이징

---

> 어플리케이션의 특정 요구를 충족하기 위해 커스터마이징을 할 수 있다.
> 

### /info 엔드포인트 커스터마이징

<span class="red">InfoContributor라는 인터페이스</span>를 사용하여 /info 엔드포인트 응답에 커스텀 데이터를 추가할 수 있다.

```java
@Component
public class TacoCountInfoContributor implements InfoContributor {
  private TacoRepository tacoRepo;
  
  public TacoCountInfoContributor(TacoRepository tacoRepo) {
    this.tacoRepo = tacoRepo;
  }

  @Override
  public void contribute(Builder builder) {
    long tacoCount = tacoRepo.count();
    Map<String, Object> tacoMap = new HashMap<>();
    tacoMap.put("count", tacoCount);
    builder.withDetail("taco-stats", tacoMap);
  }
}
```

InfoContributor 인터페이스 구현체인 contribute() 메서드 **내부에 인자로 받은 Builder 객체의 withDetails() 를 호출하여** /info 엔드포인트에 정보를 추가할 수 있다.

**빌드 정보를 주입**하려면 build.gradle 파일에 다음과 같이 추가한다.

```groovy
springBoot {
  buildInfo()
}
```

**Git 커밋 정보를 주입**하려면 build.gradle 파일에 다음과 같이 추가한다.

```groovy
plugins {
  id "com.gorylenko.gradle-git-properties" version "1.4.17"
}
```

이 정보는 프로젝트가 빌드된 시점의 간단한 코드 상태만을 나타낸다. <br/>
자세한 정보를 얻기 위해서는 `management.info.git.mode` 속성을 full로 설정하면 된다. - <span class="gray">*application.yml*</span>

### 건강 지표 커스터마이징

<span class="red">HealthIndicator 인터페이스</span>를 사용하여 *스프링 부트에서 지원하지 않는* 외부 시스템의 건강 상태 정보를 제공할 수 있다.

```java
@Component
public class WackoHealthIndicator implements HealthIndicator {
  @Override
  public Health health() {
    int hour = Calendar.getInstance().get(Calendar.HOUR_OF_DAY);
    if (hour > 12) {
      return Health.outOfService().withDetail("reason", "out of service")
                   .withDetail("hour", hour).build();
    }
    
    if (Math.random() < 0.1) {
      return Health.outOfService().withDetail("reason", "break 10% of the time")
                   .build();
    }    
    
    return Health.up().withDetail("reason", "good").build();
  }
}
```

외부 시스템에 원격 호출을 한 후 받은 응답 기준으로 건강 상태를 결정하는 방식으로 유용하게 사용할 수 있다.

### 커스텀 메트릭 등록

Micrometer의 <span class="red">MeterRegistry</span>를 사용하여 스프링 부트 어플리케이션에서 메트릭을 발행할 수 있다. 메트릭을 발행할 때는 필요한 곳에 MeterRegistry를 주입하면 된다.

```java
@Component
public class TacoMetrics extends AbstractRepositoryEventListener<Taco> {
  private MeterRegistry meterRegistry;
  public TacoMetrics(MeterRegistry meterRegistry) {
    this.meterRegistry = meterRegistry;
  }

  @Override
  protected void onAfterCreate(Taco taco) {
    List<Ingredient> ingredients = taco.getIngredients();
    for (Ingredient ingredient : ingredients) {
      meterRegistry.counter("tacocloud", "ingredient", ingredient.getId()).increment();
    }
  }
}
```

스프링 데이터 클래스인 AbstractRepositoryEventListener의 서브 클래스를 생성하고 <span class="red">새로운 객체가 저장될 때마다 호출되도록 onAfterCreate() 메서드를 오버라이딩한다.</span>

### 커스텀 엔드포인트 생성

액추에이터 앤드포인트를 지정하려면 **@EndPoint** 어노테이션을 사용한다. 또한 @GetMapping, @PostMapping 대신 @ReadOperation, @WriteOperation, @DeleteOperation 어노테이션이 지정된 메서드로 정의된다.

```java
@Component
@Endpoint(id="notes", enableByDefault=true)
public class NotesEndpoint {

  private List<Note> notes = new ArrayList<>();
  
  @ReadOperation
  public List<Note> notes() {
    return notes;
  }
  
  @WriteOperation
  public List<Note> addNote(String text) {
    notes.add(new Note(text));
    return notes;
  }
  
  @DeleteOperation
  public List<Note> deleteNote(int index) {
    if (index < notes.size()) {
      notes.remove(index);
    }
    return notes;
  }
  
  @RequiredArgsConstructor
  private class Note {
    @Getter
    private Date time = new Date();

    @Getter
    private final String text;
  }
}
```

쓰기 오퍼레이션으로 데이터를 제출하고, 읽기 오퍼레이션으로 데이터를 읽을 수 있으며, 삭제 오퍼레이션으로 데이터를 삭제할 수 있다. enableByDefault=true로 지정했기 때문에 해당 엔드포인트를 구성 속성에서 활성화하지 않아도 된다.

엔드포인트를 **HTTP 엔드포인트로만 제한**하고 싶다면 @WebEndPoint 어노테이션을 사용한다.

```java
@Component
@WebEndpoint(id="notes", enableByDefault=true)
public class NotesEndpoint {
  ...
}
```

## 액추에이터 보안 처리

---

> 권한을 갖는 클라이언트만 엔드포인트를 사용하게 하기 위해서는 스프링 시큐리티를 사용해서 액추에이터의 보안을 처리해야 한다.
> 

액추에이터 엔드포인트의 공통 기본 경로인 /actuator 경로에 대한 보안을 추가하려면 다음과 같이 설정한다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.authorizeRequests()
        .antMatchers("/actuator/**").hasRole("ADMIN")
      .and()
      .httpBasic();
}
```

ROLE_ADMIN 권한을 가지는 사용자만 액추에이터 엔드포인트를 사용할 수 있다.

**엔드포인트 경로를 구성 속성에서 변경했다면** <span class="red">EndpointRequest 클래스</span>를 통해 변경된 경로를 알 수 있다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.requestMatcher(EndpointRequest.toAnyEndpoint())
        .authorizeRequests()
          .anyRequest().hasRole("ADMIN")
      .and()
      .httpBasic();
}
```

`EndpointRequest.toAnyEndpoint()` 메서드는 액추에이터 엔드포인트와 일치하는 요청 경로 모음을 반환한다. <br/>
**일부 엔드포인트를 제외**하고 싶으면 excluding() 메서드를 사용한다.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.requestMatcher(EndpointRequest.toAnyEndpoint().excluding("health", "info"))
        .authorizeRequests()
          .anyRequest().hasRole("ADMIN")
      .and()
      .httpBasic();
}
```
<br/>

**일부 액추에이터 엔드포인트에만 보안을 적용**하고 싶으면 to() 메서드를 호출한다.
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.requestMatcher(EndpointRequest.to("health", "info"))
        .authorizeRequests()
          .anyRequest().hasRole("ADMIN")
      .and()
      .httpBasic();
}
```