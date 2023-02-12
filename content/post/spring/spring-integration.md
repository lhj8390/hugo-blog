+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 스프링 통합 플로우 사용"
author = "lhj8390"
description = "스프링에서 통합 패턴을 사용하는 방법에 대해 설명한다."
date = "2022-08-12"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

## 개요

---

> 스프링인액션 9장을 읽고 통합 패턴을 사용하는 법을 공부하였다.
> 
> - 실시간으로 데이터 처리
> - 통합 플로우 정의
> - 자바 DSL 정의 사용
> - 다른 외부 시스템과 통합
> 
> 에 대해 작성하였다.
> 

## 통합 플로우 선언

---

> 어플리케이션은 통합 플로우를 통해 **외부 리소스나 어플리케이션 자체에 데이터를 수신/전송**할 수 있다.<br/>
> 스프링 통합 중 “채널 어뎁터”는 파일을 읽거나 쓰는 컴포넌트이다.
> 

### 스프링 통합 빌드 명세

```groovy
implementation 'org.springframework.boot:spring-boot-starter-integration'
implementation 'org.springframework.integration:spring-integration-file'
```

스프링 통합 플로우 개발을 위해 스프링 부트 스타터인 스프링 통합을 추가해준다.<br/>
또한 파일 시스템으로부터 통합 플로우로 파일을 읽거나, 데이터를 쓰기 위해 스프링 통합 파일 엔드포인트 모듈도 추가해준다.

### 게이트웨이 생성

파일에 데이터를 쓸 수 있도록 어플리케이션에서 통합 플로우로 데이터를 전송하는 게이트웨이를 생성한다.<br/>

**메서드 호출을 메시지로 변환**

```java
@MessagingGateway(defaultRequestChannel="textInChannel")
public interface FileWriterGateway {
    void writeToFile(@Header(FileHeaders.FILENAME) String filename,String data);
}
```

- @MessagingGateway : 해당 인터페이스의 구현체를 <span class="red">런타임 시 생성</span>하라고 스프링에게 전달
- defaultRequestChannel : 메서드 호출로 생성된 메시지가 이 속성의 채널로 <span class="red">전송</span>

**writeToFile() 메서드의 인자**
- @Header : filename에 전달되는 값이 메시지 헤더에 있다는 것을 명시
- data : 해당 매개변수 값은 메시지 페이로드로 전달

### 통합 플로우 구성

통합 플로우 구성에는 3가지 방법이 존재한다.

1. XML 구성
2. 자바 구성
3. DSL을 이용한 자바 구성
<br/><br/>

**통합 플로우 구성 예시**

{{<figure src="/images/spring-integration/1.png" class="large" caption="파일-쓰기 통합 플로우">}}
<br/><br/>

**자바 구성**

```java
@Configuration
public class FileWriterIntegrationConfig {

  @Bean
  @Transformer(inputChannel="textInChannel", outputChannel="fileWriterChannel")
  public GenericTransformer<String, String> upperCaseTransformer() {
      return text -> text.toUpperCase();
  }

  @Bean
  @ServiceActivator(inputChannel="fileWriterChannel")
  public FileWritingMessageHandler fileWriter() {
      FileWritingMessageHandler handler = new FileWritingMessageHandler(new File("/tmp/sia5/files"));
      handler.setExpectReply(false);
      handler.setFileExistsMode(FileExistsMode.APPEND);
      handler.setAppendNewLine(true);
      return handler;
  }
}
```

**GenericTransformer (변환기 역할)** : @Transformer을 통해 textInChannel의 메시지를 받아서 fileWirterChannel로 쓰는 <span class="red">통합 플로우 변환기</span>라는 것을 명시한다.

**FileWritingMessageHandler (파일 아웃바운드 채널 어뎁터 역할)** : @ServiceActivator을 통해 fileWriterChannel로부터 메시지를 받아 FileWritingMessageHandler의 인스턴스로 정의된 서비스에 넘겨 메시지 페이로드를 지정된 디렉터리의 파일에 쓴다.

<aside>
💡 채널들을 별도로 선언하지 않았는데, textInChannel과 fileWriterChannel 이름의 빈이 없으면 채널이 <span class="ul">자동으로 생성</span>되기 때문이다.<br/>

</aside>
<br/><br/>

**DSL을 이용한 자바 구성**

```java
@Configuration
public class FileWriterIntegrationConfig {

  @Bean
  public IntegrationFlow fileWriterFlow() {
    return IntegrationFlows
        .from(MessageChannels.direct("textInChannel"))        // 입력 채널
        .<String, String>transform(t -> t.toUpperCase())      // 변환기 선언
        .channel(MessageChannels.direct("fileWriterChannel")) // 파일-쓰기 채널
        .handle(Files                                         // 아웃바운드 채널 어뎁터
            .outboundAdapter(new File("/tmp/sia5/files"))
            .fileExistsMode(FileExistsMode.APPEND)
            .appendNewLine(true))
        .get(); 
  }

}
```

통합 플로우의 각 컴포넌트를 별도의 빈으로 선언하지 않고 **전체 플로우를 하나의 빈으로 선언**한다.

**구조**

1. textInChannel이라는 이름의 채널로부터 메시지 수신
2. 메시지 페이로드를 대문자로 바꾸는 변환기 실행
3. 변환기의 메시지를 아웃바운드 채널 어댑터로 전달
4. 변환된 아웃바운드 채널 어댑터에서 변환된 메시지 처리
5. get() 메서드를 호출하여 IntegrationFlow 인스턴스를 가져온다.

DSL 구성 또한 채널을 별도로 구성하지 않는다면 채널이 자동으로 생성된다.

## 스프링 통합 컴포넌트

---
### 컴포넌트 역할

- **채널** : 한 요소로부터 다른 요소로 메시지 전달
- **필터** : 조건에 맞는 메시지만 플로우 통과
- **변환기** : 메시지 값을 변경하거나 메시지 페이로드의 타입 변환
- **라우터** : 여러 채널 중 하나로 메시지 전달 (주로 메시지 헤더 기반)
- **분배기** : 들어오는 메시지를 두 개 이상의 메시지로 분할
- **집적기** : 별개의 채널로부터 전달되는 다수의 메시지를 하나로 결합
- **서비스 액티베이터** : 자바 메서드에 메시지를 넘긴 후 반환값을 출력 채널로 전송
- **채널 어댑터** : 외부 시스템에 채널 연결. 외부 시스템으로부터 입력을 받거나 쓸 수 있다.
- **게이트웨이** : 인터페이스를 통해 통합 플로우로 데이터 전달.

### 메시지 채널

통합 파이프라인을 통해 메시지가 이동하는 수단. 

→ ***“통합 플로우의 서로 다른 컴포넌트 간에 데이터를 전달하는 통로”***

**스프링 통합이 제공하는 채널 구현체**

- PublishSubscribeChannel : 하나 이상의 컨슈머로 전달
- QueueChannel : FIFO 방식으로 컨슈머가 가져갈 때까지 큐에 저장
- PriorityChannel : QueueChannel와 비슷하지만, 메시지의 priority 헤더 기반으로 메시지 획득
- RendezvousChannel : QueueChannel와 비슷하지만, 컨슈머가 메시지를 수신할 때까지 전송자가 채널을 차단
- DirectChannel : PublishSubscribeChannel과 비슷하지만, 전송자와 동일한 스레드인 단일 컨슈머에게 메시지 전송 - <span class="gray">*트랜잭션 지원*</span>
- ExecutorChannel : DirectChannel과 비슷하지만, TaskExecutor를 통해 메시지 전송(다른 스레드) - <span class="gray">*트랜잭션 지원 X*</span>
- FluxMessageChannel : Flux를 기반으로 하는 Reactive Streams Publisher 채널

<span class="red">자바 구성과 DSL 구성 모두 입력 채널은 자동으로 생성되며, 기본적으로 DirectChannel이 사용된다.</span>

<aside>
💡 <strong>QueueChannel 사용시 주의사항</strong><br/>
컨슈머가 이 채널을 polling(도착 메시지가 있는지 지속적으로 확인)하도록 구성해야 한다.

```java
@ServiceActivator(inputChannel="orderChannel", poller=@Poller(fixedRate="1000"))
```

orderChannel로부터 1초당 한번씩 메시지가 있는지 확인하는 코드이다.<br/>
</aside>

### 필터

보통 통합 파이프라인 중간에 위치하며, 플로우의 전 단계로부터 다음 단계로의 메시지 전달을 허용하거나 불허한다.<br/><br/>

**자바 구성 예시**

```java
@Filter(inputChannel="numberChannel", outputChannel="evenNumberChannel")
public boolean evenNumberFilter(Integer number) {
    return number % 2 == 0;
}
```

@Filter 어노테이션을 통해 필터를 선언할 수 있다.<br/><br/>

**DSL 구성 예시**

```java
@Bean 
public IntegrationFlow evenNumberFlow(AtomicInteger integerSource) {
  return IntegrationFlows
      .<Integer>filter((p) -> p % 2 == 0)
      .get();
}
```
<br/>

### 변환기
메시지 값의 변경이나 타입을 변환하는 일을 수행한다.
<br/><br/>

**자바 구성 예시**

```java
@Bean
@Transformer(inputChannel="numberChannel", outputChannel="romanNumberChannel")
public GenericTransformer<Integer, String> romanNumTransformer() {
    return RomanNumbers::toRoman;
}
```

@Transformer 어노테이션을 통해 변환기 임을 선언한다.<br/>
numberChannel 채널로부터 Integer 값을 수신하고 toRoman()을 사용하여 변환을 수행한다.
<br/><br/>

**DSL 구성 예시**

```java
@Bean 
public IntegrationFlow transformerFlow() {
    return IntegrationFlows
        .transform(RomanNumbers::toRoman)
        .get();
}
```
<br/>

### 라우터

메시지에 적용된 전달 조건을 기반으로 서로 다른 채널로 메시지를 전달한다.
<br/><br/>

**자바 구성 예시**

```java
@Bean
@Router(inputChannel="numberChannel")
public AbstractMessageRouter evenOddRouter() {
  return new AbstractMessageRouter() {
    @Override
    protected Collection<MessageChannel> determineTargetChannels(Message<?> message) {
      Integer number = (Integer) message.getPayload();
      if (number % 2 == 0) {
        return Collections.singleton(evenChannel());
      }
      return Collections.singleton(oddChannel());
    }  
  };
}
```

numberChannel이름의 채널로부터 메시지를 받아 페이로드를 검사하여<br/>
짝수일 때는 <span class="red">evenChannel 채널을 반환</span>, 홀수일 때는 <span class="red">oddChannel 채널이 반환</span>된다.
<br/><br/>

**DSL 구성 예시**

```java
@Bean 
public IntegrationFlow numberRoutingFlow(AtomicInteger source) {
  return IntegrationFlows
      .<Integer, String>route(n -> n%2==0 ? "EVEN":"ODD", mapping -> mapping
        .subFlowMapping("EVEN", 
            sf -> sf.<Integer, Integer>transform(n -> n * 10)
                      .handle((i, h) -> { ... })
            )
        .subFlowMapping("ODD",
            sf -> sf.transform(RomanNumbers::toRoman)
                      .handle((i, h) -> { ... })
            )
        )
      .get();
}
```

람다식을 사용하여 짝수일 때는 “EVEN”, 홀수일 떄는 “ODD”값을 반환하고, 이 값들을 통해 메시지를 처리하는 하위 플로우를 결정한다.
<br/><br/>

### 분배기

통합 플로우에서 하나의 메시지를 여러개로 분할하여 독립적으로 처리한다.
<br/><br/>

**자바 구성 예시**

```java
@Bean
@Splitter(inputChannel="poChannel" outputChannel="splitOrderChannel")
public OrderSplitter orderSplitter() {
    return new OrderSplitter();
}

@Bean
@Router(inputChannel="splitOrderChannel")
public MessageRouter splitOrderRouter() {
    PayloadTypeRouter router = new PayloadTypeRouter();
    router.setChannelMapping(BillingInfo.class.getName(), "billingInfoChannel");
    router.setChannelMapping(List.class.getName(), "lineItemsChannel");
    return router;
}
```

poChannel 채널로 도착한 주문 메시지는 OrderSplitter로 분할된다.<br/>
분할된 메시지는 PayloadTypeRouter에 의해 **각 페이로드 타입을 기반으로 서로 다른 채널에 메시지를 전달한다.** BillingInfo 파입의 페이로드는 billingInfoChannel로, List 타입의 페이로드는 lineItemsChannel에 전달된다.

<aside>
💡 <strong>List 내부의 객체를 별도로 처리하고 싶은 경우</strong>

```java
@Splitter(inputChannel="lineItemsChannel" outputChannel="lineItemChannel")
public List<LineItem> lineItemSplitter(List<LineItem> lineItems) {
    return lineItems;
}
```

List<LineItem>을 다수의 메시지로 분할하기 위해 @Splitter과 함께 LineItem이 저장된 컬렉션을 반환하면 된다.

</aside>
<br/><br/>

**DSL 구성 예시**

```java
@Bean 
public IntegrationFlow numberRoutingFlow(AtomicInteger source) {
  return IntegrationFlows
      .split(orderSplitter())
      .<Object, String>route(
        p -> {
          if (p.getClass().isAssignableFrom(BillingInfo.class)) {
            return "BILLING_INFO";
          } else {
            return "LINE_ITEMS";
          }
        }, mapping -> mapping
        .subFlowMapping("BILLING_INFO", 
            sf -> sf.<BillingInfo> handle((billingInfo, h) -> { ... }))
        .subFlowMapping("LINE_ITEMS",
            sf -> sf.split().<LineItem> handle((lineItem, h) -> { ... }))
        )
      .get();
}
```
<br/>

### 서비스 엑티베이터

입력 채널로부터 메시지를 수신하고 이 메시지를 MessageHandler 인터페이스를 구현한 클래스(빈)에 전달한다.
<br/><br/>

**MessageHandler 빈 선언**

```java
@Bean
@ServiceActivator(inputChannel="someChannel")
public MessageHandler sysoutHandler() {
  return message -> {
    System.out.println("Message Payload: " + message.getPayload()); 
  };
}
```

someChannel 채널로부터 받은 메시지를 처리하는 서비스 액티베이터를 지정한다.<br/>
메시지의 페이로드를 표준 출력 시스템으로 내보낸다.
<br/><br/>

**새로운 페이로드로 반환 시**

```java
@Bean
@ServiceActivator(inputChannel="orderChannel", outputChannel="completeChannel")
public GenericHandler<Order> orderHandler(OrderRepository orderRepo) {
  return (payload, headers) -> {
    return orderRepo.save(payload);
  };
}
```

새로운 페이로드를 반환하려면 GenericHandler를 구현해야 한다.<br/>
Order 타입의 메시지 페이로드를 처리하는 GenericHandler를 구현하여 메시지가 도착하면 repository에 저장한다.
<br/><br/>

**MessageHandler 빈 선언(DSL)**

```java
@Bean 
public IntegrationFlow someFlow() {
  return IntegrationFlows
      .handle(msg -> {
        System.out.println("Message Payload: " + msg.getPayload());
      })
      .get();
}
```
<br/>

**새로운 페이로드로 반환 시(DSL)**

```java
@Bean 
public IntegrationFlow orderFlow(OrderRepository orderRepo) {
  return IntegrationFlows
      .<Order>handle((payload, headers) -> {
        return orderRepo.save(payload);
      })
      .get();
}
```

GenericHandler를 플로우 맨 끝에 사용할 경우에는 출력 채널이 없다는 에러가 발생하기 때문에 null을 반환해야 한다.

### 게이트웨이

어플리케이션이 통합 플로우로 데이터를 제출하고 선택적으로 플로우의 응답 결과를 받을 수 있다.
<br/><br/>

**자바 구성 예시**

```java
@Component
@MessageGateway(defaultRequestChannel="inChannel", 
                defaultReplyChannel="outChannel")
public interface UpperCaseGateway {
  String uppercase(String in);
}
```

스프링 통합이 런타임 시에 자동으로 구현체를 제공하기 때문에 인터페이스를 구현할 필요가 없다.
<br/><br/>

**DSL 구성 예시**

```java
@Bean 
public IntegrationFlow uppercaseFlow() {
  return IntegrationFlows
      .from("inChannel")
      .<String, String>transform(s -> s.toUpperCase())
      .channel("outChannel")
      .get();
}
```
<br/>

### 채널 어댑터

통합플로우의 입구와 출구를 나타낸다. 데이터는 인바운드 채널 어댑터를 통해 들어오고, 아웃바운드 채널 어댑터를 통해 통합 플로우에서 나간다.
<br/><br/>

**인바운드 채널 어댑터 구성**

```java
@Bean
@InboundChannelAdapter(poller=@Poller(fixedRate="1000"), channel="numberChannel")
public MessageSource<Integer> numberSource(AtomicInteger source) {
  return () -> {
    return new GenericMessage<>(source.getAndIncrement());
  };
}
```

@InboundChannelAdapter 어노테이션을 통해 인바운드 채널 어댑터 빈으로 선언된다.<br/>
AtomicInteger로부터 numberChannel 채널로 매초마다 한번씩 데이터를 전달한다.<br/><br/>

**인바운드 채널 어댑터 구성(DSL)**

```java
@Bean 
public IntegrationFlow someFlow(AtomicInteger integerSource) {
  return IntegrationFlows
      .from(integerSource, "getAndIncrement", 
          c -> c.poller(Pollers.fixedRate(1000)))
      .get();
}
```

DSL 구성에서는 from() 메서드를 통해 인바운드 채널 어댑터를 구성한다.

### 엔드포인트 모듈

스프링 통합은 커스텀 채널 어댑터를 생성할 수 있게 채널 어댑터가 포함된 엔드포인트 모듈을 제공한다.

| Module | Inbound Adapter | Outbound Adapter | Inbound Gateway | Outbound Gateway |
| --- | --- | --- | --- | --- |
| AMQP | Inbound Channel Adapter | Outbound Channel Adapter | Inbound Gateway | Outbound Gateway |
| Events | Receiving Spring Application Events | Sending Spring Application Events | N | N |
| Feed | Feed Inbound Channel Adapter | N | N | N |
| File | Reading Files and 'tail’ing Files | Writing files | N | Writing files |
| FTP(S) | FTP Inbound Channel Adapter | FTP Outbound Channel Adapter | N | FTP Outbound Gateway |
| Gemfire | Inbound Channel Adapter and Continuous Query Inbound Channel Adapter | Outbound Channel Adapter | N | N |
| HTTP | HTTP Namespace Support | HTTP Namespace Support | Http Inbound Components | HTTP Outbound Components |
| JDBC | Inbound Channel Adapter and Stored Procedure Inbound Channel Adapter | Outbound Channel Adapter and Stored Procedure Outbound Channel Adapter | N | Outbound Gateway and Stored Procedure Outbound Gateway |
| JMS | Inbound Channel Adapter and Message-driven Channel Adapter | Outbound Channel Adapter | Inbound Gateway | Outbound Gateway |
| JMX | Notification-listening Channel Adapter and Attribute-polling Channel Adapter and Tree-polling Channel Adapter | Notification-publishing Channel Adapter and Operation-invoking Channel Adapter | N | Operation-invoking Outbound Gateway |
| JPA | Inbound Channel Adapter | Outbound Channel Adapter | N | Updating Outbound Gateway and Retrieving Outbound Gateway |
| Apache Kafka | Message Driven Channel Adapter and Inbound Channel Adapter | Outbound Channel Adapter | Inbound Gateway | Outbound Gateway |
| Mail | Mail-receiving Channel Adapter | Mail-sending Channel Adapter | N | N |
| MongoDB | MongoDB Inbound Channel Adapter | MongoDB Outbound Channel Adapter | N | N |
| MQTT | Inbound (Message-driven) Channel Adapter | Outbound Channel Adapter | N | N |
| R2DBC | R2DBC Inbound Channel Adapter | R2DBC Outbound Channel Adapter | N | N |
| Redis | Redis Inbound Channel Adapter, Redis Queue Inbound Channel Adapter, Redis Store Inbound Channel Adapter, Redis Stream Inbound Channel Adapter | Redis Outbound Channel Adapter, Redis Queue Outbound Channel Adapter, RedisStore Outbound Channel Adapter, Redis Stream Outbound Channel Adapter | Redis Queue Inbound Gateway | Redis Outbound Command Gateway and Redis Queue Outbound Gateway |
| Resource | Resource Inbound Channel Adapter | N | N | N |
| RMI | N | N | Inbound RMI | Outbound RMI |
| RSocket | N | N | RSocket Inbound Gateway | RSocket Outbound Gateway |
| SFTP | SFTP Inbound Channel Adapter | SFTP Outbound Channel Adapter | N | SFTP Outbound Gateway |
| STOMP | STOMP Inbound Channel Adapter | STOMP Outbound Channel Adapter | N | N |
| Stream | Reading from Streams | Writing to Streams | N | N |
| Syslog | Syslog Inbound Channel Adapter | N | N | N |
| TCP | TCP Adapters | TCP Adapters | TCP Gateways | TCP Gateways |
| UDP | UDP Adapters | UDP Adapters | N | N |
| WebFlux | WebFlux Inbound Channel Adapter | WebFlux Outbound Channel Adapter | Inbound WebFlux Gateway | Outbound WebFlux Gateway |
| Web Services | N | N | Inbound Web Service Gateways | Outbound Web Service Gateways |
| Web Sockets | WebSocket Inbound Channel Adapter | WebSocket Outbound Channel Adapter | N | N |
| XMPP | XMPP Messages and XMPP Presence | XMPP Messages and XMPP Presence | N | N |
| ZeroMQ | ZeroMQ Inbound Channel Adapter | ZeroMQ outbound Channel Adapter | N | N |

출처 : https://docs.spring.io/spring-integration/reference/html/endpoint-summary.html