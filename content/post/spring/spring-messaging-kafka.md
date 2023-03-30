+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 비동기 메시지 전송하기 - Kafka"
author = "lhj8390"
description = "스프링에서 Kafka를 사용하여 비동기 메시지를 전송하는 방법에 대해 설명한다."
date = "2022-08-11"
tags = ["spring", "web", "java", "spring boot", "kafka"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
## 개요

---

> 스프링인액션 8장을 읽고 스프링이 제공하는 비동기 메시징을 공부하였다.
> 
> - Kafka 구조 및 설정
> - Kafka 이용한 메시지 전송
> - Kafka 이용한 메시지 수신
> 
> 에 대해 알 수 있다.
> 

## 카프카 사용

---

> 아파치 카프카는 가장 새로운 메시징 시스템이다.<br/>
> 카프카는 높은 확장성을 제공하는 클러스터로 실행되도록 설계되었는데, 클러스터의 모든 카프카 인스턴스에 걸쳐 토픽을 파티션으로 분할하여 메시지를 관리한다.
> 
> → RabbitMQ가 거래소와 큐를 사용해서 메시지를 처리하는 반면, 카프카는 토픽만 사용한다.
> 

### 카프카의 구조

{{<figure src="/images/spring-messaging-kafka/1.png" class="large" caption="각 브로커는 토픽의 파티션의 리더로 동작">}}

카프카의 토픽은 클러스터의 모든 브로커에 걸쳐 복제되는 형태이다.<br/>
클러스터의 각 노드는 하나 이상의 토픽에 대한 리더(leader)로 동작하며, 토픽 데이터를 관리하고 클러스터의 다른 노드로 데이터를 복제한다.<br/>
각 토픽은 여러 개의 파티션으로 분할될 수 있다.

### 카프카 설정

카프카를 프로젝트 빌드에 추가한다.<br/>
JMS나 RabbitMQ와 달리 카프카는 스프링 부트 스타터가 없다.

```groovy
implementation 'org.springframework.kafka:spring-kafka:2.1.8.RELEASE'
```

이를 통해 KafkaTemplate을 사용할 수 있다.

### 실무환경에서의 카프카 설정

카프카는 기본적으로 localhost의 9092 포트를 사용한다. 로컬에서 실행되는 개발 환경일 경우 로컬의 카프카 브로커를 사용하면 좋다.<br/>
하지만 실무 환경에서는 카프카의 호스트나 포트를 application.yaml 파일에 구성해야 한다.

**카프카 서버 설정**

```yaml
spring:
  kafka:
    bootstrap-servers:
      - kafka.tacocloud.com:9092
      - kafka.tacocloud.com:9092
      - kafka.tacocloud.com:9092
```

`spring.kafka.bootstrap-servers` 속성을 통해 카프카 클러스터로의 초기 연결에 사용되는 하나 이상의 카프카 서버 위치를 설정할 수 있다. 이 속성은 복수형이며, 여러 서버를 지정할 수 있다.

## KafkaTemplate을 사용하여 메시지 전송

---

> KafkaTemplate 메서드들은 JMS나 RabbitMQ의 메서드들과 유사하지만 다른 부분도 존재한다.
> 
> 1. **convertAndSend() 메서드 존재 여부**<br/>
>     KafkaTemplate은 제너릭 타입을 사용하고 메시지를 전송할 때 직접 도메인 타입을 처리할 수 있기 때문에 convertAndSend() 메서드가 없다.
>     
> 2. **send()와 sendDefault() 메서드의 매개변수**<br/>
>     카프카에서 메시지를 전송할 때는 (1) <span class="ul">메시지가 전송될 토픽</span>, (2) <span class="ul">토픽 데이터를 쓰는 파티션</span>,  (3) <span class="ul">레코드 전송 키</span>, (4) <span class="ul">타임스탬프</span>(기본값 System.currentTimeMillis()), (5) <span class="ul">페이로드를 매개변수</span>로 지정할 수 있다.
>
>여기서 필수값은 **메시지가 전송될 토픽과 페이로드**이다.
>     

### send() 메서드로 메시지 전송

```java
private KafkaTemplate<String, Order> kafkaTemplate;

public void sendOrder(Order order) {
  kafkaTemplate.send("tacocloud.orders.topic", order);
}
```

send() 메서드를 이용하여 "tacocloud.orders.topic"이라는 이름의 토픽으로 Order 객체를 전송한다.

### sendDefault() 메서드로 메시지 전송

```java
private KafkaTemplate<String, Order> kafkaTemplate;

public void sendOrder(Order order) {
  kafkaTemplate.sendDefault(order);
}
```

**기본 토픽**으로 Order 객체를 전송하고 싶으면 토픽 이름을 인자로 전달하지 않고 send() 메소드 대신 sendDefault() 메서드를 사용한다.<br/><br/>

**기본 토픽 설정**

```yaml
spring:
  kafka:
    template:
      default-topic: tacocloud.orders.topic
```

기본 토픽은 `spring.kafka.template.default-topic` 속성을 이용하면 된다.

## 카프카 리스너 작성

---

> KafkaTemplate는 메시지를 수신하는 메서드를 일체 제공하지 않는다.<br/>
> 따라서 스프링을 사용하여 카프카 토픽의 메시지를 가져오는 유일한 방법은 메시지 리스너를 작성하는 것이다.
> 

### 리스너를 이용하여 메시지 수신

```java
private KitchenUI ui;

@KafkaListener(topics="tacocloud.orders.topic")
public void handle(Order order) {
  ui.displayOrder(order);
}
```

@KafkaListener 어노테이션을 지정하여 "tacocloud.orders.topic" 이름의 토픽에 메시지가 도착할 때 자동 호출되도록 설정하였다. 

### 추가적인 메타데이터가 필요한 경우

ConsumerRecord나 Message 객체를 인자로 받아 메시지의 메타데이터를 받을 수 있다. 
<br/><br/>

**ConsumerRecord 사용**

```java
private KitchenUI ui;

@KafkaListener(topics="tacocloud.orders.topic")
public void handle(Order order, ConsumerRecord<String, Order> record) {
  log.info("파티션 : {}, timestamp : {}", record.partition(), record.timestamp());
  
  ui.displayOrder(order);
}
```

ConsumerRecord를 통해 수신된 메시지의 파티션과 타입스탬프를 로깅하는 코드이다.
<br/><br/>

**Message 객체 사용**

```java
private KitchenUI ui;

@KafkaListener(topics="tacocloud.orders.topic")
public void handle(Order order, Message<Order> message) {
  MessageHeaders headers = message.getHeaders();
  log.info("Received from partition {} with timestamp {}",
      headers.get(KafkaHeaders.RECEIVED_PARTITION_ID),
      headers.get(KafkaHeaders.RECEIVED_TIMESTAMP));
  ui.displayOrder(order);
}
```

Message 객체를 통해 메시지의 파티션과 타입스탬프를 로깅하는 코드이다.
<br/><br/>

<aside>
<strong>메시지 페이로드도 ConsumerRecord나 Message 객체를 통해 받을 수 있다!</strong>  

	- ConsumerRecord.value()
	-  Message.getPayload()
</aside>