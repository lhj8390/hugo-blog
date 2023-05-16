+++
author = "lhj8390"
title = "gRPC 통신 방식"
date = "2023-05-15"
description = "grpc에서 통신 방식의 종류인 unary RPC, Server streaming RPC, Client streaming RPC, bidi streaming RPC에 대해 설명한다."
tags = ["Go", "Golang", "gRPC"]
categories = ["language"]
subcategories = ["Go"]
+++

> gRPC에서 지원하는 통신 방식은 총 4가지로 나눌 수 있으며 그 중 3가지 방식은 Streaming 처리 방식을 지원한다. 

기본적으로 gRPC는 서비스 인터페이스와 메시지의 구조를 설명하기 위해 프로토콜 버퍼를 IDL로 사용하고 있으며 이 인터페이스를 기준으로 다양한 환경의 클라이언트와 통신할 수 있도록 도와준다.

<br/>

### Unary RPC

가장 단순한 서비스 구현 방법으로, 클라이언트에서 단일 요청을 보내고 서버에서 단일 응답을 반환하는 형태이다.

```protobuf
service HelloService {
    rpc SayHello(HelloRequest) returns (HelloResponse);
}
```

1. 클라이언트가 스텁 메서드를 호출하면 서버는 RPC가 이 호출에 대한 클라이언트의 메타데이터, 메서드 이름, 지정된 마감일(해당되는 경우)과 함께 호출되었음을 알린다.
2. 그런 다음 서버는 자신의 초기 메타데이터(응답 전에 전송됨)를 즉시 다시 보내거나 클라이언트의 요청 메시지를 기다린다. 발생하는 순서는 애플리케이션에 따라 다르다.
3. 서버는 클라이언트의 요청 메시지를 수신하면 응답을 작성한다. 응답이 성공한 경우 상태 코드 및 상태 메시지와 후행 메타데이터를 클라이언트로 반환한다.
4. 응답 상태가 정상이면 클라이언트는 응답을 수신하여 클라이언트 측에서의 호출을 완료한다.

<br/>

### Server streaming RPC

클라이언트의 요청은 한번만 전달하고 서버에서 응답은 여러 번에 걸쳐 전송하는 형태이다.

```protobuf
service HelloService {
    rpc HelloReplies(HelloRequest) returns (stream HelloResponse);
}
```

서버가 클라이언트의 요청에 응답하여 메시지 스트림을 반환한다는 점을 제외하고는 Unary RPC와 유사하다. 서버는 모든 스트림 메시지를 보낸 후 서버의 상태 코드 및 상태 메시지와 후행 메타데이터를 클라이언트로 전송한다. 또한 각각의 요청에 대해 스트림 메세지 순서를 보장한다.

<br/>

### Client streaming RPC

클라이언트는 Stream 형태로 요청을 전달하고 일반적으로 클라이언트의 요청이 끝나면 서버에서 한번에 응답을 반환하는 형태이다.

```protobuf
service HelloService {
   rpc SayHellos(stream HelloRequest) returns (HelloResponse);
}
```

클라이언트가 단일 메시지 대신 메시지 스트림을 서버로 보낸다는 점을 제외하면 Unary RPC와 유사하다. 서버는 일반적으로 클라이언트가 Stream을 종료하는 것을 인식하기 위해 클라이언트의 모든 메시지를 수신받은 다음에 상태 세부 정보 및 후행 메타데이터를 응답으로 보낸다. 
그러나 서버는 클라이언트가 모든 메시지를 보낼 때까지 기다리지 않고 스트림 메시지를 즉시 처리할 수도 있으며, 클라이언트 또한 언제든지 끝낼 수 있다. 또한 각각의 요청에 대해 스트림 메세지 순서를 보장한다.


<br/>

### Bidirectional streaming RPC

클라이언트와 서버 양방향 모두 Stream으로 데이터를 전송할 수 있는 형태이다.

```protobuf
service HelloService {
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
}
```

클라이언트는 메소드를 호출하고 이를 수신하는 서버에 의해 호출이 시작된다. 서버는 초기 메타데이터를 다시 보내거나 클라이언트가 스트리밍 메시지를 시작할 때까지 기다린다.  
클라이언트 및 서버 측 스트림 처리는 응용 프로그램마다 다르며 독립적이므로 클라이언트와 서버 모두 임의의 순서로 메시지를 읽고 쓸 수 있다. <font color="#7f7f7f">ex. 서버는 메시지를 작성하기 전에 클라이언트의 모든 메시지를 수신할 때까지 기다리거나 서버와 클라이언트가 서로 ping을 보내는 등의 방법으로 상호작용할 수 있다. </font>
 
<br/><br/><br/>