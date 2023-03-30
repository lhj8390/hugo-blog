+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] REST API 생성하기"
author = "lhj8390"
description = "스프링에서 REST API를 생성하는 방법에 대해 설명한다."
date = "2022-07-22"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

## 개요

---

> 스프링인액션 6장을 읽고 REST 서비스를 생성해보았다.
> 
> - REST 엔드포인트 정의하기
> - REST 리소스 활성화
> 
> 에 대해 기록하였다.
> 

## REST API 기본 정의

---

> REST API란 REST 아키텍처의 제약 조건을 준수하는 애플리케이션 프로그래밍 인터페이스를 뜻한다.<br/>
> 범용성 있는 서버 구축과 함께 클라이언트와 서버 간의 역할을 명확히 분리하도록 설계하는 것이 일반적이 되면서 REST API를 많이 사용하고 있다.
> 

REST API는 **HTTP URI를 통해 자원을 명시하고 HTTP Method를 통해 해당 자원의 대한 CRUD Operation을 적용**한다.

### GET 요청 처리

@RestController 어노테이션은 컨트롤러의 모든 HTTP 요청 처리 메서드에서 HTTP 응답 body에 값을 반환한다는 것을 스프링에게 알려준다.

```java
@RestController
@RequestMapping(path="/design", produces="application/json")    
@CrossOrigin(origins="*")                                       
public class DesignTacoController {

  @GetMapping("/{id}")
  public ResponseEntity<Taco> tacoById(@PathVariable("id") Long id) {
    Optional<Taco> optTaco = tacoRepo.findById(id);
    if (optTaco.isPresent()) {
      return new ResponseEntity<>(optTaco.get(), HttpStatus.OK);
    }
    return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
  }
}

```

- **@RequestMapping(path="/design")** : /design 경로의 요청을 처리하도록 지시한다.
- **@RequestMapping(produces="application/json")** : request의 Accept 헤더에 “application/json이 포함된 요청만을 해당 메서드에서 처리하도록 설정한다.
- **@CrossOrigin(origins="*")** : 다른 도메인의 클라이언트에서 해당 REST API를 사용할 수 있도록 설정한다.
- 반환값은 상태코드와 함께 객체를 포함하는 ResponseEntity를 전달한다.

### POST 요청 처리

```java
@PostMapping(consumes="application/json")
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
  return tacoRepo.save(taco);
}
```

- **consumes="application/json"** : Content-type이 application/json과 일치하는 요청만 처리한다.
- **@RequestBody** : json 데이터가 Taco 객체로 변환되어 매개변수와 바인딩된다.
- **@ResponseStatus(HttpStatus.CREATED)** : 요청이 성공적이라면 201(CREATED) 상태코드를 반환한다.

### PUT/PATCH 요청 처리

- PUT : 클라이언트부터 서버로 데이터를 전송한다. <span class="gray it">- 데이터 전체를 교체</span>
- PATCH : PUT과 비슷하지만 데이터의 일부분을 변경한다.

<br/>

**PUT 요청 예시**
```java
@PutMapping(path="/{orderId}", consumes="application/json")
public Order putOrder(@RequestBody Order order) {
  return repo.save(order);
}
```

해당 주문 데이터 전체를 PUT 요청으로 제출하는 것을 알 수 있다.<br/><br/>

**PATCH 요청 예시**
```java
@PatchMapping(path="/{orderId}", consumes="application/json")
public Order patchOrder(@PathVariable("orderId") Long orderId, @RequestBody Order patch) {
  
  Order order = repo.findById(orderId).get();
  if (patch.getDeliveryName() != null)
    order.setDeliveryName(patch.getDeliveryName());
  
  if (patch.getDeliveryStreet() != null)
    order.setDeliveryStreet(patch.getDeliveryStreet());
  
  if (patch.getDeliveryCity() != null)
    order.setDeliveryCity(patch.getDeliveryCity());
  
  return repo.save(order);
}
```

PATCH는 데이터의 일부만 변경하도록 각각의 필드값이 null이 아닌지 확인 후 해당 필드 값만 변경을 시도한다.

<aside>
어노테이션은 어떤 종류의 요청을 메서드에서 처리하는지만 나타내기 때문에 일부 데이터를 변경하는 로직을 직접 작성해야 한다.<br/>

</aside>

### DELETE 요청 처리

```java
@DeleteMapping("/{orderId}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteOrder(@PathVariable("orderId") Long orderId) {
  try {
    repo.deleteById(orderId);
  } catch (EmptyResultDataAccessException e) {}
}
```

PathVariable로 제공된 주문ID를 인자로 받아 해당 주문이 존재하면 삭제를 수행한다.<br/>
만약 해당 주문 데이터가 없다면 **EmptyResultDataAccessException**이 발생된다. 

- **@ResponseStatus(HttpStatus.NO_CONTENT)** : 삭제 완료 후 204(no content) 처리

※ DELETE 요청의 응답은 body 데이터를 갖지 않으며, 반환 데이터가 없다는 것을 알리기 위해 204 상태코드를 사용한다.

## HATEOAS 사용하기

---

> **HATEOAS** : Hypermedia As The Engine of Application State<br/>
> API로부터 반환되는 리소스에 해당 리소스와 관련된 하이퍼링크들이 포함된다.<br/>
> 기존의 REST API의 경우 기존의 URL을 변경하면 이를 사용하는 모든 클라이언트가 함께 수정되어야 하기 때문에 API URL의 관리가 어렵다는 단점이 존재한다.
> 
> → 하지만 HATEOAS를 사용한다면 <span class="red it">최소한의 API URL만 알면 처리 가능한 다른 API URL을 알아내여 사용할 수 있다.</span>
> 

### 기존 REST API와 HATEOAS 비교

**기존 REST API**

```json
[
  {
    "id": 4,
    "name": "Veg-Out",
    "createdAt": "2018-01-31T20:15:53.219+0000",
    "ingredients": [
      {"id": "FLTO", "name": "Flour Tortilla", "type": "WRAP"},
      {"id": "COTO", "name": "Corn Tortilla", "type": "WRAP"},
      {"id": "TMTO", "name": "Diced Tomatoes", "type": "VEGGIES"},
    ]
  },
]
```

클라이언트가 HTTP 작업을 수행하고 싶다면 경로의 URL에 id 속성 값을 하드코딩해야 한다.<br/><br/>

**HATEOAS**

```json
{
  "_embedded": {
    "tacoModelList": [
      {
        "name": "Veg-Out",
        "createdAt": "2018-01-31T20:15:53.219+0000",
        "ingredients": [
          {
            "name": "Flour Tortilla", "type": "WRAP",
            "_links": {
              "self": { "href": "http://localhost:8080/ingredients/FLTO" }
            },
            "name": "Corn Tortilla", "type": "WRAP",
            "_links": {
              "self": { "href": "http://localhost:8080/ingredients/COTO" }
            },
            "name": "Diced Tomatoes", "type": "VEGGIES",
            "_links": {
              "self": { "href": "http://localhost:8080/ingredients/TMTO" }
            }
          }
        ],
        "_links": {
          "self": { "href": "http://localhost:8080/design/4" }
        }
      }
    ]
  },
  "_links": {
    "recent": {
      "self": { "href": "http://localhost:8080/design/resent" }
    }
  }
}
```

이런 형태의 HATEOAS를 HAL(Hypertext Application Language)라고 한다.<br/>
JSON 응답에 하이퍼링크를 포함시킬 때 주로 사용된다.

### HATEOAS 빌드 명세

```groovy
implementation 'org.springframework.hateoas:spring-hateoas:1.1.4.RELEASE'
```

API에서 하이퍼미디어를 사용할 수 있게 하려면 해당 dependency를 추가한다.

### 하이퍼링크 추가

HATEOAS는 하이퍼링크 리소스를 나타내는 두 개의 기본 타입이 있다.

- EntityModel : 단일 모델을 의미
- CollectionModel : 모델 컬렉션을 의미

두 타입이 전달하는 링크는 JSON 반환값에 포함된다.<br/><br/>

**기본 예제** <br/>
List<Taco>를 반환하는 대신 CollectionModel 객체를 반환하는 코드이다.

```java
@GetMapping("/recent")
public CollectionModel<EntityModel<Taco>> recentTacos() {                 
  PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());

  List<Taco> tacos = tacoRepo.findAll(page).getContent();
  CollectionModel<EntityModel<Taco>> recentModels = CollectionModel.wrap(tacos);
  recentModels.add(WebMvcLinkBuilder.linkTo(DesignTacoController.class)
                                    .slash("recent")
                                    .withRel("recents"));
  return recentModels ;
}
```

- **CollectionModel.wrap()** : Taco 리스트를 CollectionModel 인스턴스로 래핑한다.
- **WebMvcLinkBuilder** : URL을 하드코딩하지 않고 호스트 이름을 알 수 있다.
- **WebMvcLinkBuilder.slash() :** 지정한 값을 URL에 추가한다. **-** *URL의 경로는 /design/recent가 된다.*
- **WebMvcLinkBuilder.withRel() :** 해당 URL의 relation name을 지정한다.

<aside>
WebMvcLinkBuilder의 slash() 메서드 대신 <strong>methodOn()</strong> 메서드를 사용할 수 있다.

```java
WebMvcLinkBuilder.linkTo(methodOn(DesignTacoController.class).recentTacos()).withRel("recents")
```

method() 메서드를 통해 recentTacos()의 매핑 경로를 알 수 있다.

</aside>

### 모델 어셈블러 생성

리스트에 포함된 각각의 객체에 대한 링크를 추가할 경우 리스트를 반환하는 API마다 루프를 실행하는 코드가 있어야 하기 때문에 상당히 번거롭다.<br/>
따라서 CollectionModel.wrap()를 사용하여 EntityModel로 생성하는 대신 **각 객체를 새로운 EntityModel 객체로 변환하는 유틸리티 클래스를 정의**할 필요가 있다.

```java
public class TacoModel extends RepresentationModel<TacoModel> {

  private static final IngredientModelAssembler ingredientAssembler = new IngredientModelAssembler();
  
  @Getter
  private final String name;

  @Getter
  private final Date createdAt;

  @Getter
  private final CollectionModel<IngredientModel> ingredients;
  
  public TacoModel(Taco taco) {
    this.name = taco.getName();
    this.createdAt = taco.getCreatedAt();
    this.ingredients = ingredientAssembler.toCollectionModel(taco.getIngredients());
  }
}
```

TacoModel 객체는 Taco 객체와 유사하지만 링크를 추가로 가질 수 있다.<br/>
또한 RepresentationModel의 서브클래스로서 Link 객체 리스트와 이것을 관리하는 메서드를 상속받는다.

Taco 객체를 TacoModel 객체들로 변환하는 데 도움을 주는 모델 어셈블러 클래스를 생성한다.

```java
public class TacoModelAssembler extends RepresentationModelAssemblerSupport<Taco, TacoModel> {

  public TacoModelAssembler() {
    super(DesignTacoController.class, TacoModel.class);
  }

  @Override
  public TacoModel toModel(Taco taco) {
    return new TacoModel(taco);
  }
}
```

- **TacoModelAssembler 생성자** : URL의 기본 경로를 설정하기 위해 DesignTacoController를 사용한다.
- **toModel()** : RepresentationModelAssemblerSupport 부터 상속받을 때 반드시 오버라이드 해야한다. <span class="red">**TacoModel 인스턴스를 생성하면서 Taco 객체의 id 속성 값으로 생성되는 self 링크가 URL에 자동 지정된다.**</span>
<br/><br/>

**모델 어셈블러 사용**

```java
@GetMapping("/recent")
public CollectionModel<TacoModel> recentTacos() {
  PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
  List<Taco> tacos = tacoRepo.findAll(page).getContent();

  CollectionModel<TacoModel> recentCollection = new TacoModelAssembler().toCollectionModel(tacos);
  recentCollection.add(linkTo(methodOn(DesignTacoController.class).recentTacos())
                              .withRel("recents"));
  return recentCollection;
}
```

- **toCollectionModel()** : TacoModel 객체를 저장한 CollectionModel을 생성한다.

### embedded 관계 이름 짓기

```json
{
  "_embedded": {
    "tacoModelList": [
      
    ]
  }
}
```

embedded 밑의 “tacoModelList” 이름은 CollectionModel 객체가 List<TacoModel>로부터 생성되었다는 것을 나타낸다. 즉 TacoModel 클래스가 변경될 경우 JSON 필드 이름이 변경된다.
<br/><br/>

이를 해결하기 위해서는 @Relation 어노테이션을 사용한다.

```java
@Relation(value="taco", collectionRelation="tacos")
public class TacoModel extends RepresentationModel<TacoModel> {
  ...
}
```

TacoModel 객체 리스트가 CollectionModel 객체에서 사용될 때 tacos라는 이름이 되도록 지정할 수 있다.
<br/><br/>

JSON 형식은 다음과 같이 변경된다.

```json
{
  "_embedded": {
    "tacos": [
		
    ]
  }
}
```

## 데이터 기반 서비스 활성화

---

> 스프링 데이터에는 어플리케이션의 API를 정의하는 데 도움을 주는 기능도 존재한다.<br/>
> 스프링 데이터 REST는 스프링 데이터가 생성하는 repository의 REST API를 자동 생성한다.
> 

### 스프링 데이터 REST 빌드 명세

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-rest:2.3.9.RELEASE'
```

dependency를 추가함으로써 모든 repository의 REST API를 노출시킬 수 있다.

### 홈 리소스 내역 확인

스프링 데이터 REST dependency만 추가해도 REST API를 자동 생성할 수 있다.

```shell
$ curl http://localhost:8080/api/
```

해당 명령어를 통해 노출된 모든 엔드포인트의 링크를 가지는 홈 리소스 내역을 확인할 수 있다.

### 리소스 경로 커스터마이징

스프링 데이터 REST의 문제점 중의 하나는 **클래스 이름의 복수형을 사용한다**는 점에 있다.<br/>
리소스 경로를 변경하기 위해서는 다음과 같이 설정한다. 

```java
@Data
@Entity
@RestResource(rel="tacos", path="tacos")
public class Taco {
  ...
}
```

**@RestResource** 어노테이션을 통해 <span class="red">tacoes로 자동 설정되었던 엔드포인트를 /api/tacos로 변경해주었다.</span>

### 페이징, 정렬 처리

홈 리소스의 모든 링크는 page, size, sort를 제공한다.<br/>
기본적으로 한 페이지 당 20개의 항목을 반환하는데, page, size 매개변수를 통해 페이지 번호와 크기를 조절할 수 있다.

```shell
$ curl localhost:8080/api/tacos?sort=createdAt,desc&page=0&size=12
```

createdAt 속성 값을 기준으로 내림차순으로 정렬하고 첫번째 페이지에 12개의 taco가 나타나도록 설정하는 예시이다.

### 커스텀 엔드포인트 추가하기

커스텀 엔드포인트를 추가하기 위해서는 두가지 사항을 고려하여 작성해야 한다.<br/>
1. 스프링 데이터 REST 기본 경로를 포함하여 원하는 기본 경로가 앞에 붙도록 매핑시켜야 한다.
2. 커스텀한 엔드포인트는 스프링 데이터 REST 엔드포인트에서 반환되는 리소스의 하이퍼링크에 포함되지 않는다.
<br/><br/>

**스프링 데이터 REST 기본 경로 컨트롤러에 적용**

```java
@RepositoryRestController
public class RecentTacosController {

  private TacoRepository tacoRepo;

  public RecentTacosController(TacoRepository tacoRepo) {
    this.tacoRepo = tacoRepo;
  }

  @GetMapping(path="/tacos/recent", produces="application/hal+json")
  public ResponseEntity<RepresentationModel<TacoModel>> recentTacos() {
    PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
    List<Taco> tacos = tacoRepo.findAll(page).getContent();

    CollectionModel<TacoModel> tacoCollection = new TacoResourceAssembler().toCollectionModel(tacos);
    CollectionModel<TacoModel> recentCollection = new CollectionModel<TacoModel>(tacoCollection);
    
    recentResources.add(linkTo(methodOn(RecentTacosController.class).recentTacos())
                                .withRel("recents"));
    return new ResponseEntity(recentCollection, HttpStatus.OK);
  }
}
```

스프링 데이터 REST의 기본 경로를 request mapping에 적용하기 위해 <span class="red">@RepositoryRestController</span>를 사용하였다.<br/>
즉 @GetMapping은 /tacos/recent 경로로 매핑되지만 @RepositoryRestController를 통해 recentTacos() 메서드는 /api/tacos/recent의 GET 요청을 처리하게 된다.

<aside>
<strong>주의할 점!</strong>

@RepositoryRestController는 @RestController와 동일한 기능을 수행하지 않는다.
따라서 @ResponseBody 어노테이션을 지정하거나 응답 데이터를 포함하는 ResponseEntity를 반환해야 한다.

</aside>

### 커스텀 하이퍼링크를 스프링 데이터 엔드포인트에 추가하기

위에서 커스텀한 /api/tacos/recent는 스프링 데이터 엔드포인트(/api/tacos)를 요청할 때 하이퍼링크 리스트에 나타나지 않는다. <br/>
이를 해결하기 위해 `RepresentationModelProcessor` 를 사용한다.

```java
@Bean
public RepresentationModelProcessor<PagedModel<EntityModel<Taco>>>
  tacoProcessor(EntityLinks links) {

  return new RepresentationModelProcessor<PagedModel<EntityModel<Taco>>>() {
    @Override
    public PagedModel<EntityModel<Taco>> process(PagedModel<EntityModel<Taco>> resource) {
      resource.add(links.linkFor(Taco.class)
                        .slash("recent")
                        .withRel("recents"));
      return resource;
    }
  };
}
```

해당 코드를 통해 스프링 HATEOAS는 자동으로 빈을 찾은 후 해당되는 리소스에 적용한다.<br/>
여기서는 /api/tacos 요청 응답에 /api/tacos/recent 링크를 포함하게 된다.