+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 비동기 메시지 전송하기 - RabbitMQ"
author = "lhj8390"
description = "스프링에서 RabbitMQ를 사용하여 비동기 메시지를 전송하는 방법에 대해 설명한다."
date = "2022-08-07"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

## 개요

---

> 스프링인액션 8장을 읽고 RabbitMQ와 AMQP를 사용한 비동기 메시징을 공부하였다.
> 
> - RabbitMQ의 기본 개념 및 설정
> - RabbitMQ를 이용한 메시지 전송
> - RabbitMQ를 이용한 메시지 수신
> 
> 에 대해 알 수 있다.
> 



## RabbitMQ 사용

---

> RabbitMQ는 JMS보다 진보된 메시지 라우팅 전략을 제공한다.<br/>
> AMQP(Advenced Message Queing Protocol)을 따르는 오픈소스 메시지 브로커 중 하나이다.
> 

### RabbitMQ 구조


{{<figure src="/images/spring-messaging-rabbitmq/1.png" class="large" caption="거래소와 큐 간의 관계">}}
<br/><br/>


RabbitMQ 거래소로 전송되는 메시지는 라우팅 키와 바인딩을 기반으로 하나 이상의 큐에 전달된다.

| 거래소 타입 | 설명 |
| --- | --- |
| 기본(Default) | 브로커가 자동으로 생성. 라우팅 키와 이름이 같은 큐로 메시지를 전달한다. 모든 큐는 자동으로 기본 거래소와 연결된다. |
| 다이렉트(Direct) | 바인딩 키가 해당 메시지의 라우팅 키와 같은 큐에 메시지를 전달한다. |
| 토픽(Topic) | 바인딩 키가 해당 메시지의 라우팅 키와 일치하는 하나 이상의 큐에 메시지를 전달한다. |
| 팬아웃(Fanout) | 바인딩 키나 라우팅 키에 상관없이 모든 연결된 큐에 메시지를 전달한다. |
| 헤더(Header) | 토픽 거래소와 유사하며, 라우팅 키 대신 메시지 헤더 값을 기반으로 한다. |
| 데드 레터(Dead Letter) | 전달 불가능한(거래소, 큐 바인딩과 일치하지 않는) 모든 메시지를 보관하는 거래소이다. |

### RabbitMQ 설정

스프링 부트의 AMQP를 프로젝트 빌드에 추가한다.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-amqp'
```

AMQP 연결 팩토리와 RabbitTemplate 빈을 자동으로 구성해준다.
<br/>


### 실무환경에서의 RabbitMQ 설정

스프링은 RabbitMQ 브로커가 localhost의 5672 포트를 리스닝하고 인증 정보가 필요없기 때문에 로컬에서 실행되는 개발 환경일 경우 설정값을 따로 지정해 줄 필요가 없다.<br/>
하지만 실무 환경에서는 브로커를 어떻게 사용하는지 application.yml에 속성 값을 지정해 줄 필요가 있다.
<br/><br/>

**ActiveMQ 브로커의 위치와 인증 정보를 구성하는 속성**

| 속성 | 설명 |
| --- | --- |
| spring.rabbitmq.addresses | 쉼표로 구분된 리스트 형태의 RabbitMQ 브로커 주소 |
| spring.rabbitmq.host | 브로커의 호스트(기본값 localhost) |
| spring.rabbitmq.port | 브로커의 포트(기본값 5672) |
| spring.rabbitmq.username | 브로커를 사용하기 위한 사용자(선택) |
| spring.rabbitmq.password | 브로커를 사용하기 위한 암호(선택) |

## RabbitTemplate을 사용하여 메시지 전송

---

> RabbitTemplate는 JmsTemplate과 유사한 메서드를 제공하지만 미세한 차이가 존재한다.
> 
> - JmsTemplate : 지정된 큐나 토픽에만 메시지 전송
> - RabbitTemplate : **거래소와 라우팅 키의 형태**로 메시지 전송

<br/>

RabbitTemplate는 다음과 같은 메서드들을 가지고 있다.

- **send()** : 원시 메시지 전송
- **convertAndSend()** : 객체로부터 변환된 메시지 전송하거나 인자에 MessagePostProcessor를 포함하여 전송에 앞서 후처리되는 메시지 전송 가능

<aside>
Destination 객체 대신 거래소와 라우팅 키를 지정하는 문자열 값을 인자로 받는다!
라우팅 키를 인자로 받지 않을 경우 기본 라우팅 키로 전송되는 메시지를 가진다.<br/>

</aside>
<br/>

### send() 메서드로 메시지 전송

```java
private RabbitTemplate rabbit;

public void sendOrder(Order order) {
    MessageConverter converter = rabbit.getMessageConverter();
    MessageProperties props = new MessageProperties();
    Message message = converter.toMessage(order, props);
    rabbit.send("tacocloud.order", message);
}
```

MessageConverter로 Order 객체를 Message 객체로 변환한다.<br/>
**메시지 속성 제공** : MessageProperties - <span class="gray">*메시지 속성 설정하지 않을 경우에는 기본 인스턴스 전달*</span>
<br/><br/>

**거래소 기본값 설정**

```yaml
spring:
    rabbitmq:
        template:
            exchange: tacocloud.orders
            routing-key: kitchens.central
```

기본 거래소 이름은 빈 문자열인 “”이며, RabbitMQ 브로커가 자동으로 생성하는 기본 거래소이다.<br/>
기본 라우팅 키도 거래소와 동일하게 “”이다.

기본값을 변경할 경우 해당 yaml 파일처럼 지정해준다. (exchange, routing-key 속성)

### 메시지 변환기 구성

| 메시지 변환기 | 하는 일 |
| --- | --- |
| Jackson2JsonMessageConverter | Jackson2JSONProcessor를 사용해서 객체를 JSON으로 상호 변환한다. |
| MarshallingMessageConverter | 스프링 Marshaller와 Unmarshaller를 사용해서 변환한다. |
| SerializerMessageConverter | 스프링의 Serializer와 Deserializer를 사용해서 String과 객체를 변환한다. |
| SimpleMessageConverter | String, byte 배열, Serializable 타입을 변환한다. |
| ContentTypeDelegatingMessageConverter | contentType 헤더를 기반으로 다른 메시지 변환기에 변환을 위임한다. |

기본적으로는 SimpleMessageConverter가 사용되며, 메시지 변환기를 변경해야 할 경우에는 MessageConverter 타입의 빈을 구성하면 된다.
<br/><br/>

**메시지 변환기 변경 예제**

```java
@Bean
public MessageConverter messageConverter() {
    return new Jackson2JsonMessageConverter();
}
```

### 메시지 속성 설정

**send() 메서드에서 속성 설정**

```java
public void sendOrder(Order order) {
    MessageConverter converter = rabbit.getMessageConverter();
    MessageProperties props = new MessageProperties();
    props.setHeader("X_ORDER_SOURCE", "WEB");
    Message message = converter.toMessage(order, props);
    rabbit.send("tacocloud.order", message);
}
```

MessageProperties 인스턴스를 통해 커스텀 헤더를 설정할 수 있다.
<br/><br/>

**convertAndSend() 메서드에서 속성 설정**

```java
public void sendOrder(Order order) {
  rabbit.convertAndSend("tacocloud.order.queue", order,
          message -> {
            MessageProperties props = message.getMessageProperties();
            props.setHeader("X_ORDER_SOURCE", "WEB");
            return message;
          });
}
```

convertAndSend() 메서드에서는 MessagePostProcessor를 사용하여 구현한다.<br/>
Message 객체의 MessageProperties를 가져온 후 setHeader()를 호출하여 헤더를 설정한다.

## RabbitMQ로부터 메시지 수신

---

> RabbitMQ 큐로부터의 메시지 수신도 JMS과 다르지 않다.
> 
> - 풀 모델(pull model) 기반 : RabbitTemplate를 통해 큐로부터 메시지 수신
> - 푸시 모델(push model) 기반 : @RabbitListener가 지정된 메서드로 메시지 푸시

<br/>


### RabbitTemplate을 사용하여 메시지 수신

RabbitTemplate는 다음과 같은 메서드들을 가지고 있다.

- receive() : 원시 메시지 수신
- receiveAndConvert() : 메시지를 수신한 후 메시지 변환기를 사용하여 도메인 객체로 변환하고 반환

<aside>
<strong>JmsTemplate와의 차이점</strong><br/>
1. 수신 메서드는 거래소나 라우팅 키를 갖지 않는다. 
  → 어플리케이션은 큐만 알면 되기 때문에 <span class="red">(거래소나 라우팅 키는 메시지를 큐로 전달할 때만 사용)</span>
2. 메시지의 수신 타임아웃을 나타내기 위해 long 타입의 매개변수를 가진다.
   타임아웃 값을 인자로 전달하면 메시지가 도착하거나 타임아웃에 걸릴 때까지 메서드는 대기하게 된다. (기본값 0)<br/>
</aside>
<br/><br/>

**receive() 사용**

```java
private RabbitTemplate rabbit;
private MessageConverter converter;

public Order receiveOrder() {
    Message message = rabbit.receive("tacocloud.orders", 30000);
    return message != null ? (Order) converter.fromMessage(message) : null;
}
```

타입아웃 값인 30초가 지났을 경우에 Message 객체 혹은 null이 반환된다. (30초간 대기)<br/>
Message 객체 반환되었을 경우 : MessageConverter를 사용하여 Message 객체를 Order 객체로 변환
<br/><br/>

**구성을 통해 타임아웃 값 지정**

```yaml
spring:
    rabbitmq:
        template:
            receive-timeout: 30000
```

receive() 메서드의 타임아웃 값을 제거하고 구성 파일을 통해 타임아웃 값을 설정할 수 있다.
<br/><br/>

**receiveAndConvert() 사용**

```java
private RabbitTemplate rabbit;

public Order receiveOrder() {
    return (Order) rabbit.receiveAndConvert("tacocloud.order.queue");
}
```

메시지 변환기 대신 receiveAndConvert() 메서드를 이용하면 손쉽게 메시지 변환이 가능하다.

**캐스팅 대신 특정 객체 수신**

```java
public Order receiveOrder() {
    return (Order) rabbit.receiveAndConvert("tacocloud.order.queue", 
	new ParameterizedTypeReference<Order>() {});
}
```

ParameterizedTypeReference를 인자로 전달하여 직접 Order 객체를 수신하게 할 수 있다.<br/>
<span class="red">※ 메시지 변환기가 SmartMessageConverter 인터페이스를 구현한 클래스이어야 한다.</span>

### 리스너 사용

스프링은 RabbitListener라는 RabbitMQ 리스너를 제공한다. <br/>
메시지가 큐에 도착할 때 메서드가 자동 호출되도록 하기 위해서는 @RabbitListener 어노테이션을 RabbitMQ 빈의 메서드에 지정해야 한다.
<br/><br/>

**리스너 선언**

```java
private KitchenUI ui;

@RabbitListener(queues = "tacocloud.order.queue")
public void receiveOrder(Order order) {
	ui.displayOrder(order);
}
```

리스너 선언은 어노테이션을 RabbitListener로 지정했다는 것 이외에는 JmsListener와 동일하다.

