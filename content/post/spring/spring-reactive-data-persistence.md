+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 리액티브 데이터 퍼시스턴스"
author = "lhj8390"
description = "스프링에서 Reactive Repository를 생성하는 방법에 대해 설명한다."
date = "2022-08-24"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Framework"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
## 개요

---

> 스프링인액션 12장을 읽고 Reactive Repository를 생성하는 방법에 대해 공부하였다.
> - Reactive Repository의 개념
> - 카산드라와 몽고DB 사용하기
> - 기본 Repository를 리액티브 사용에 맞추어 조정하기
> - 카산드라를 활용한 데이터 모델링
> 
> 에 대해 작성하였다.
> 

## Reactive Repository

---

> Reactive Repository는 도메인 타입이나 컬렉션을 반환하지 않고 Mono나 Flux를 인자로 받거나 반환한다. 카산드라, 몽고DB, 카우치베이스, 레디스로 데이터를 저장할 경우 리액티브 프로그래밍 모델을 지원한다.
> 

### 기존 Repository에서의 변환

스프링 데이터 JPA를 사용할 경우 Mono나 Flux 리액티브 타입을 반환하지 않기 때문에 오퍼레이션을 수행할 수 없게 된다. <br/>
이를 해결하기 위해서는 다음과 같이 사용한다.

```java
List<Order> orders = repo.findByUser(user);
Flux<Order> orderFlux = Flux.fromIterable(orders);
```

Flux의 fromIterable(), fromArray(), fromStream() 메서드를 통해 Flux로 변환시킬 수 있다.

Mono일 경우는 다음과 같다.

```java
Order order = repo.findById(id);
Mono<Order> orderMono = Mono.just(order);
```

Mono의 just() 메서드를 통해 Mono로 변환이 가능하다.<br/><br/>

**Mono에서의 데이터 추출**

```java
Taco taco = tacoMono.block();
tacoRepo.save(taco);
```

Mono의 block() 메서드를 호출해서 데이터를 추출할 수 있다.<br/><br/>

**Flux에서의 데이터 추출**

```java
Iterable<Taco> tacos = tacoFlux.toIterable();
tacoRepo.saveAll(tacos);
```

Flux의 toIterable() 메서드를 통해 Flux의 모든 객체를 Iterable 타입으로 추출한다.

<aside>
Mono의 block() 메서드나 Flux의 toIterable() 메서드는 <strong><span class="red">추출 작업 시 블로킹 처리 된다.</span></strong> <br/>
이를 해결하기 위해서는 구독 발행 요소 각각에 오퍼레이션을 수행하는 것이 나은 방법이다.

```java
tacoFlux.subscribe(taco -> {
  tacoRepo.save(taco);
});
```

<span class="gray">※ JPA repository의 save() 메서드를 제외하고는 리액티브하다.</span>

</aside>

## 리액티브 카산드라 사용

---

> 카산드라는 분산처리, 고성능, 상시 가용, 궁극적인 일관성을 가지는 NoSQL 데이터베이스이다.
> 
> 
> 카산드라의 특징은 다음과 같다.
> 
> 1. 데이터베이스는 **다수의 파티션에 걸쳐 분할**된다.
> 2.  테이블은 **파티션 키와 클러스터링 키**라는 두 종류의 키를 가지고, 많은 column을 가질 수 있다. 
> 3. 읽기 오퍼레이션에 최적화 되어 있어 **데이터가 다수의 테이블에 중복되는 경우**가 흔하다.

### 카산드라 의존성 추가

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-cassandra-reactive'
```

리액티브한 카산드라 repository를 작성할 것이므로 해당 스타터 dependency를 추가한다.<br/>
스프링 Initializr에서 지정할 경우 NoSQL의 **“Spring Data Reactive for Apache Cassandra”** 를 선택한다.

### 카산드라 설정 변경

```yaml
spring:
  data:
    cassandra:
      keyspace-name: tacocloud
      schema-action: recreate-drop-unused
      contact-points:
      - casshost-1.tacocloud.com
      - casshost-2.tacocloud.com
      - casshost-3.tacocloud.com
      port: 9043
      username: tacocloud
      password: taco123
```

- `keyspace-name` : **카산드라가 해당 키 공간을 사용하도록 알린다.**<br/>
    스프링 데이터 카산드라에서 키 공간을 자동으로 생성하도록 설정할 수 있지만 직접 생성할 수도 있다. 키 공간은 **CQL**(*Cassandra Query Language*) 셸에서 `create keyspace` 명령어를 통해 생성할 수 있다.
    
    <aside>
      💡 키 공간 : 카산드라 노드의 테이블을 모아 놓은 것이다. 관계형 데이터베이스의 <strong>스키마와 유사하다.</strong>
    </aside>
    
- `schema-action` : **스키마의 행동을 지정한다.**<br/>
    **recreate-drop-unused** : 어플리케이션이 시작할 때마다 모든 테이블과 사용자 정의 타입을 재생성하도록 지정한다. (개발용)<br/>
    **none** : 스키마에 대해 아무 조치를 취하지 않는다. (실무용)
- `contact-points` : **카산드라 노드가 실행 중인 호스트를 나타낸다.** <br/>
    각 노드의 연결을 시도하여 카산드라 클러스터에 단일 장애점이 생기지 않게 해준다.
- `port` : **카산드라의 포트 번호를 설정할 수 있다.**
- `username`, `password` : **카산드라 클러스터의 사용자 이름과 비밀번호를 설정한다.**

### 카산드라 도메인 타입 매핑

카산드라 퍼시스턴스는 도메인 타입 매핑 어노테이션을 제공한다.

```java
@Data
@RestResource(rel = "tacos", path = "tacos")
@Table("tacos")
public class Taco {

  @PrimaryKeyColumn(type=PrimaryKeyType.PARTITIONED)
  private UUID id = UUIDs.timeBased();
  
  @NotNull
  @Size(min = 5, message = "Name must be at least 5 characters long")
  private String name;
  
  @PrimaryKeyColumn(type=PrimaryKeyType.CLUSTERED,
                    ordering=Ordering.DESCENDING)
  private Date createdAt = new Date();
  
  @Size(min=1, message="You must choose at least 1 ingredient")
  @Column("ingredients")
  private List<IngredientUDT> ingredients;

}
```

- **@Table** : 테이블의 이름 지정 <span class="gray"> **-** *JPA에서는 @Entity*</span>
- **@PrimaryKeyColumn** : 기본 키 지정<br/>
    PrimaryKeyType.PARTITIONED : 해당 속성이 파티션 키임을 명시<br/>
    PrimaryKeyType.CLUSTERED : 해당 속성이 클러스터링 키임을 명시
- **@Column** : 데이터의 컬렉션을 포함하는 열<br/>
    여기서는 **네이티브 타입(정수, 문자열 등)의 컬렉션**이거나, **UDT(사용자 정의 타입)의 컬렉션**이어야 한다.
    
    
<aside>
  💡 <strong>Ingredient 클래스를 재사용할 수 없는 이유</strong><br/>
  해당 클래스에는 이미 @Table 어노테이션을 통해 카산드라의 도메인 타입으로 매핑을 했기 때문이다. <br/>
  <span class="red">UDT 클래스를 새로 만들어 데이터가 어떻게 저장되는지 정의해야 한다.</span><br/>
</aside>   <br/><br/>

**UDT 클래스 예시**

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
@UserDefinedType("ingredient")
public class IngredientUDT {
  private final String name;
  private final Ingredient.Type type;
}
```

- @UserDefinedType : 카산드라의 UDT(사용자 정의 타입)임을 명시
- id 속성을 별도로 포함하지 않는다.

### 카산드라 Repository 작성

Repository를 작성할 때는 두 개의 기본 인터페이스인 ReactiveCassandraRepository 나 ReactiveCrudRepository를 선택할 수 있다. ReactiveCassandraRepository가 ReactiveCrudRepository의 확장된 버전이다.

**기본 예시**

```java
public interface IngredientRepository extends ReactiveCrudRepository<Ingredient, String> {
}
```

이와 같이 ReactiveCrudRepository를 extends 처리하면 된다. findAll() 메서드 사용 시 Flux<Ingredient>를 반환한다. Flux를 반환하기 때문에 페이징 처리도 신경 쓸 필요가 없다. - <span class="gray">*take() 메서드 활용 가능*</span>

**where 절**

```java
public interface UserRepository extends ReactiveCassandraRepository<User, UUID> {

  @AllowFiltering
  Mono<User> findByUsername(String username);
}
```

카산드라에서는 where 절로 쿼리할 수 없다. where 절을 사용하기 위해서는 @AllowFiltering 어노테이션을 사용한다. <span class="red">allow filtering 절은 <span class="ul">쿼리 성능에 영향을 주지만 수행해야 한다</span>라는 것을 카산드라에게 알려준다.</span>

## 리액티브 몽고DB 사용

---

> 몽고DB는 잘 알려진 NoSQL 데이터베이스 중 하나이다. 카산드라가 테이블의 행으로 데이터를 저장하는 반면, 몽고DB는 **문서형 데이터베이스**이다.
> 

### 몽고DB 의존성 추가

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-mongodb-reactive'
```

리액티브한 몽고DB repository를 작성할 것이므로 해당 스타터 dependency를 추가한다.<br/>
스프링 Initializr에서 지정할 경우 NoSQL의 **“Spring Data Reactive MongoDB”** 를 선택한다.

### 몽고DB 설정 변경

```yaml
spring:
  data:
    mongodb:
      host: mongodb.tacocloud.com
      port: 27018
      username: tacocloud
      password: taco123
```

- `host` : 몽고DB 서버가 실행 중인 호스트 이름. 기본값 - localhost
- `port` : 몽고DB 서버가 리스닝하는 포트. 기본값 - 27017
- `username`, `password` : 몽고 DB에 접근하는 사용자 이름과 비밀번호
- `database` : 데이터베이스 이름. 기본값 - test

### 도메인 타입 매핑

스프링 데이터 몽고DB는 문서 구조로 도메인 타입을 매핑하는 데 유용한 어노테이션을 제공한다.

```java
@Data
@RestResource(rel="tacos", path="tacos")
@Document
public class Taco {
  @Id
  private String id;

  @NotNull
  @Size(min = 5, message = "Name must be at least 5 characters long")
  private String name;
  
  private Date createdAt = new Date();
  
  @Size(min=1, message="You must choose at least 1 ingredient")
  private List<Ingredient> ingredients;

  @Field("customer")
  private User user;
}
```

- **@Id** : 문서 ID로 지정
    - String 타입으로 지정할 경우 속성 값이 DB에 저장될 때 자동으로 ID 값을 지정해준다.
- **@Document** : 몽고 DB에 저장되는 문서로 선언
- **@Field** : 몽고DB의 문서에 속성을 저장하기 위해 필드 이름을 지정
    - customer 열을 문서에 저장한다는 것을 의미
- 카산드라와 다르게 UDT(사용자 정의 타입)을 만들지 않고 어떤 타입이라도 사용 가능하다.

### 몽고DB Repository 작성

Repository를 작성할 때는 두 개의 기본 인터페이스인 ReactiveMongoRepository 나 ReactiveCrudRepository를 선택할 수 있다. ReactiveCrudRepository가 새로운 문서나 기존 문서의 save() 메서드에 의존하는 반면, ReactiveMongoRepository는 **새로운 문서의 저장에 최적화된 insert() 메서드**를 제공한다.

**기본 예시**

```java
public interface IngredientRepository 
         extends ReactiveCrudRepository<Ingredient, String> {
}
```

ReactiveCrudRepository를 이용하여 Repository를 작성할 경우 몽고DB와 카산드라 모두 똑같은 인터페이스를 활용할 수 있다. 

**where 절**

```java
public interface UserRepository extends ReactiveMongoRepository<User, String> {

  Mono<User> findByUsername(String username);
}
```

몽고DB는 커스텀 쿼리 메서드의 명명 규칙을 따른다.