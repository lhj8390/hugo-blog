+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 스프링 초기 설정"
author = "lhj8390"
description = "스프링부트 초기의 gradle 빌드 명세, 부트스트랩(구동) 클래스, DevTools에 대해 설명한다."
date = "2022-06-05"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Framework"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
folder = "spring"
+++

## 개요

---

> 스프링인액션 1장을 읽고 스프링 부트 어플리케이션을 구축하기 위한 초기 설정 등을 기록하였다.
> 
> 이 문서를 통해
> - 초기의 gradle 빌드 명세 설명
> - 부트스트랩(구동) 클래스 설명
> - 스프링부트 DevTools
> 
> 에 대해 알 수 있다.
> 

## 초기의 gradle 빌드 명세

---

> Spring Initalizr를 이용하여 초기 스프링 프로젝트를 생성하면 (gradle으로 빌드)<br/>
> 해당 build.gradle 파일이 생긴다.
> 
> *Spring Initalizr에서 Spring Web, Thymeleaf, Spring Boot DevTools, Lombok을 선택.*
> 

```groovy
plugins {                                                                                                                          
    id 'org.springframework.boot' version '2.7.0'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'tacos'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'       
    implementation 'org.springframework.boot:spring-boot-starter-web'                
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'           
}
tasks.named('test') {
    useJUnitPlatform()
}
```
<br/>

### 플러그인

```groovy
plugins {
    id 'org.springframework.boot' version '2.7.0'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}
```

plugins 영역의 3개 플러그인 → 자바와 스프링 부트를 사용하기 위한 필수 플러그인

id ‘org.springframework.boot’ : 스프링 부트 프로젝트를 실행할 수 있게 만들어주는 스프링 부트 관련 플러그인 <br/>
id ‘io.spring.dependency-management’ : 스프링 부트의 의존성들을 관리해 주는 플러그인<br/>
id 'java’ : Java 프로그램을 위한 기능을 제공하는 플러그인 - *compileJava, classes, test 등의 task 제공*
<br/><br/>


### 저장소

```groovy
repositories {
    mavenCentral()
}
```

온라인으로 접속하여 사용할 수 있는 저장소로 Gradle은 대체로 다음 2개의 저장소 서비스를 이용한다.

- 메이븐 중앙 저장소 추가를 위한 `mavenCentral()` 메소드
- Bintray의 jCenter 저장소 추가를 위한 `jcenter()` 메소드를 제공

<br/>

### **의존 라이브러리**

```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])           // 1
    implementation 'org.springframework.boot:spring-boot-starter'      // 2
}
```

의존관계를 설정하는 부분이다. 빌드 스크립트에서 직접 의존성을 지정한다.

1️⃣  패키지화되어 있는 라이브러리 모듈의 의존을 정의하는 방법이다. 
[프로젝트]/libs/" 폴더 내의 JAR 파일들과의 의존성을 선언하였다.

2️⃣  라이브러리가 Local에 존재하지 않는다면 정의된 원격 저장소(ex: Maven repository, Nexus 등)에서 라이브러리를 다운받아 빌드할 수 있는 환경을 제공한다.
<br/><br/>


### 스프링 부트 스타터 의존성

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'       
    implementation 'org.springframework.boot:spring-boot-starter-web'               
    testImplementation 'org.springframework.boot:spring-boot-starter-test'            
}
```

Spring Web, Thymeleaf, Test 의존성 항목은 starter 단어를 포함하고 있는데, 이는 스프링 부트 스타터 의존성을 나타낸다.

스타터 의존성의 장점은,

1. 필요한 모든 라이브러리의 의존성을 선언하지 않아도 되어 빌드 파일이 더 작아지고 관리의 이점.
2. 기능의 관점 : 웹 어플리케이션을 개발한다면 일일이 라이브러리를 지정하는 대신 웹 스타터 의존성만 추가하면 된다.
3. 라이브러리의 버전 호환성 보장

## 어플리케이션 구동

---

> 어플리케이션의 구동을 위해서는 부트스트랩(구동) 클래스가 있어야 한다.<br/>
> 또한 최소한의 스프링 구성도 있어야 한다.
> 

```java
@SpringBootApplication
public class TacoCloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(TacoCloudApplication.class, args);
    }
}
```
<br/>

### **@SpringBootApplication** 이란?
**@SpringBootApplication은 다음 세 개의 어노테이션이 결합한 것.**

- @SpringBootConfiguration : 현재 클래스를 구성 클래스로 지정.
- @EnableAutoConfiguration : 스프링 자동-구성 활성화. - *필요로 하는 컴포넌트를 자동으로 구성하도록 스프링부트에게 알려준다.*
- @ComponentScan : 컴포넌트 검색 활성화. - *@Component, @Controller, @Service 등의 어노테이션과 함께 클래스를 선언할 수 있도록 도와줌.*
<br/><br/>

### main()

실제로 어플리케이션을 시작시키고 스프링 어플리케이션 컨텍스트를 생성하는 SpringApplication 클래스의 run() 메서드 호출.<br/>
run() 메서드의 두 개의 매개변수는 구성 클래스와 명령행 인자.

## 스프링부트 DevTools

---

> 스프링 부트 DevTools는 개발 시점의 편리한 도구를 제공한다.
> 
> - 코드가 변경될 때, 자동으로 어플리케이션을 재시작시킨다.
> - 리소스(템플릿, 자바스크립트, 스타일시트)가 변경될 때 자동으로 브라우저를 새로고침한다.
> - 탬플릿 캐시를 자동으로 비활성화한다.
> - H2 콘솔을 활성화시킨다.
<br/><br/>

**어플리케이션 재시작**

자바 코드, 속성 파일, 프로젝트의 src/main/ 경로에 있는 모든 것들은 DevTools가 다시 재시작시킨다.

다만, 의존성 라이브러리를 포함하는 클래스 로더는 자동으로 다시 로드되지 않는다.

→ 빌드 명세(gradle 등)에 의존성을 추가, 변경, 삭제할 때는 어플리케이션을 재시작해야만 적용된다.
<br/><br/>

**템플릿 캐시 비활성화**

Thymeleaf나 freeMarker 같은 템플릿은 모든 웹 요청마다 매번 파싱하지 않기 위해 파싱 결과를 캐시에 저장한다. (성능의 이점)

하지만 템플릿 캐싱은 브라우저를 새로고침하더라도 이전의 캐시를 사용하기 때문에 변경된 효과를 볼 수 없다.

DevTools는 템플릿 캐싱을 비활성화하여 브라우저를 새로고침하면 변경된 템플릿이 적용된다.

<aside>
💡 <strong>LiveReload</strong> 플러그인을 사용한다면 템플릿, 이미지, 자바스크립트 등 변경이 생길 때 브라우저가 자동으로 새로고침된다.<br/>
</aside>
<br/>

### Intellij 적용 방법

> 준비 사항 : build.gradle에 org.springframework.boot:spring-boot-devtools 라이브러리 추가 필수!
> 

1. application.properties 파일에 spring.thymeleaf.cache = **false** 추가

2. [Edit Configurations] 구성 편집 창에서 [Modify options] 설정 변경
    1. 상단 구성을 클릭하여 [Edit Configurations] 클릭
        {{< figure src="/images/spring-basic/1.png" alt="image" class="left" >}}
    <br/>
    <br/>
    <br/>

    2. 구성 편집 창에서 [Modify options] 클릭
        
        {{< figure src="/images/spring-basic/2.png" alt="image" class="large" >}}
        
    3. **On ‘Update’ action 에서 Update classess and resources 선택 (On frame deactivation도 동일 선택)**
        
        {{< figure src="/images/spring-basic/3.png" alt="image" class="large" >}}
        
        이러한 설정을 통해 DevTools를 적용시킬 수 있다.
        <br/><br/>