+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 리액티브 API 개발"
author = "lhj8390"
description = "스프링에서 리액티브 API를 개발하는 방법에 대해 설명한다."
date = "2022-08-19"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Framework"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
## 개요

---

> 스프링인액션 11장을 읽고 리액티브 WebFlux를 사용하여 리액티브 API를 개발하는 방법에 대해 공부하였다.
> 
> - 스프링 WebFlux 사용하기
> - 리액티브 컨트롤러와 클라이언트 작성 및 테스트
> - REST API 소비하기
> - 리액티브 웹 어플리케이션의 보안
> 
> 에 대해 작성하였다.
> 

## WebFlux 사용

---

> 비동기 웹 프레임워크는 적은 수의 스레드로 높은 확장성을 가지고 있다. <br/>
> **이벤트 루핑**이라는 기법을 적용하여 한 스레드 당 많은 요청을 처리할 수 있어서 스레드 관리 부담이 줄어들고 확장이 용이하다.
> 
> {{<figure src="/images/spring-reactive-api/1.png" class="large" caption="비동기 웹 프레임워크는 이벤트 루핑을 통해 적은 수의 스레드로 많은 요청을 처리한다.">}}
> 
> 스프링 5에서는 프로젝트 리액터를 기반으로 한 비동기 웹 프레임워크인 스프링 WebFlux가 존재한다.
> 

### WebFlux 빌드 명세 추가

```groovy
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

스프링 WebFlux를 사용하기 위해서는 `spring-boot-starter-web` 대신 스프링 부트 WebFlux 스타터 dependency를 추가해야 한다.
WebFlux를 사용할 경우에는 내장 서버가 톰캣 대신 Netty가 된다.

<aside>
💡 <strong>Netty</strong> : 비동기적인 이벤트 중심의 서버 중 하나<br/>

</aside>

### 리액티브 컨트롤러 생성

리액티브 타입인 Flux 객체를 반환하는 컨트롤러를 생성한다.
WebFlux 컨트롤러에서는 Flux와 같은 리액티브 타입을 받을 때 프레임워크에서 subscirbe()를 호출해주기 때문에 별도로 subscirbe()를 호출할 필요가 없다.<br/><br/>

**End-to-End 리액티브 스택**

{{<figure src="/images/spring-reactive-api/2.png" class="large" >}}

리액티브 컨트롤러가 리액티브 End-to-End 스택의 제일 끝에 위치해야 하며 리퍼지터리에서 Flux를 반환하도록 작성되어야 한다.<br/><br/>

**기본 예제**

```java
@GetMapping("/recent")
public Flux<Taco> recentTacos() {
    return tacoRepo.findAll().take(12);
}
```

페이징 처리를 위해 take() 메서드를 사용한 예제이다. (repository에서 Flux 타입 반환)<br/><br/>

**RxJava 타입 사용**

```java
@GetMapping("/recent")
public Observable<Taco> recentTacos() {
    return tacoService.getRecentTacos();
}
```

스프링 WebFlux를 사용할 때는 Observable이나 Single 같은 Rxjava 타입을 사용할 수 있다.
이뿐만 아니라 리액터의 Mono 타입과 동일한 Rxjava의 Completable 타입, Observable이나 Flux 타입의 대안으로 Flowable 타입을 반환할 수도 있다.<br/><br/>

**입력처리**

```java
@PostMapping(consumes = "application/json")
@ResponseStatus(HttpStatus.CREATED)
public Mono<Taco> postTaco(@RequestBody Mono<Taco> tacoMono) {
  return tacoRepo.saveAll(tacoMono).next();
}
```

요청을 처리하는 핸들러 메서드의 입력으로 Mono나 Flux 타입을 받을 수 있다.<br/>
saveAll() 메서드는 리액티브 스트림의 Publisher 인터페이스를 구현한 타입(Mono, Flux)을 인자로 받을 수 있다.

- **next()** : saveAll() 메서드가 반환하는 Flux가 하나의 객체만 포함하고 있기 때문에 해당 메서드를 통해 <span class="red">Mono 타입 객체를 받을 수 있다.</span>

*“MVC 패턴과 다르게 requestBody로부터 객체가 분석되지 않고 즉시 호출된다.”*

## 함수형 요청 핸들러 정의

---

> MVC나 WebFlux 같은 어노테이션 기반 프로그래밍의 단점(확장, 디버깅의 어려움)을 개선하기 위해 리액티브 API를 정의하는 함수형 프로그래밍 모델이 존재한다.
> 

**함수형 프로그래밍 모델 기본 타입**

| Type | Description |
| --- | --- |
| RequestPredicate  | 처리될 요청의 종류 선언 |
| RouterFunction | 일치하는 요청이 어떻게 핸들러에게 전달되는지 선언 |
| ServerRequest | HTTP 요청을 나타내며, header와 body 정보 사용 |
| ServerResponse | HTTP 응답을 나타내며, header와 body 정보 포함 |

### 기본 예제

```java
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

@Configuration
public class RouterFunctionConfig {
  @Bean
  public RouterFunction<?> helloRouterFunction() {
    return route(GET("/hello"), 
        request -> ok().body(just("Hello!"), String.class))
        .andRoute(GET("/bye"),
        request -> ok().body(just("See ya!"), String.class));
  }
}
```

함수형 타입을 생성하는 데 사용하는 도우미 클래스를 static import한다.<br/>
route() 메서드는 (1) <span class="ul">RequestPredicate 객체</span>와 (2) <span class="ul">일치하는 요청을 처리하는 함수</span>, 두 가지 인자를 가진다.

다른 종류의 요청을 처리해야 할 경우는 **andRoute() 메서드**를 호출한다.

### 함수형 방식을 사용하여 컨트롤러 생성

```java
@Configuration
public class RouterFunctionConfig {

  @Autowired
  private TacoRepository tacoRepo;

  @Bean
  public RouterFunction<?> routerFunction() {
    return route(GET("/design/taco"), this::recents)
        .andRoute(POST("/design"), this::postTaco);
  }

  public Mono<ServerResponse> recents(ServerRequest request) {
    return ServerResponse.ok().body(tacoRepo.findAll().take(12), Taco.class);
  }

  public Mono<ServerResponse> postTaco(ServerRequest request) {
    Mono<Taco> taco = request.bodyToMono(Taco.class);
    Mono<Taco> savedTaco = tacoRepo.save(taco);
    return ServerResponse.created(URI.create("http://localhost:8080/design/taco/" +
        savedTaco.getId()))
        .body(savedTaco, Taco.class);
  }
}
```

`/design/taco GET` 요청이 들어왔을 경우 recents() 메서드에서 처리되고, `/design POST` 요청이 들어왔을 경우 postTaco() 메서드에서 처리한다.

## 리액티브 컨트롤러 테스트

WebFlux를 사용하는 리액티브 컨트롤러의 테스트를 쉽게 작성할 수 있게 도와주는 테스트 유틸리티로 **WebTestClient**가 있다.

### GET 요청 테스트

```java
@Test
public void shouldReturnRecentTacos() {
  Taco[] tacos = {
      testTaco(1L), testTaco(2L),
      testTaco(3L), testTaco(4L),
      testTaco(5L), testTaco(6L)};      // 테스트 데이터 생성
  Flux<Taco> tacoFlux = Flux.just(tacos);
  
  TacoRepository tacoRepo = Mockito.mock(TacoRepository.class);
  when(tacoRepo.findAll()).thenReturn(tacoFlux);  // 모의 TacoRepository
  
  WebTestClient testClient = WebTestClient.bindToController(
      new DesignTacoController(tacoRepo))
      .build();    // WebTestClient 생성
  
  testClient.get().uri("/design/recent")
    .exchange()    // 가장 최근의 객체 요청
    .expectStatus().isOk()    // 기대한 응답인지 검사
    .expectBody()
      .jsonPath("$").isArray()
      .jsonPath("$").isNotEmpty()
      .jsonPath("$[0].id").isEqualTo(tacos[0].getId().toString())
      .jsonPath("$[0].name").isEqualTo("Taco 1")
      .jsonPath("$[12]").doesNotExist();
}
```

expectStatus() 메서드를 호출하여 응답이 200(OK) 상태 코드를 가지는지 확인한다. <br/>
jsonPath() 메서드를 호출하여 응답 몸체의 JSON이 기대값을 가지는지 검사한다.
<br/><br/>
<aside>
❗ &nbsp;응답에 JSON 데이터가 많아 복잡한 경우에는 json() 메서드를 사용한다. 

**json()** : <span class="red">JSON을 포함하는 String을 인자로 받아 응답과 비교한다.</span>

```java
ClassPathResource recentsResource = new ClassPathResource("/tacos/recent.json");
String recentsJson = StreamUtils.copyToString(
    recentsResource.getInputStream(), Charset.defaultCharset());

testClient.get().uri("/design/recent")
  .accept(MediaType.APPLICATION_JSON)
  .exchange()
  .expectStatus().isOk()
  .expectBody()
  .json(recentsJson);
```

recentsJson이라는 JSON 응답이 담긴 String 값을 인자로 전달하여 올바른 응답인지 확인한다.
리스트 형태의 응답을 비교할 수 있는 **expectBodyList()** 메서드도 존재한다.

```java
testClient.get().uri("/design/recent")
  .accept(MediaType.APPLICATION_JSON)
  .exchange()
  .expectStatus().isOk()
  .expectBodyList(Taco.class)
  .contains(Arrays.copyOf(tacos, 12));
```

응답이 <span class="red">List에 저장된 12가지 요소를 모두 포함하는지</span> 검사하는 로직이다.

</aside>

### POST 요청 테스트

```java
@Test
public void shouldSaveATaco() {
  TacoRepository tacoRepo = Mockito.mock(TacoRepository.class);
  Mono<Taco> unsavedTacoMono = Mono.just(testTaco(null));
  Taco savedTaco = testTaco(null);
  savedTaco.setId(UUID.randomUUID());
  Mono<Taco> savedTacoMono = Mono.just(savedTaco);
  
  when(tacoRepo.save(any())).thenReturn(savedTacoMono);  // 모의 TestRepository
  
  WebTestClient testClient = WebTestClient.bindToController(  // WebTestClient 생성
      new DesignTacoController(tacoRepo)).build();
  
  testClient.post()    // post 처리
      .uri("/design")
      .contentType(MediaType.APPLICATION_JSON)
      .body(unsavedTacoMono, Taco.class)
    .exchange()
    .expectStatus().isCreated()  // 응답을 검사한다
    .expectBody(Taco.class)
      .isEqualTo(savedTaco);
}
```

post() 메서드를 통해 POST 요청을 제출하고 application/json 타입의 몸체와 페이로드를 같이 전달한다.<br/>
exchange() 실행 후 응답이 CREATED(201) 상태 코드를 가지는지, 응답으로 객체와 동일한 페이로드를 가지는지 검사한다.

### 실행 중인 서버로 테스트

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class DesignTacoControllerWebTest {

  @Autowired
  private WebTestClient testClient;

  @Test
  public void shouldReturnRecentTacos() {
    testClient.get().uri("/design/recent")
      .accept(MediaType.APPLICATION_JSON).exchange()    // 가장 최근의 객체 요청
      .expectStatus().isOk()    // 기대한 응답인지 검사
      .expectBody()
        .jsonPath("$[?(@.id == 'TACO1')].name").isEqualTo("Carnivore")
        .jsonPath("$[?(@.id == 'TACO2')].name").isEqualTo("Bovine Bounty")
        .jsonPath("$[?(@.id == 'TACO3')].name").isEqualTo("Veg-out")
  }
}
```

WebTestClient의 통합 테스트를 위해서는 @RunWith와 @SpringBootTest 어노테이션을 지정해야 한다.

- `WebEnvironment.RANDOM_PORT` : 무작위로 선택된 포트로 실행 서버가 리스닝하도록 요청

자동 연결되는 WebTestClient를 사용하기 때문에 인스턴스를 생성할 필요가 없다는 특징이 있다.

## REST API 사용

---

> RestTemplate이 제공하는 모든 메서드는 도메인 타입이나 컬렉션 타입을 처리하기 때문에 스프링 5에서는 RestTemplate 대신 리액티브 타입을 사용할 수 있는 대안으로 **WebClient**가 존재한다.
> 
<br/>

**일반적인 WebClient 패턴**

1. WebClient 인스턴스를 생성한다.
2. 요청을 전송할 HTTP 메서드를 지정한다.
3. 요청에 필요한 URI와 헤더를 지정한다.
4. 요청을 제출한다.
5. 응답을 사용한다.

### GET 사용

```java
Mono<Ingredient> ingredient = WebClient.create()
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .retrive()
    .bodyToMono(Ingredient.class); // 단일 값

ingredient.subscribe(i -> {});
```

create() 메서드로 새로운 WebClient 인스턴스를 생성하고, get()과 uri()를 통해 GET 요청을 정의한다.<br/>
bodyToMono() 메서드 호출을 통해 response body의 페이로드를 Mono<Ingredient>로 추출한다.

- `bodyToMono()` : 단일 값 반환 요청
- `bodyToFlux()` : 컬렉션 값 반환 요청

<span class="gray">※ Mono에 추가적으로 오퍼레이션을 적용하려면 subscribe() 메서드를 호출한다.</span><br/><br/>

**요청 타임아웃**

```java
ingredients.timeout(Duration.ofSeconds(1))
  .subscribe(i -> {}, e -> { // handle timeout error });
```

클라이언트 요청이 지체되는 것을 방지하기 위해 **timeout() 메서드**를 사용할 수 있다.<br/>
경과 시간을 1초로 지정하여 1초보다 오래 걸리면 타임아웃이 되어 <span class="red">subscribe 두번째 인자로 지정한 에러 핸들러가 호출</span>된다.

### 리소스 전송

```java
Mono<Ingredient> ingredientMono= ...;
Mono<Ingredient> result = WebClient.create()
    .post()
    .uri("http://localhost:8080/ingredients/")
    .body(ingredientMono, Ingredient.class)
    .retrive()
    .bodyToMono(Ingredient.class); 

result.subscribe(i -> {});
```

post()과 uri()를 통해 POST 요청을 정의하고 body() 메서드를 호출하여 Mono를 responsebody에 포함시킨다.<br/><br/>

**도메인 객체 전송**

```java
Ingredient ingredient = ...;
Mono<Ingredient> result = WebClient.create()
    .post()
    .uri("http://localhost:8080/ingredients/")
    .syncBody(ingredient)
    .retrive()
    .bodyToMono(Ingredient.class); 
```

Mono 타입 대신 도메인 객체를 responsebody에 포함시켜 전송하려면 **syncBody() 메서드**를 사용한다.

### 리소스 삭제

```java
Mono<Ingredient> ingredient = WebClient.create()
    .delete()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .retrive()
    .bodyToMono(Void.class)
    .subscribe();
```

DELETE 요청은 응답 페이로드를 가지지 않기 때문에 Mono<Void>를 반환하고 subscribe()로 구독해야 한다.

### 에러 처리

에러가 생길 수 있는 Mono나 Flux를 구독할 때는 subscribe() 메서드 내에 에러 컨슈머를 등록할 필요가 있다.

```java
ingredientMono.subscribe(
    ingredient -> {
      // 데이터 처리
    },
    error -> {
      // 에러 처리
    });
```

404 같은 상태 코드가 반환되면 두번째 인자로 전달된 에러 컨슈머가 실행되어 기본적으로 WebClientResponseException을 발생시킨다.<br/>
→ **WebClientResponseException** : <span class="red">구체적인 예외를 나타내지 않기 때문에 커스텀 에러 핸들러가 필요!</span>
<br/><br/>

**커스텀 에러 핸들러 구현**

```java
Mono<Ingredient> ingredient = WebClient.create()
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .retrive()
    .onStatus(HttpStatus::is4xxClientError,
            response -> Mono.just(new UnknownIngredientException()))
    .bodyToMono(Ingredient.class);
```

HTTP 상태 코드가 400 수준의 상태코드일 경우 UnknownIngredientException을 포함하는 Mono가 반환되고 ingredient 실행이 실패하게 된다.

### 요청 교환

retrieve() 메서드는 ResponseSpec 타입을 반환하는데, <span class="ul">응답의 헤더나 쿠키 값을 사용할 경우</span>에는 ResponseSpec 타입으로 처리할 수 없다는 단점이 존재한다.<br/>
이 단점을 해결하기 위해 **ClientResponse 타입의 Mono를 반환**하는 exchange() 메서드가 존재하는데, (1) <span class="ul">리액티브 오퍼레이션을 적용</span>하거나, (2) <span class="ul">응답의 모든 부분을 사용할 수 있다.</span><br/><br/>

**기본 예제**

```java
Mono<Ingredient> ingredient = WebClient.create()
    .get()
    .uri("http://localhost:8080/ingredients/{id}", ingredientId)
    .exchange()
    .flatMap(cr -> {
      if (cr.headers().header("X-UNAVAILABLE").contains("true")) {
        return Mono.empty();
      }
      return Mono.just(cr);
    })
    .flatMap(cr -> cr.bodyToMono(Ingredient.class));
```

“X-UNAVAILABLE” 헤더가 존재한다면 비어있는 Mono를 반환하고, 존재하지 않는다면 ClientResponse를 포함하는 새로운 Mono를 반환한다.

## 리액티브 웹 API 보안

---

> 스프링 시큐리티는 스프링 MVC와 리액티브 스프링 WebFlux 어플리케이션 모두 사용할 수 있다.
> 이를 도와주는 것이 스프링 **WebFilter**이다.
> 
> - WebFilter : 서블릿 API에 의존하지 않는 스프링의 서블릿 필터

### 리액티브 웹 보안 구성

스프링 WebFlux에서 스프링 시큐리티를 구성하는 코드이다.

```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {
  @Bean
  public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return http.authorizeExchange()
        .pathMatchers("/design", "/orders").hasAuthority("USER")
        .anyExchange().permitAll()
        .and().build();
  }
}
```

스프링 MVC와 다르게 @EnableWebSecurity 대신 <span class="red">@EnableWebFluxSecurity가 지정</span>되어 있다. 또한 인자로 HttpSecurity 대신 <span class="red">SecurityHttpSecurity를 사용</span>해서 authorizeExchange()를 호출하였다.<br/>
중요한 점은 프레임워크 메서드를 오버라이딩 하는 대신 **build() 메서드를 호출하여 모든 보안 규칙을 반환해야 한다.**<br/><br/>

**ReactiveUserDetailsService 구현**

```java
@Service
public ReactiveUserDetailsService userDetailsService(UserRepository userRepo) {
    return new ReactiveUserDetailsService() {
        @Override
        public Mono<UserDetails> findByUsername(String username) {
            return userRepo.findByUsername(username).map(user -> {
                return user.toUserDetails();
            });
        }
    };
}
```

userDetailsService의 리액티브 버전인 ReactiveUserDetailsService 빈을 선언하였다.<br/>
`userRepo.findByUsername()` 에서 반환된 Mono가 발행하는 User 객체의 toUserDetails() 메서드를 호출하여 <span class="red">User 객체를 UserDetails 객체로 변환</span>한다.