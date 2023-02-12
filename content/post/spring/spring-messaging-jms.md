+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 비동기 메시지 전송하기 - JMS"
author = "lhj8390"
description = "스프링에서 JMS를 사용하여 비동기 메시지를 전송하는 방법에 대해 설명한다."
date = "2022-08-05"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

## 개요

---

> 스프링인액션 8장을 읽고 JMS를 이용하여 스프링이 제공하는 비동기 메시징을 공부하였다.
> 
> - JMS 기본 개념 및 설정
> - JMS를 이용한 메시지 전송
> - JMS를 이용한 메시지 수신
> 
> 에 대해 작성해보았다.
> 

## JMS로 메시지 전송하기

---

> JMS는 "**두 개 이상의 클라이언트 간에 메시지 통신을 위한 공통 API를 정의하는 자바 표준**"이다.<br/>
> 스프링에서는 JmsTemplate이라는 템플릿 기반의 클래스를 통해 JMS를 지원한다. 
> 
> JmsTemplate를 사용하여 프로듀서(producer)가 큐와 토픽에 메시지를 전송하고 컨슈머(<span class="gray">consumer</span>)는 그 메시지를 받을 수 있다.
> 
<br/>

### JMS 설정

JMS 클라이언트를 프로젝트 빌드에 추가한다. <br/>
하지만 우선 메시지 브로커를 선택해야 하는데, 아파치 ActiveMQ 또는 ActiveMQ Artemis(아르테미스) 가 있다.

**ActiveMQ 사용 시**

```groovy
implementation 'org.springframework.boot:spring-boot-starter-activemq'
```
<br/>

**ActiveMQ Artemis 사용 시**

```groovy
implementation 'org.springframework.boot:spring-boot-starter-artemis'
```

Artemis는 ActiveMQ를 새롭게 다시 구현한 차세대 브로커이다. 
<br/><br/>

### 실무환경에서의 브로커 설정

스프링은 브로커가 localhost의 61616 포트를 리스닝하는 것으로 간주하기 때문에 로컬에서 실행되는 개발 환경일 경우 설정값을 따로 지정해 줄 필요가 없다.<br/>
하지만 실무 환경에서는 브로커를 어떻게 사용하는지 application.yml에 속성 값을 지정해 줄 필요가 있다.
<br/><br/>

**ActiveMQ 브로커의 위치와 인증 정보를 구성하는 속성**

| 속성 | 설명 |
| --- | --- |
| spring.activemq.broker-url | 브로커의 URL |
| spring.activemq.user | 브로커를 사용하기 위한 사용자(선택) |
| spring.activemq.password | 브로커를 사용하기 위한 암호(선택) |
| spring.activemq.in-memory | 인메모리 브로커로 시작할 것인지의 여부(기본값 true) |

```yaml
spring:
  activemq:
    broker-url: tcp://activemq.tacocloud.com
    user: user
    password: admin123
```

URL은 YAML 파일에 지정한 것처럼 <span class="ul">tcp://URL 형태</span>로 지정해야 한다.
<br/><br/>

**Artemis 브로커의 위치와 인증 정보를 구성하는 속성**

| 속성 | 설명 |
| --- | --- |
| spring.artemis.host | 브로커의 호스트 |
| spring.artemis.port | 브로커의 포트 |
| spring.artemis.user | 브로커를 사용하기 위한 사용자(선택) |
| spring.artemis.password | 브로커를 사용하기 위한 암호(선택) |

```yaml
spring:
  artemis:
    host: artemis.tacocloud.com
    port: 61617
    user: user
    password: admin123
```

artemis.tacocloud.com의 61617 포트를 리스닝하는 브로커에 대한 연결을 생성하기 위해 스프링을 설정하였다.

## JmsTemplate을 사용하여 메시지 전송

---

Artemis 또는 ActiveMQ가 의존성에 추가되면 스프링 부트가 메시지를 송수신하기 위해 주입 및 사용할 수 있는 JmsTemplate을 자동-구성한다.
JmsTemplate는 다음과 같은 메서드들을 가지고 있다.
- **send()** : 원시 메시지 전송
- **convertAndSend()** : 객체로부터 변환된 메시지 전송하거나 인자에 MessagePostProcessor를 포함하여 전송에 앞서 후처리되는 메시지 전송 가능

### 원시 메시지 전송

```java
private JmsTemplate jms;

@Override
public void sendOrder(Order order) {
  jms.send(new MessageCreator() {
    @Override
    public Message createMessage(Session session) throws JMSException {
      return session.createObjectMessage(order);
    }
  });
}
```

여기서 send() 메서드는 

1. Message 객체를 생성하기 위해 MessageCreator를 필요로 한다.
2. 도착지 매개변수가 없어 해당 메시지를 기본 도착지로 전송한다.
<br/><br/>

MessageCreator 인터페이스의 **createMessage(Session) 함수를 오버라이딩하여** 전달된 Order 객체로부터 새로운 메시지를 생성한다.

<aside>
💡 람다식으로 적용하면 다음과 같다.

```java
@Override
public void sendOrder(Order order) {
  jms.send(session -> session.createObjectMessage(order));
}
```

<span class="red">※ MessageCreator는 함수형 인터페이스이므로 람다식으로 표현할 수 있다.</span>

</aside>
<br/><br/>

메시지의 도착지를 지정하지 않았기 때문에 기본 도착지를 application.yml 파일에 지정해야 한다.

```yaml
spring:
  jms:
    template:
      default-destination: tacocloud.order.queue
```

### 메시지 도착지 지정

Destination 객체를 통해 메시지 도착지를 지정하거나, 메시지 도착지를 나타내는 문자열(String 타입)을 통해 도착지를 지정할 수 있다.
<br/><br/>

#### 1. **Destination 객체 활용**
```java
@Bean
public Destination orderQueue(){
    return new ActiveMQQueue("tacocloud.order.queue");
}
```

Destination 빈을 선언하고 메시지 전송을 수행하는 빈에 주입한다.<br/><br/>

```java
private JmsTemplate jms;
private Destination orderQueue;

@Override
public void sendOrder(Order order) {
    jms.send(orderQueue, session -> session.createObjectMessage(order));
}
```

여기서 send() 메서드는 

1. 첫번째 매개변수로 Destination 객체를 전달한다.
2. Message 객체를 생성하기 위해 MessageCreator를 활용한다.
<br/><br/>

#### 2. **문자열 활용**

```java
private JmsTemplate jms;

@Override
public void sendOrder(Order order) {
    jms.send("tacocloud.order.queue", session -> session.createObjectMessage(order));
}
```

여기서 send() 메서드는 

1. 첫번째 매개변수로 목적지 이름을 String 타입으로 전달한다.
2. Message 객체를 생성하기 위해 MessageCreator를 활용한다.
<br/><br/>

### 메시지 변환하고 전송

convertAndSend() 메서드는 MessageCreator를 제공하지 않고 전송될 객체를 convertAndSend()의 인자로 직접 전달하면 해당 객체가 Message 객체로 변환되어 전송된다.
<br/><br/>

**기본 예제**

```java
@Override
public void sendOrder(Order order) {
    jms.convertAndSend("tacocloud.order.queue", order);
}
```

send() 메서드처럼 convertAndSend()는 Destination 객체나 문자열 값으로 지정한 도착지를 인자로 받거나, 도착지를 생략하여 기본 도착지로 메시지를 전송할 수 있다.
<br/><br/>

**메시지 변환기 구현**

```java
public interface MessageConverter {
    Message toMessage(Object object, Session session) throw JMSException, MessageConversionException;
    Object fromMessage(Message message);
}
```

메시지 변환기를 구현하기 위해서는 스프링에 정의된 인터페이스인 MessageConverter를 사용한다.<br/><br/>

📢 **변환작업을 해주는 스프링 메시지 변환기**

| 메시지 변환기 | 하는 일 |
| --- | --- |
| MappingJackson2MessageConverter | Jackson 2 JSON 라이브러리를 사용해서 메시지를 JSON으로 상호 변환한다. |
| MarshallingMessageConverter | JAXB를 사용해서 메시지를 XML로 상호 변환한다. |
| MessagingMessageConverter | 수신된 메시지의 MessageConverter를 사용해서 해당 메시지를 Message 객체로 상호 변환한다. 또는 JMS 헤더와 연관된 JmsHeaderMapper를 표준 메시지 헤더로 상호 변환한다. |
| SimpleMessageConverter | 문자열을 TextMessage로, byte 배열을 BytesMessage로, Map를 MapMessage로, Serializable 객체를 ObjectMessage로 상호 변환한다. |

상단의 메시지 변환기는 모두 org.springframework.jms.support.converter 패키지에 존재한다.<br/>
기본적으로는 **SimpleMessageConverter**가 사용되며, 이 경우 전송될 객체가 Serializable 인터페이스를 구현한다.
<br/><br/>

**다른 메시지 변환기 사용**

```java
@Bean
public MappingJackson2MessageConverter messageConverter() {
    MappingJackson2MessageConverter messageConverter = new MappingJackson2MessageConverter();
    messageConverter.setTypeIdPropertyName("_typeId");
    return messageConverter;
}
```

다른 메시지 변환기를 적용할 때는 해당 변환기의 인스턴스를 빈으로 선언한다. <br/>
setTypeIdPropertyName() 메서드에는 <span class="red">매핑된 ID 값 또는 Java 클래스 이름</span>을 포함한 type id를 지정할 수 있다. 이 속성을 설정해야 수신 메시지를 Java 객체로 변환시킬 수 있다.
<br/><br/>

**임의의 타입 이름 매핑**

```java
@Bean
public MappingJackson2MessageConverter messageConverter() {
    MappingJackson2MessageConverter messageConverter = new MappingJackson2MessageConverter();
    messageConverter.setTypeIdPropertyName("_typeId");
  
    Map<String, Class<?>> typeIdMappings = new HashMap<String, Class<?>>();
    typeIdMappings.put("order", Order.class);
    messageConverter.setTypeIdMappings(typeIdMappings);
  
    return messageConverter;
}
```

setTypeIdPropertyName() 메서드 만을 이용하여 타입을 지정할 경우 메시지 수신자도 똑같은 클래스와 타입을 가져야 하기 때문에 유연성이 떨어진다.<br/>
하지만 setTypeIdMappings() 메서드를 이용한다면 메시지의 _typeId 속성에 전송되는 클래스 이름 대신 **실제 타입에 임의의 타입 이름을 매핑시킬 수 있다.** <span class="gray it">- 이 코드에서는 Order 클래스를 order라는 타입 ID로 매핑하도록 메시지 변환기를 변경한다.</span>
<br/><br/>

### 후처리 메시지

**send()를 이용한 후처리**

```java
jms.send("tacocloud.order.queue",
        session -> {
            Message message = session.createObjectMessage(order);
            message.setStringProperty("X_ORDER_SOURCE", "WEB");
        });
```

order 객체에 새로운 source 속성을 추가하고 싶다. <br/>
이 경우 Message 객체의 setStringProperty() 메서드를 통해 커스텀 헤더를 메시지에 추가한다.<br/><br/>

**convertAndSend()를 이용한 후처리**

```java
jms.convertAndSend("tacocloud.order.queue", order,
        message -> {
            message.setStringProperty("X_ORDER_SOURCE", "WEB");
            return message;
        });
```

convertAndSend() 메서드를 이용할 경우 마지막 인자로 MessagePostProcessor를 전달하면 내부적으로 생성된 Message 객체에 후처리를 할 수 있다.
<br/><br/>

**MessagePostProcessor 메서드 참조**

```java
public String convertAndSendOrder() {
    Order order = buildOrder();
    jms.convertAndSend("tacocloud.order.queue", order, this::addOrderSource);
    return "Convert and send order";
}

private Message addOrderSource(Message message) throws JMSException {
    message.setStringProperty("X_ORDER_SOURCE", "WEB");
    return message;
}
```

동일한 MessagePostProcessor를 사용하려면 메서드 참조를 통해 구현할 수 있다.

## JMS 메시지 수신

---

> 메시지를 수신하는 방식은 두가지가 존재하는데,
> 
> - 메시지를 요청하고 도착할 때까지 기다리는 **pull model(풀 모델)**
> - 메시지가 수신 가능한 상태라면 코드로 자동 전달하는 **push model(푸시 모델)**
> 
> 이 있다.
> 
> JmsTemplate는 메시지를 수신하는 모든 메서드가 풀 모델을 사용한다. <br/>
> → *메시지를 요청하면 스레드에서 메시지를 수신할 수 있을 때까지 기다린다.*
> 
> 푸시 모델을 사용할 경우에는 메시지 리스너를 정의해야 한다.
> 
<br/>

### JmsTemplate 사용하여 메시지 수신

JmsTemplate는 다음과 같은 메서드들을 가지고 있다.
- receive() : 원시 메시지 수신
- receiveAndConvert() : 메시지를 도메인 타입으로 변환하기 위해 구성된 메시지 변환기 사용
<br/><br/>

**receive()  사용**

```java
private JmsTemplate jms;
private MessageConverter converter;

public Order receiveOrder() {
    Message message = jms.receive("tacocloud.order.queue");
    return (Order) converter.fromMessage(message);
}
```

도착지는 문자열 타입(String)을 이용하여 지정한다. <span class="gray it">- send()와 같이 Destination 객체를 이용할 수도 있다.</span><br/>
메시지 내부의 Order 객체를 받아오기 위해 메시지 변환기를 이용하여 수신 메시지를 Order 객체로 변환하였다.
<br/><br/>

**receiveAndConvert() 사용**

```java
private JmsTemplate jms;

public Order receiveOrder() {
    return (Order) jms.receiveAndConvert("tacocloud.order.queue");
}
```

메시지 변환기를 이용하지 않고 receiveAndConvert() 메서드를 사용하면 훨씬 간단하게 메시지 변환을 수행할 수 있다.<br/><br/>

### 메시지 리스너 선언

receive()나 receiveAndConvert()를 호출하는 풀 모델과 달리, 메시지 리스너는 메시지가 도착할 때까지 기다리는 수동적 컴포넌트이다.

**메시지 리스너 생성**

```java
private KitchenUI ui;

@JmsListener(destination = "tacocloud.order.queue")
public void receiveOrder(Order order) {
    ui.displayOrder(order);
}
```

"tacocloud.order.queue" 도착지의 메시지를 읽기 위해 **@JmsListener** 어노테이션을 지정한다. <br/>
메시지가 도착한다면 Order 객체가 인자로 전달되면서 receiveOrder() 메서드가 자동 호출된다.
