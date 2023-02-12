+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 구성 속성 사용"
author = "lhj8390"
description = "스프링에서 구성 속성을 사용하는 방법에 대해 설명한다."
date = "2022-07-20"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
## 개요

---

> 스프링인액션 5장을 읽고 스프링 부트 구성 속성에 대해 작성해본다.
> 
> - 자동-구성되는 빈 조정하기
> - 구성 속성을 애플리케이션 컴포넌트에 적용하기
> - 스프링 프로파일 사용하기
>
> 에 대해 알 수 있다.
> 

## 스프링의 구성

---

스프링 구성에는 두 가지 형태가 존재한다.

- 빈 연결 : 스프링 애플리케이션 컨텍스트에서 빈으로 생성되는 애플리케이션 컴포넌트 및 상호 간에 주입되는 방법을 선언하는 구성
- 속성 주입 : 스프링 애플리케이션 컨텍스트에서 빈의 속성 값을 설정하는 구성

<aside>
💡 스프링에서 자바 기반 구성을 사용하고 있다면 <strong>@Bean</strong> 애노테이션을 통해 빈의 인스턴스를 생성하고 속성 값도 설정한다.<br/>

</aside>

### 데이터 소스 구성하기

스프링 부트에서 구성 속성을 통해 간단히 데이터베이스의 URL과 인증을 구성할 수 있다.
<br/><br/>
MySQL 데이터베이스를 설정하는 예시이다.

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/dbname
    username: username
    password: passwd
    driver-class-name: com.mysql.jdbc.Driver
```

이 DataSource 빈을 자동-구성할 때 스프링 부트가 이 속성을 연결 데이터로 사용한다.
<br/><br/>
데이터베이스를 초기화하는 SQL 스크립트를 실행해야 할 경우 다음과 같이 사용한다.

```yaml
spring:
  datasource:
    schema: 
      - order-schema.sql
      - ingredient-schema.sql
    data:
      - ingredients.sql
```

### 내장 서버 구성하기

스프링 부트는 자체적으로 톰캣 내장 서버가 존재한다. 이 내장 서버도 구성 속성을 사용해 구성할 수 있다.
<br/><br/>
포트를 무작위로 설정하는 예시이다.

```yaml
server:
  port: 0
```

server 포트를 0으로 설정한다면 내장 서버가 0번 포트로 시작하지 않고 사용 가능한 포트로 무작위로 선택하여 실행된다.<br/>
해당 구성을 통해 (1)<span class="ul">자동화된(동적) 테스트를 실행</span>하거나, (2) <span class="ul">마이크로서비스를 이용</span>할 때 유용하다.
<br/><br/>
내장 서버의 HTTS를 활성화시키는 방법이다.

```yaml
server:
  port: 0
  ssl:
    key-store: file:///path/to/mykeys/jks
    key-store-password: letmein
    key-password: letmein
```

ssl.key-store : 키스토어 파일이 생성된 경로로 설정 (파일 시스템: `file://`, JAR 파일: `classpath:`)<br/>
ssl.key-store-password, ssl.key-password : 키스토어를 생성할 때 지정했던 비밀번호 설정

### 로깅 구성하기

스프링 부트는 logback.xml 파일을 생성하지 않고 로그 구성을 변경할 수 있다.<br/><br/>

로그를 설정하는 예시이다.

```yaml
logging:
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```

루트의 로깅 수준은 WARN으로 하되, 스프링 시큐리티의 로그는 DEBUG 수준으로 설정한다는 의미이다.<br/><br/>

로그들을 logging.log 파일에 저장하도록 설정할 경우 다음과 같이 사용한다.

```yaml
logging:
  path: /var/logs/
  file: logging.log
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```

애플리케이션이 /var/logs/ 경로에 대해 권한을 가지고 있다면 로그 항목들이 해당 경로에 저장된다.

## 프로파일 설정하기

---

애플리케이션이 서로 다른 런타임 환경에 배포될 때에는 구성 명세가 달라진다.<br/>
프로파일을 이용해서 환경에 따라 서로 다른 빈, 구성 클래스, 구성 속성들이 적용, 무시되도록 설정할 수 있다.

### 프로파일 속성 정의

프로파일에 특정한 속성을 정의하는 방법은 두 가지가 존재한다.

1. 특정 환경의 속성들만 포함하는 또다른 .yml 파일이나 .properties 파일을 생성한다.<br/>
    ※ 파일 이름은 application-{파일이름}.yml 이런 식의 규칙을 따라야 한다.
    
2. 공통으로 적용되는 기본 속성과 함께 프로파일 특정 속성을 .yml 에 지정할 수 있다.

```yaml
logging:
  level:
    tacos: DEBUG
---
spring:
  profiles: prod
  datasource:
    url: jdbc:mysql://localhost/dbname
    username: username
    password: passwd
logging:
  level:
    tacos: WARN
```

3개의 하이픈(—-)을 통해 yml 파일을 두 부분으로 구분짓는다.<br/>
첫번째 부분은 공통 영역, 두 번째 부분은 prod 프로파일만 적용되는 부분이다.

### 프로파일 활성화

프로파일은 기본적으로 다음과 같이 활성화할 수 있다.

```yaml
spring:
  profiles:
    active:
      -dev
```

하지만 이처럼 구성할 경우 해당 프로파일이 기본 프로파일로 지정되어 프로덕션 환경과 개발 속성을 분리할 수 없게 된다.

이를 해결하기 위해서는 다음과 같이 명령행 인자로 활성화 프로파일을 설정할 수 있다.

```shell
% java -jar app.jar --spring.profiles.active=prod
```

이 명령어를 통해 기본 프로파일 속성(dev)보다 더 높은 우선순위를 부여받을 수 있게 된다. 

### 빈 생성하기

빈은 활성화되는 프로파일과는 무관하게 생성되지만 특정 프로파일이 활성화될 때만 생성되어야 하는 빈들이 존재한다.<br/>
이 경우에는 @Profile 어노테이션을 활용한다.

```java
@Bean
@Profile("dev")
public CommandLineRunner dataLoader(IngredientRepository repo, serRepository userRepo, 
                                     PasswordEncoder encoder) {
  ...
}
```

해당 빈은 데이터를 내장 데이터베이스에 로드할 때 사용하는 것으로, 개발환경에서는 필요하지만 프로덕션 환경에서는 불필요하다.<br/>
따라서 이 메서드에 @Profile을 지정하여 개발 환경에서만 실행되도록 설정하였다.<br/><br/>

여러가지 Profile을 지정할 경우에는 다음과 같이 사용한다.

```java
@Profile({"dev", "qa"})
```
<br/>
특정 Profile을 제외할 경우에는 다음과 같다.

```java
@Profile("!prod")
```