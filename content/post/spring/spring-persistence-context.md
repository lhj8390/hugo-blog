+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "영속성 컨텍스트(persistence context)의 개념"
author = "lhj8390"
description = "JPA에서 영속성 컨텍스트(persistence context)의 개념을 정리한다."
date = "2023-01-15"
tags = ["spring", "web", "java", "spring boot", 'JPA', 'persistence context']
categories = ["Framework"]
subcategories = ["Spring"]
+++

## EntityManagerFactory와 EntityManager

---

> EntityManagerFactory는 고객의 요청이 올 때마다 (스레드가 하나 생성될 때마다) EntityManager를 생성한다.  EntityManager는 내부적으로 DB connection pool을 사용해서 DB에 접근한다.
> 

EntityManagerFactory는 애플리케이션이 로딩되는 시점에 데이터베이스 당 딱 하나만 생성한다.<br/>
EntityManager는 실제 트랜잭션 단위가 수행될 때마다 생성되고, 클라이언트의 요청이 들어올 때 생성했다가 요청이 끝나면 닫는다. → <span class="red">스레드 간에 공유를 하면 안된다.</span>

## 영속성 컨텍스트

---

> JPA를 이해하는데 가장 중요한 용어 - “*<span class="gray">엔티티를 영구 저장하는 환경*</span>”<br/>
> 영속성 컨텍스트는 논리적인 개념으로 엔티티 매니저를 통해 영속성 컨텍스트에 접근 가능하다.
> 

엔티티 매니저로 Entity를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티리를 보관하고 관리한다.

```java
EntityManager.persist(entity);
```

persist() 메소드는 엔티티 매니저를 사용해서 Entity를 영속성 컨텍스트 안에 저장한다.

## 엔티티의 생성주기
{{<figure src="/images/spring-persistence-context/1.png" class="small">}}
- 비영속 (new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속 (managed) : 영속성 컨텍스트에 관리되는 상태
- 준영속 (detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제 (removed) : 삭제된 상태

### 비영속

> 순수한 객체 상태이며 아직 저장하지 않은 상태 → 영속성 컨텍스트나 데이터베이스와는 관련 없음.
> 

```java
// 객체를 실행한 상태
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

### 영속

> 영속성 컨텍스트에 의해 관리되는 상태.
> 

```java
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
//객체를 저장한 상태(영속)
em.persist(member);
```

### 준영속, 삭제

```java
//회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
em.detach(member);

//객체를 삭제한 상태(삭제)
em.remove(member);
```

**준영속** : 영속 상태의 엔티티가 영속성 컨텍스트에서 분리

- 영속성 컨텍스트가 제공하는 기능을 사용하지 못한다.
- 식별자 값을 가지고 있다. - 한번 영속 상태였으므로
- 지연 로딩을 할 수 없다.

**삭제** : 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제

**준영속 상태로 만드는 방법**

- `em.detach(entity)` : 특정 엔티티만 준영속 상태로 전환
- `em.clear()` : 영속성 컨텍스트를 완전 초기화
- `em.close()` : 영속성 컨텍스트를 종

## 영속성 컨텍스트의 특징
---
- 식별자 값 : 영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다. (식별자 값이 없으면 예외 발생)
- 데이터베이스 저장 : 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 저장된 엔티티를 데이터베이스에 반영

## 영속성 컨텍스트의 이점
---
- 1차 캐시에서 조회 : 영속 컨텍스트에서 1차 캐시에서 데이터를 찾은 후 없으면 DB에서 조회한다.
- 영속 엔티티의 동일성(identity) 보장<br/>
    : 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 DB가 아닌 애플리케이션 차원에서 제공 
    
    ```java
    Member a = em.find(Member.class, "member1");
    Member b = em.find(Member.class, "member1");
    System.out.println(a == b); //동일성 비교 true
    ```
- 엔티티를 등록할 때 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)<br/>
    : 커밋을 하는 순간에 INSERT가 가능, 버퍼링 기능
    ```java
    EntityManager em = emf.createEntityManager();
    EntityTransaction transaction = em.getTransaction();
    //엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다.
    transaction.begin(); // [트랜잭션] 시작
    em.persist(memberA);
    em.persist(memberB);
    //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.
    //커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다.
    transaction.commit(); // [트랜잭션] 커밋
    ```
- 엔티티를 수정할 때 변경 감지(Dirty Checking)<br/>
    : commit 시 엔티티와 스냅샷(값을 읽어온 최초 시점)을 비교하여 UPDATE 쿼리를 생성한다. (JPA의 기본 전략은 엔티티의 모든 필드를 업데이트한다.)
- 지연 로딩(Lazy Loading)

## flush
---
> 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화한다.<br/>
> ⚠️ flush를 실행한다고 해서 영속성 컨텍스트 내용을 지우지 않는다.<br/>
> 트랜잭션이라는 작업 단위가 중요 - 커밋 직전에 동기화하면 된다.
> 

영속성 컨텍스트를 flush하려면 세가지 방법이 있다.

1. em.flush() : 직접 호출 (거의 사용하지 않음)
2. 트랜잭션 commit : flush 자동 호출 - 트랜잭션만 commit하면 데이터베이스에 반영되지 않기 때문에 JPA는 자동으로 flush를 호출된다.
3. JPQL 쿼리 실행 : flush 자동 호출