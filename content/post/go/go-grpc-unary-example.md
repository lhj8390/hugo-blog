+++
author = "lhj8390"
title = "Unary RPC 통신 방식 예제"
date = "2023-05-18"
description = "grpc에서 통신 방식 중 unary RPC에 대해 설명한다."
tags = ["Go", "Golang", "gRPC", "unary"]
categories = ["language"]
subcategories = ["Go"]
+++

> gRPC 통신 방법 중 하나로 클라이언트에서 단일 요청을 보내고 서버에서 단일 응답을 반환하는 형태이다.

<br/>

### 1. proto 파일 생성

```protobuf
syntax = "proto3";  
package ecommerce;  
option go_package = "productinfo/service/ecommerce";  
  
service ProductInfo {  
	rpc addProduct(Product) returns (ProductId);  
	rpc getProduct(ProductId) returns (Product);  
}  
  
message Product {  
	string id = 1;  
	string name = 2;  
	string description = 3;  
	float price = 4;  
}  
  
message ProductId {  
	string value = 1;  
}
```

전체 서비스를 정의해준다. Product를 생성하고 id 값을 사용하여 Product 값을 반환받는 서비스를 작성할 예정이다.

<br/>

```sh
protoc productinfo/service/ecommerce/product.proto \
	--go_out=. \
    --go_opt=paths=source_relative \
    --go-grpc_out=. \
    --go-grpc_opt=paths=source_relative
```

입력한 proto 파일을 토대로 protoc 명령어를 사용하여 protocol buffer 메시지와 gRPC 서비스 코드가 포함되어 있는 pb.go 파일과 grpc.pb.go 파일을 생성해준다. 

<br/>

### 2. 서버 로직 구현

proto 파일에서 정의한 service 구현 작업이 필요하다. addProduct와 getProduct 메서드를 구현해준다.

```go
type server struct {  
	pb.UnimplementedProductInfoServer  // 상위 호환성 위해 추가
	productMap map[string]*pb.Product  
}  

// Product 생성 기능 구현
func (s *server) AddProduct(ctx context.Context, in *pb.Product) (*pb.ProductId, error) {  
	out, err := uuid.NewV4()  
	if err != nil {  
		return nil, status.Errorf(codes.Internal, "Error while generating Product ID", err)  
	}  
	in.Id = out.String()  
	if s.productMap == nil {  
		s.productMap = make(map[string]*pb.Product)  
	}  
	s.productMap[in.Id] = in  
	return &pb.ProductId{Value: in.Id}, status.New(codes.OK, "").Err()  
}  

// Product 조회 기능 구현
func (s *server) GetProduct(ctx context.Context, in *pb.ProductId) (*pb.Product, error) {  
	value, exists := s.productMap[in.Value]  
	if exists {  
		return value, status.New(codes.OK, "").Err()  
	}  
	return nil, status.Errorf(codes.NotFound, "Product does not exist.", in.Value)  
}
```

이때, protoc 명령어를 통해 생성된 인터페이스에 맞게 구현을 해야 한다. 

또한 server 구조체에 대해 `pb.UnimplementedProductInfoServer` 를 추가해준 것을 볼 수 있다. 그 이유는 mustEmbedUnimplemented 가 추가됨에 따라 모든 구현체는 상위 호환성을 위해 Unimplemented 메서드를 추가해주어야 하기 때문이다.

<br/>

이제 gRPC 서버를 구현하는 코드를 작성해준다.

```go
func main() {  
    // tcp 연결을 위한 port binding
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {  
		log.Fatalf("failed to listen: %v", err)  
	}
	// gRPC server 생성 및 서비스 등록
	s := grpc.NewServer()  
	pb.RegisterProductInfoServer(s, &server{})  
	  
	log.Printf("Starting gRPC listener on port " + port)  
	if err := s.Serve(lis); err != nil {  
		log.Fatalf("failed to serve: %v", err)  
	}
}
```

50051 포트에 대해 TCP 연결을 수신 대기하고 새로운 gRPC 서버를 생성한다. 그리고 protoc 명령어를 통해 생성한 `ProductInfoServer` 서비스를 등록한다. 이때, `&server{}`는 ProductInfoServer 인터페이스를 구현한 구조체이다. -> *<font color="#7f7f7f">이 구조체를 사용하여 실제 클라이언트 요청을 처리하는 로직을 구현한다.</font>*

마지막으로 `s.Serve(lis)` 를 호출하여 서버를 실행한다.

<br/>

### 3. 클라이언트 로직 구현

```go
func main() {  
	conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())  
	if err != nil {  
		log.Fatalf("did not connect: %v", err)  
	}  
	defer conn.Close()  

	// 서버와 통신할 클라이언트 인스턴스 생성
	c := pb.NewProductInfoClient(conn)  
	  
	name := "Apple iPhone 11"  
	description := "Meet Apple iPhone 11."  
	price := float32(1000.0)  

	// 호출 시간 제한 : 1초
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)  
	defer cancel()  
	  
	r, err := c.AddProduct(ctx, &pb.Product{Name: name, Description: description, Price: price})  
	if err != nil {  
		log.Fatalf("Could not add product: %v", err)  
	}  
	log.Printf("Product ID: %s added successfully", r.Value)  
	  
	product, err := c.GetProduct(ctx, &pb.ProductId{Value: r.Value})  
	if err != nil {  
		log.Fatalf("Could not get product: %v", err)  
	}  
	log.Printf("Product: %s", product.String())  
}
```

localhost의 50051 포트의 gRPC 서버와의 연결을 시도하는 클라이언트 코드이다. 

AddProduct 메서드를 호출하여 Product 정보를 추가하고, GetProduct 메서드를 호출하여 추가한 Product 정보를 조회하였다. 서버가 1초 이상 응답을 반환하지 않으면 오류가 발생한다. 마지막으로 defer 키워드를 사용하여 클라이언트 연결을 종료한다.

<br/>

위 코드는 Unary RPC를 이용한 통신을 보여준다. 이 코드를 통해 클라이언트와 서버가 서로 단일 요청과 단일 응답을 주고 받는 1:1 관계라는 것을 알 수 있다. 또한 클라이언트가 요청을 보내고 서버로부터 응답을 받을 때까지 대기하는 동기적 호출을 한다는 것을 보여준다.

즉, unary 통신은 <font color="#c0504d">단순하고 신뢰성이 높은 통신을 주고 받을 경우</font> 많이 사용한다.

<br/><br/><br/>