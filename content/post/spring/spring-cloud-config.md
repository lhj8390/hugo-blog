+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 클라우드 구성 관리"
author = "lhj8390"
description = "스프링에서 클라우드를 구성하는 법에 대해 설명한다."
date = "2022-10-16"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Framework"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
> 스프링인액션 14장을 읽고 클라우드 구성 관리에 대해 공부하였다.
> 
> - 스프링 클라우드 구성 서버 실행
> - 구성 서버 클라이언트 생성
> - 보안에 민감한 구성 정보 저장
> - 구성 자동으로 리프레시
> 
> 에 대해 작성하였다.
> 

## 중앙 집중적인 구성 관리

---

> 스프링 어플리케이션을 구성할 때, 간단한 어플리케이션이라면 자바 시스템 속성이나 운영체제의 환경 변수를 구성 속성으로 사용하거나 application.yml 파일에 구성 속성을 지정한다. 이 때, 속성을 변경해야 한다면 <span class="red">어플리케이션을 재시작시키거나, 재빌드 및 배포를 수행</span>해야 한다.
> 
> 
> 하지만 속성만 변경하기 위해 어플리케이션을 재배포하거나 재시작한다면 매우 불편하며, 다수의 인스턴스를 가지고 있는 마이크로서비스 기반 어플리케이션의 경우에는 모든 서비스 인스턴스에 동일한 변경을 적용하는 것이 불합리하다.
> 

### 중앙 집중적인 구성 시 장점

1. **구성이 어플리케이션 코드에 패키징되어 배포되지 않는다.** <br/>
    : 재빌드 및 재배포 없이 구성을 변경할 수 있다. 어플리케이션 실행 중에 구성 변경이 가능하다.
    
2. **동일한 속성들을 공유할 수 있다.**<br/>
    : 한번만 변경해도 모든 마이크로서비스에 적용할 수 있다.
    
3. **보안에 민감한 구성 속성들은 별도로 암호화하고 유지, 관리할 수 있다.**<br/>
    : 복호화된 속성은 언제든 사용 가능하며, 복호화 코드가 어플리케이션에 없어도 된다.
    

## 구성 서버 실행

---

> 스프링 클라우드 구성 서버는 집중화된 구성 데이터 소스를 제공하며, 같은 어플리케이션에 있는 다른 서비스들의 구성 데이터를 제공하는 역할을 수행한다.
> 

### 구성 서버 활성화

마이크로서비스 구성 서버는 별개의 어플리케이션으로 개발되어 배포되므로 구성 서버를 사용하기 위해서는 새로운 구성 서버를 생성해야 한다.<br/><br/>

**스프링 클라우드 구성 서버 의존성 추가**<br/>
스프링 클라우드 구성 서버 스타터 의존성을 추가하는 코드이다.
```groovy
dependencies {
  implementation 'org.springframework.cloud:spring-cloud-config-server'
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
<br/>

**부트스트랩** **클래스 설정**<br/>

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

  public static void main(String[] args) {
    SpringApplication.run(ConfigServerApplication.class, args);
  }
}
```

@EnableConfigServer 어노테이션을 통해 어플리케이션이 실행될 때 구성 서버를 활성화 시킨다.<br/><br/>

**구성 속성 위치 명시**

```yaml
server:
  port: 8888
spring:
  cloud:
    config:
      server:
        git:
          uri : https://github.com/habma/tacocloud-config
```

구성 서버가 처리할 구성 속성들이 있는 곳을 알려준다. 여기서는 github 주소를 사용했기 때문에 `spring.cloud.config.server.git.uri` 속성에 github repository URL을 설정해주었다. <br/>
또 구성 서버는 기본적으로 8080 포트를 리스닝하는데, 포트 충돌을 막기 위해 포트 번호를 고유값으로 지정해준다. (이 포트번호는 구성 서버의 클라이언트 포트와 같아야 한다)

### 구성 서버 GET 요청

구성 서버 경로는 세 부분으로 이루어져 있으며, 경로에 대해 HTTP GET 요청을 수행한다.

{{<figure src="/images/spring-cloud-config/1.png" class="large" caption="구성 서버 URL에 대한 설명">}}


- 첫번째 부분 : 어플리케이션 이름.
- 두번째 부분 : 활성화된 스프링 프로파일 이름.
- 세번째 부분 : 구성 속성을 가져올 백엔드 Git repository의 라벨/분기. 생략 가능

요청에 대한 응답으로는 구성 서버가 제공하는 것에 관한 기본 정보가 포함된다.

### Git repository에 구성 속성 저장

가장 기본적인 방법은 Git repository의 루트 경로에 application.yml 파일을 커밋하는 것이다. 커밋 후 구성 서버에게 curl 명령을 사용한다면 다음과 같은 응답을 받는다.

```bash
$ curl localhost:8888/someapp/someconfig
```

```json
{
  "name": "someapp",
  "profiles": [
    "someconfig"
  ],
  "label": null,
  "version": "95df0cbc3ba106199bd804b27a1de7c3ef5c35e",
  "state": null,
  "propertySources": [
    {
      "name": "http://localhost:10080/tacocloud/tacocloud-config/application.yml",
      "source": {
        // Git 내부 application.yml 파일 내용
      }
    }
  ]
}
```

propertySources 속성의 배열에는 **name**과 **source**라는 두 개의 속성이 포함되는데, 

- name 속성은 <span class="bg_y">Git repository를 참조하는 uri 값</span>을 가지고
- source 속성은 해당 <span class="bg_y">Git repository에 저장했던 구성 속성을 포함</span>한다.

루트 경로대신 **하위 경로에 구성 속성을 저장**할 수도 있다.<br/>
구성서버 자체 속성을 지정한 application.yml 파일을 변경한다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri : https://localhost:10080/tacocloud/tacocloud-config
          search-paths: config, more*
```

search-paths 속성을 통해 하위 경로를 지정해줄 수 있다. 또한 search-paths는 복수형이며, 구성 속성을 여러 하위 경로에 저장할 수 있다.

**특정 분기나 라벨에 구성 속성을 저장**할 수도 있다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri : https://localhost:10080/tacocloud/tacocloud-config
          default-label: sidework
```

sidework라는 이름의 분기에 저장된 구성 속성을 가져올 때는 이와 같이 사용한다. 해당 분기는 default 분기로 지정된다.

**Git 백엔드 인증 처리**를 위해서는 다음과 같이 사용한다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri : https://localhost:10080/tacocloud/tacocloud-config
          username : tacocloud
          password : passwd123
```

사용자 이름(username), 비밀번호(password)를 사용하여 Git repository 인증을 수행한다.

## 구성 데이터 사용

---

> 스프링 클라우드 구성 서버는 클라이언트 라이브러리도 제공한다. 해당 라이브러리가 어플리케이션 빌드에 포함되면 구성 서버의 클라이언트가 될 수 있다.
> 

### 클라이언트 지정

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-config'
```

스프링 Initializr 화면의 <span class="red">Config Client 체크박스</span>를 선택해도 빌드에 추가할 수 있다.

### 구성 서버 설정

```yaml
spring:
  cloud:
    config:
      uri : http://config.tacocloud.com:8888
```

spring.cloud.config.uri 속성을 통해 구성 서버의 위치를 알려줄 수 있다. 기본적으로 구성 서버는 localhost의 8888 포트로 실행중인 것으로 간주한다.<br/>
→ 이 속성은 구성 서버의 클라이언트가 되는 어플리케이션 자체에서 설정되어야 한다. 중앙 집중식 구성 서버가 있을 경우에는 각 마이크로서비스가 자신의 구성을 가질 필요는 없다.

## 특정 속성 제공하기

---

> 구성 서버 클라이언트는 어플리케이션 이름과 활성화된 프로파일을 포함하는 요청 경로를 사용해서 구성 서버에게 속성을 요청한다.
> 

### 어플리케이션에 특정 속성 제공

구성 서버는 특정 어플리케이션을 대상으로 하는 구성 속성을 관리할 수 있다. <br/>
4개의 마이크로서비스가 존재할 경우 각 서비스 어플리케이션의 application.yml 파일을 생성한다. application.yml 파일 내부의 <span class="red">spring.application.name 속성이 구성 서버에 전송</span>되고, <span class="red">이 속성값과 일치하는 구성 파일이 있을 경우 해당 속성들이 반환</span>된다.

### 프로파일로부터 속성 제공

스프링 클라우드 구성 서버는 스프링 부트 어플리케이션에서 프로파일을 활성화하는 것과 동일한 방법으로 프로파일에 특정된 속성을 지원한다.

- 프로파일에 특정된 application-production.yml, application-dev.yml 같은 이름의 구성 파일을 제공한다.
- 하나의 yml 파일 내부에 여러 개의 프로파일 구성 그룹을 추가할 수 있다. 3개의 하이픈(---)을 추가하고 spring.profiles 속성을 지정한다.

## 구성 속성 보안 유지하기

---

> 구성 서버에서 민감한 정보들(비밀번호나 보안토큰 등)을 포함하는 속성들을 제공할 때 다음과 같은 옵션을 제공한다.
> 
> - 구성파일에 암호화된 값 쓰기
> - 구성 서버의 백엔드 저장소로 해시코프의 Vault 사용하기

### 속성 암호화하기

암호화된 속성을 사용하려면 **암호화 키**를 사용해서 구성 서버를 구성하고, 암호화 키를 이용하여 속성 값을 클라이언트에게 제공하기 전에 복호화 한다.

대칭키를 설정하려면 구성 서버 자체 구성의 encrypt.key 속성에 암호화 키와 복호화 키를 설정하면 된다.

```yaml
encrypt:
  key: s3cr3t
```

이 속성은 구성 서버를 활성화시키기 전에 로드되어 사용할 수 있기 때문에 <span class="red">부트스트랩 구성(bootstrap.yml)에 설정되어야 한다. </span><br/>
강력한 보안을 위해서 구성 서버가 한 쌍의 비대칭 RSA 키나 키스토어 참조를 사용하도록 구성할 수 있다.

키를 생성하는 예제이다.

```bash
$ keytool -genkeypair -alias tacokey -keyalg RSA -dname "CN=Web Server ,OU=Unit ,O=Organization ,L=City ,S=State ,C=US" -keypass s3cr3t -keyStore.jks -storepass l3tm31n
```

keystore.jks이름으로 키스토어가 저장되며, 해당 키스토어의 위치와 인증 정보를 구성 서버의 bootstrap.yml 파일에 명시해야 한다.

키스토어가 어플리케이션 내부에 위치할 경우 bootstrap.yml 구성 예제이다. 

```yaml
encrypt:
  key-store:
    alias: tacokey
    location: classpath:/keystore.jks
    password: 13tm31n
    secret: s3cr3t
```

해당 설정을 통해 구성 서버가 키스토어를 사용하도록 구성할 수 있다.

### 암호화된 데이터 설정

구성 서버는 /encrypt 엔드포인트를 제공한다. 암호화될 데이터를 갖는 POST 요청을 /encrypt 엔드포인트에 하면 된다.

```bash
$ curl localhost:8888/encrypt -d "비밀번호"
# 암호화된 비밀번호가 출력된다.
```

POST 요청이 제출된 후 암호화된 값을 응답으로 받는다. 이 값을 이용하여 구성 파일에 붙여넣기 하면 된다.

몽고 DB 비밀번호를 설정하는 예시이다.

```yaml
spring:
  data:
    mongodb:
      password: '{cipher}출력된 비밀번호 넣기'
```

비밀번호 앞의 ‘{cipher}’는 <span class="red">해당 값이 암호화된 값</span>이라는 것을 구성 서버에 알려주는 역할을 수행한다.

### 복호화 해제

구성 서버가 암호화된 속성의 값을 있는 그대로 제공하길 원한다면 다음과 같이 설정하면 된다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: http://localhost:10080/tacocloud/tacocloud-config
        encrypt:
          enabled: false
```

해당 속성을 사용하면 클라이언트에서 암호화된 속성 값을 받으므로 <span class="red">클라이언트가 복호화를 수행해야 한다.</span>

### Vault 시작하기

Vault는 보안 관리 도구로, **보안 정보를 자체적으로 처리**한다. Valut 명령어로 서버를 시작하고 보안 속성을 관리할 수 있다.<br/>
개발 모드로 Vault 서버를 시작하는 명령어이다.

```bash
$ vault server -dev -dev-root-token-id=roottoken
$ export VAULT_ADDR='http://127.0.0.1::8200'
$ vault status
$ vault secrets disable secret
$ vault secrets enable -path=secret kv
```

개발 모드는 보안이 되지 않는 Vault 런타임이므로, 프로덕션 설정에서 사용하면 안된다. 각각의 명령어에 대한 설명은 다음과 같다.

1. Vault 서버를 사용하기 위해서는 토큰을 제공해야 하는데, 여기서 설정한 **루트 토큰**은 <span class="ul">관리용 토큰</span>으로 <span class="red">보안 정보를 읽거나 쓰는 데도 사용할 수 있다.</span> 루트 토큰을 지정하지 않으면 Vault가 자동으로 생성한다.
2. Vault 서버 위치를 알 수 있도록 VAULT_ADDR 환경변수를 지정한 명령어이다.
3. 이전의 두 명령어가 제대로 수행되어 Vault 서버가 실행중인지 검사하는 명령어이다.<br/>
4, 5. 구성 서버와 연계되도록 구성 서버와 호환되는 secret라는 이름의 Vault 백엔드를 생성한다.

### 보안 데이터를 Vault에 쓰기

몽고DB의 비밀번호를 Vault 서버에 저장하고 싶을 때는 다음과 같이 사용한다.

```bash
$ vault write secret/application spring.data.mongodb.password=p3cr3t
```

보안데이터 경로 앞의 secret/는 Vault 백엔드 서버를 나타내며, application은 보안 데이터의 경로를 나타낸다.

저장된 구성 데이터를 확인하려면 다음과 같이 사용한다.

```bash
$ vault read secret/application
key                                    value
---                                    -----
refresh_interval                       768h
spring.data.mongodb.password           s3cr3t
```

지정된 경로에 보안 데이터를 쓸 경우 이전에 썼던 데이터를 **덮어쓰기** 한다. 

```bash
$ vault write secret/application spring.data.mongodb.password=p3cr3t \
    spring.data.mongodb.username=tacocloud
```

하나의 속성 지정 후 다른 속성을 지정한다면 데이터가 사라지기 때문에 이와 같이 <span class="red">두 속성 모두를 같이 써야 한다.</span>

### 구성 서버에서 Vault 백엔드 활성화

구성 서버에서 Vault 서버를 추가할 때는 활성화 프로파일로 Vault를 추가한다.

```yaml
spring:
  profiles:
    active:
      - vault
      - git
```

이 경우 보안에 민감한 구성 속성은 Vault에만 쓰고 그렇지 않은 구성 속성은 git을 사용하면 된다.

구성 서버에서 Vault 기본값을 변경하는 코드이다.

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: http://localhost:10080/tococloud/tacocloud-config
          order: 2
        vault:
          host: vault.tacocloud.com
          port: 8200
          scheme: https
          order: 1
```

spring.cloud.config.server.vault 속성은 구성 서버에서 Vault에 대한 기본값을 변경할 수 있게 한다. <span class="red">order 속성은 Vault의 속성이 git의 속성보다 우선한다는 것을 나타낸다.</span> 기본적으로 구성 서버는 Vault 서버가 localhost의 8200 포트를 리스닝한다고 간주한다.

Vault 서버에 요청을 보낼 때는 다음과 같이 사용한다.

```bash
$ curl localhost:8888/application/default -H "X-config-Token: roottoken" | jq
```

Vault에 대한 모든 요청은 X-Config-Token 헤더를 포함해야 한다. 구성 서버 자체에 해당 토큰을 구성하는 대신, **구성 서버 클라이언트가 구성 서버에 대한 모든 요청의 X-Config-Token 헤더에 토큰을 포함시켜야 한다.**<br/>
<span class="red">이때는 토큰을 사용하지 않는 Git 속성일 경우에도 토큰이 누락되면 구성 서버에서 속성 제공을 거부한다.</span>

### 구성 서버 클라이언트에 Vault 토큰 생성

구성 서버 클라이언트 로컬 구성에 Vault 토큰을 설정할 수 있다.

```yaml
spring:
  cloud:
    config:
      token: roottoken
```

지정한 토큰 값을 구성 서버의 모든 요청에 포함하라고 구성 서버 클라이언트에게 알려준다.

### 특정 서비스에 보안 속성 지정

application 경로의 보안 속성은 이름과 상관없이 모든 어플리케이션에 제공한다. 만일 특정한 어플리케이션에 보안 속성을 저장해야 한다면 application 대신 해당 어플리케이션 이름으로 교체하면 된다.

ingredient-service인 어플리케이션에 특정된 보안 속성을 쓰는 예시이다.

```bash
$ vault write secret/ingredient-service spring.data.mongodb.password=p3cr3t
```

특정 프로파일에 관련된 보안 속성을 쓰는 예시이다.

```bash
$ vault write secret/application,production spring.data.mongodb.password=p3cr3t
```

활성 프로파일이 production인 어플리케이션에만 제공하는 보안 속성을 쓰는 명령어이다.

## 구성 속성 리프레시

---

> 클라우드 기반의 어플리케이션에서는 어플리케이션을 중단시키지 않고 실시간으로 구성 속성을 변경할 수 있어야 한다. 구성 속성을 리프레시하는 방법에는 두 가지 방법이 있다.
> 
> - 수동식 : 클라이언트가 **Actuator 엔드포인트를 활성화**하여 해당 엔드포인트로 POST 요청 후 최근 구성을 가져오는 방식
> - 자동식 : **레포지터리의 커밋 후크(commit hook)** 가 모든 서비스의 리프레시를 촉발하는 방식

수동식 리프레시는 서비스가 리프레시되는 시점을 정확히 제어할 수 있는 반면 마이크로서비스에 대해 개별적인 HTTP 요청이 수행되어야 한다.
자동식 리프레시는 어플리케이션의 모든 마이크로서비스에 대해 즉시 변경 구성을 적용하는 반면, 구성 레포지터리에 커밋을 할 때 수행되므로 프로젝트에 따라 부담이 될 수 있다.

### 수동식 리프레시

스프링부트 액추에이터는 런타임 파악 및 런타임 상태의 일부 제한적인 제어를 가능하게 한다. 어플리케이션이 구성 서버의 클라이언트로 활성화되면 자동-구성이 액추에이터 엔드포인트를 구성한다.

엔드포인트를 사용하려면 액추에이터 스타터 의존성을 추가해야 한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

스프링 Initializr 화면의 ‘액추에이터’ 체크박스를 선택해도 추가된다.
<br/>

*액추에이터가 활성화되면 /actuator/refresh에 대한 HTTP POST 요청을 보내면 언제든 구성 속성을 리프레시할 수 있다.*

```bash
$ curl localhost:8080/actuator/refresh -X POST
["config.client.version", "greeting.message"]   # 리프레시한 속성 정보가 응답 데이터로 반환
```

### 자동식 리프레시

모든 클라이언트에게 자동으로 구성 변경을 알려주려면 스프링 클라우드 버스를 사용한다.

{{<figure src="/images/spring-cloud-config/2.png" class="large" caption="스프링 클라우드 버스를 함께 사용하여 구성 서버는 각 어플리케이션에 변경 사항을 전파한다.">}}

1. 웹훅이 Git 레포지터리에 대한 변경이 생겼음을 구성 서버에 알려준다.
2. 구성 서버는 RabbitMQ나 카프카같은 메시지 브로커를 통해 변경 관련 메시지를 전파한다.
3. 알림을 구독하는 구성 서버 클라이언트는 새로운 속성 값으로 리프레시한다.
<br/><br/>


**구성 서버에서 웹 훅 처리하기**

웹 훅을 처리하려면 구성 서버의 /monitor 엔드포인트를 활성화해야 한다. 이를 위해서는 스프링 클라우드 구성 모니터링 의존성을 구성 서버 프로젝트 빌드에 추가하면 된다.

```groovy
implementation 'org.springframework.cloud:spring-cloud-config-monitor'
// rabbitMQ 사용시
implementation 'org.springframework.cloud:spring-starter-stream-rabbit'
// 카프카 사용시
implementation 'org.springframework.cloud:spring-starter-stream-kafka'
```

구성 서버가 변경 알림을 전파하는 수단도 가져야 하므로 스프링 클라우드 스트림 의존성도 추가해준다. 스프링 클라우드 스트림을 이용하여 RabbitMQ나 카프카를 통해 통신하는 서비스들을 생성할 수 있다. 
<br/><br/>

메시지 브로커의 포트나 인증 정보 등을 변경했다면 구성 서버 속성을 설정해야 한다.

**RabbitMQ의 경우**

```yaml
spring:
  rabbitmq:
    host: rabbit.tacocloud.com
    port: 5672
    username: tacocloud
    password: s3cr3t
```

**kafka의 경우**

```yaml
spring:
  kafka:
    bootstrap-servers:
      - kafka.tacocloud.com:9092
      - kafka.tacocloud.com:9093
      - kafka.tacocloud.com:9094
```

이와 같이 속성을 변경해주면 된다.

### 구성 서버 클라이언트에서 자동 리프레시 활성화하기

구성 서버 클라이언트에 속성을 자동 리프레시하려면 스프링 클라우드 버스 스타터를 빌드에 추가해야 한다.

```groovy
// rabbitMQ 사용시
implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
// kafka 사용시
implementation 'org.springframework.cloud:spring-cloud-starter-bus-kafka'
```

스프링 클라우드 버스 스타터가 추가되면 어플리케이션이 시작될 때 자동-구성이 수행되어 rabbitMQ나 카프카에 자동으로 바인딩된다.