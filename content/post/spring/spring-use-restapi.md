+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] REST API 사용하기"
author = "lhj8390"
description = "스프링에서 REST API를 사용하는 방법에 대해 설명한다."
date = "2022-07-22"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Framework"]
subcategories = ["Spring"]
series = ["스프링인액션"]
+++
## 개요

---

> 스프링인액션 7장을 일고 스프링을 사용하여 다른 REST API를 사용하는 방법을 작성했다.
> 
> - RestTemplate를 사용해서 REST API 사용
> - Traverson을 사용해서 하이퍼미디어 API 이동
> 
> 을 작성하였다.
> 

## RestTemplate 사용

---

> Spring에서 HTTP 통신을 RESTful 형식에 맞게 손쉬운 사용을 제공해주는 템플릿이다. <br/>
> REST 리소스를 사용하는 데 있어 메서드 호출만으로도 반복적인 코드를 깔끔하게 처리해준다.
> 
<br/>

**RestTemplate의 대표 메서드**

| METHOD | Description |
| --- | --- |
| delete(…) | 지정된 URL의 리소스에 HTTP DELETE 요청을 수행 |
| exchange(…) | 지정된 HTTP 메서드를 URL에 대해 실행하며, 응답 body와 연결되는 객체를 포함하는 ResponseEntity를 반환 |
| execute(…) | 지정된 HTTP 메서드를 URL에 대해 실행하며, 응답 body와 연결되는 객체 반환 |
| getForEntity(…) | HTTP GET 요청을 전송하며, 응답 body와 연결되는 객체를 포함하는 RespnseEntity 반환 |
| getForObject(…) | HTTP GET 요청을 전송하며, 응답 body와 연결되는 객체 반환 |
| headForHeaders(…) | HTTP HEAD 요청을 전송하며, 지정된 리소스 URL의 HTTP 헤더 반환 |
| optionsForAllow(…) | HTTP OPTIONS 요청을 전송하며, 지정된 URL의 Allow 헤더 반환 |
| patchForObject(…) | HTTP PATCH 요청을 전송하며, 응답 body와 연결되는 결과 객체 반환 |
| postForEntity(…) | URL에 데이터를 POST하며, 응답 body와 연결되는 객체를 포함하는 RespnseEntity 반환 |
| postForLocation(…) | URL에 데이터를 POST하며, 새로 생성된 리소스의 URL 반환 |
| postForObject(…) | URL에 데이터를 POST하며, 응답 body와 연결되는 객체 반환 |
| put(…) | 리소스 데이터를 지정된 URL에 PUT한다. |

<br/>

**RestTemplate 생성**

```java
RestTemplate rest = new RestTemplate();
```

RestTemplate를 사용하려면 필요한 시점에 RestTemplate 인스턴스를 생성해야 한다.

### GET method

RestTemplate에서는 GET 요청을 지원하는 메서드로 getForObject()와 getForEntity()가 있다.<br/><br/>

**getForObject() 메서드**

```java
public Ingredient getIngredientById(String ingredientId) {
    return rest.getForObject("http://localhost:8080/ingredients/{id}", Ingredient.class, ingredientId);
}
```

getForObject의 매개변수는 차례로 (1) <span class="ul">URL 문자열</span>, (2) <span class="ul">응답이 바인딩되는 타입</span>, (3) <span class="ul">URL 문자열의 변수 값</span>이 된다.
<br/>
<br/>

**URL 변수 지정**

URL 변수가 여러 개일 경우 getForObject의 세번째 매개변수부터 URL 변수가 순서대로 지정된다.<br/>
하지만 하단의 코드처럼 Map을 사용하여 URL 변수를 지정해줄 수 있다.

```java
public Ingredient getIngredientById(String ingredientId) {
    Map<String, String> urlVariables = new HashMap<>();
    urlVariables.put("id", ingredientId);
    return rest.getForObject("http://localhost:8080/ingredients/{id}", Ingredient.class, urlVariables);
}
```
URL 문자열의 {id} 변수는 키가 id인 Map 항목 값으로 교체된다.
<br/><br/>

**URI 매개변수 사용**

```java
public Ingredient getIngredientById(String ingredientId) {
    Map<String, String> urlVariables = new HashMap<>();
    urlVariables.put("id", ingredientId);
    URI url = UriComponentsBuilder.fromHttpUrl("http://localhost:8080/ingredients/{id}")
                                  .build(urlVariables);
    return rest.getForObject(url, Ingredient.class);
}
```

URL 문자열 대신 URI 객체를 사용할 수 있다.
<br/><br/>

**getForEntity() 메서드**

```java
public Ingredient getIngredientById(String ingredientId) {
    ResponseEntity<Ingredient> responseEntity = 
            rest.getForEntity("http://localhost:8080/ingredients/{id}", Ingredient.class, ingredientId);
    return responseEntity.getBody();
}
```

getForEntity()는 getForObject()와 동일하지만, 응답 결과를 나타내는 도메인 객체를 반환하는 대신 도메인 객체를 포함하는 ResponseEntity 객체를 반환한다.
<br/>
→ 응답 헤더와 같은 상세한 응답 컨텐츠가 포함된다.


### PUT method

RestTemplate에서는 PUT 요청을 지원하는 메서드로 put()를 제공한다. 
<br/><br/>

**put() 메서드**

```java
public void updateIngredient(Ingredient ingredient) {
    return rest.put("http://localhost:8080/ingredients/{id}", Ingredient, ingredient.getId());
}
```

put()의 매개변수는 차례로 (1) <span class="ul">URL 문자열</span>, (2) <span class="ul">URL로 전송하는 객체</span>, (3) <span class="ul">URL 문자열의 변수 값</span>이 된다. <br/>
반환 타입은 void이므로 반환값을 처리할 필요가 없다.

### DELETE method

RestTemplate에서는 DELETE 요청을 지원하는 메서드로 delete()를 제공한다. 
<br/><br/>

**delete() 메서드**

```java
public void deleteIngredient(Ingredient ingredient) {
	return rest.delete("http://localhost:8080/ingredients/{id}", ingredient.getId());
}
```

delete()의 매개변수는 차례로 (1) <span class="ul">URL 문자열</span>, (2) <span class="ul">URL 문자열의 변수 값</span>이 된다. 

### POST method

RestTemplate에서는 DELETE 요청을 지원하는 메서드로 postForObject(), postForLocation(), postForEntity()를 제공한다. 
<br/><br/>

**postForObject() 메서드**

```java
public Ingredient createIngredient(Ingredient ingredient) {
    return rest.postForObject("http://localhost:8080/ingredients", ingredient, ingredient.class);
}
```

postForObject()의 매개변수는 차례로 (1) <span class="ul">URL 문자열</span>, (2) <span class="ul">서버에 전송될 객체</span>, (3) <span class="ul">리소스 객체의 타입</span>, (4) <span class="ul">URL 변수 값을 가지는 Map이나 가변 매개변수 리스트</span>가 된다.
<br/><br/>

**postForLocation() 메서드**

```java
public URI createIngredient(Ingredient ingredient) {
    return rest.postForLocation("http://localhost:8080/ingredients", ingredient);
}
```

postForLocation()은 리소스 객체 대신 새로 생성된 리소스의 URI를 반환한다. <br/>
반환된 URI는 해당 응답의 Locaton 헤더에서 얻는다.
<br/><br/>

**postForEntity() 메서드**

```java
public Ingredient createIngredient(Ingredient ingredient) {
    ResponseEntity<Ingredient> responseEntity = 
            rest.postForEntity("http://localhost:8080/ingredients", ingredient, ingredient.class);
    return responseEntity.getBody();
}
```

postForEntity() 메서드를 사용한다면 새로 생성된 리소스의 위치와 리소스 객체 모두 얻을 수 있다.

## Traverson 사용

---

> Traverson은 스프링 데이터 HATEOAS에 같이 제공되며, 스프링에서 <span class="red">하이퍼미디어 API</span>를 사용할 수 있도록 해준다.<br/>
> Traverson을 통해 relation 이름으로 원하는 API를 사용할 수 있다.
> 

### Traverson 객체 생성

```java
Traverson traverson = new Traverson(URI.create("http://location:8080/api"), MediaTypes.HAL_JSON);
```

기본 API URI을 가지는 Traverson 객체를 생성하여 각 링크의 relation 이름으로 API를 사용할 수 있게 한다.<br/>
또한 해당 API가 HAL 스타일의 하이퍼링크를 가지는 JSON 응답을 생성한다는 것을 인자로 지정해준다. - <span class="gray">*수신되는 리소스 데이터를 Traverson이 분석하기 위해*</span>

### Traverson 응용

```java
ParameterizedTypeReference<Resources<Ingredient>> ingredientType = 
                                new ParameterizedTypeReference<Resources<Ingredient>>() {};

Resources<Ingredient> ingredientRes = traverson.follow("ingredients").toObject(ingredientType);
Collection<Ingredient> ingredients = ingredientRes.getContent();
```

Traverson 객체의 follow() 메서드를 호출하면 리소스 링크의 relation 이름이 ingredients인 리소스로 이동할 수 있다. <br/>
클라이언트는 ingredients로 이동했으므로 toObject()를 호출하여 해당 리소스를 가져올 수 있다.<br/>
toObject() 메서드의 인자에는 데이터를 읽어들이는 객체 타입을 지정한다. <br/>
→ 자바에서는 제네릭 타입의 타입 정보(<Ingredient>)가 런타임 시에 소거되므로  ParameterizedTypeReference를 생성하여 리소스 타입을 지정해주어야 한다.

```java
Resources<Taco> tacoRes= traverson.follow("tacos", "recents").toObject(tacoType);
```

follow() 메서드는 이와 같이 두 개 이상의 relation 이름을 인자로 지정하여 한번만 호출할 수 있다.

### Traverson 단점

Traverson을 사용하면 HATEOAS가 활성화된 API를 이동하면서 해당 API 리소스를 쉽게 가져올 수 있지만 <span class="red">**리소스를 쓰거나 삭제하는 메서드를 제공하지 않는다.**</span><br/>
따라서 API의 이동과 리소스 변경/삭제를 모두 해야 한다면 RestTemplate과 Traverson을 함께 사용해야 한다.
<br/><br/>

**RestTemplate과 Traverson 사용 예시**

```java
private Ingredient addIngredient(Ingredient ingredient) {
    String ingredientsUrl = traverson.follow("ingredients").asLink().getHref();

    return rest.postForObject(ingredientsUrl, ingredient, Ingredient.class);
}
```

ingredients 링크를 따라가 asLink()를 통해 ingredients 링크 자체를 요청하고, getHref()로 URL을 가져온다. 해당 URL을 통해 postForObject()를 호출하여 새로운 객체를 추가할 수 있다.