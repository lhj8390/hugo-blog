+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 리액터 개요"
author = "lhj8390"
description = "스프링에서 리액터에 대한 기초 개념에 대해 설명한다."
date = "2022-08-13"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

## 개요

---

> 스프링인액션 10장을 읽고 리액터에 대한 기초 개념을 공부하였다.
> 
> - 리액티브 프로그래밍 이해
> - 프로젝트 리액터
> - 리액티브 데이터 오퍼레이션
> 
> 에 대해 작성하였다.
> 

## 리액티브 프로그래밍

---

> 어플리케이션을 개발할 때 두 가지 형태의 코드를 작성할 수 있다.
> 
> - 명령형 : 순차적으로 진행되는 작업. 각 작업은 한 번에 하나씩 처리된다.
> - 리액티브 : 일련의 작업들이 정의되지만, 병렬로 실행 가능하다.
> 
> 대부분의 프로그래밍 언어는 스레드를 통해 동시 작업을 처리하지만, 리액티브 프로그래밍은 사용 가능한 데이터가 존재할 때마다 처리할 수 있다. <br/>
> → <span class="red">**확장성과 성능**</span> 관점에서 리액티브 프로그래밍이 유리하다.
> 

### 리액티브 스트림 정의

리액티브 스트림은 차단되지 않는 백 프레셔(backpressure)를 갖는 비동기 스트림 처리의 표준을 제공하는 것이 목적이다.<br/>
어떤 크기의 데이터셋이건 비동기 처리를 지원하고, 실시간으로 데이터를 처리하며, 백 프레셔를 사용하여 데이터 전달 폭주를 막는다.

<aside>
<strong>백프레셔란?</strong>

데이터를 읽는 <span class="red">컨슈머가 처리할 수 있는 만큼 전달 데이터를 제한</span>하여 지나치게 빠른 데이터 소스로부터 데이터 전달 폭주를 피할 수 있는 수단이다.

</aside>

**리액티브 스트림 인터페이스**

- Publisher : 발행자
- Subscriber : 구독자
- Subscription : 구독
- Processor : 프로세서

### Publisher(발행자)

하나의 Subscription(구독)당 하나의 Subscriber(구독자)에 전송하는 데이터를 생성한다.

```java
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> subscriber);
}
```

- **subscribe()** : Subscriber가 Publisher를 구독 신청할 수 있는 메서드

### Subscriber(구독자)

Subscriber(구독자)가 구독 신청되면 Publisher(발행자)로부터 이벤트를 수신할 수 있다.

```java
public interface Subscriber<T> {
    void onSubscribe(Subscription sub);
    void onNext(T item);
    void onError(Throwable ex);
    void onComplete();
}
```

- onSubscribe() : Publisher가 subscribe를 호출한 뒤에 호출되는 메서드이다. 이 메서드의 인자인 **Subscription 객체를 Subscriber에 전달**한다. - <span class="gray">*이때 Subscription 객체는 Subscription.request() 메서드를 통해 받아온 객체이다.*</span>
- onNext() : Subscription.request() 에서 요청하는 데이터를 Publisher에게 전달하는 메서드

### Subscription(구독)

데이터를 어떻게 전송하는 지에 대해 정의한다.

```java
public interface Subscription {
    void request(long n);
    void cancel();
}
```

- request() : 전송되는 데이터를 요청한다. 해당 메서드의 인자는 Subscriber가 받고자 하는 데이터 항목의 수이다. <span class="gray">*- Subscriber가 처리할 수 있는 것보다 많은 데이터를 Publisher가 전송하는 것을 막아준다. (백프레셔)*</span>
- cancel() : 구독을 취소한다는 것을 나타낸다.

### Processor(프로세서)

Subscriber 인터페이스와 Publisher 인터페이스를 결합한 것이다.

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {}
```

processor는 Subscriber 역할로 데이터를 수신, 처리하고, Publisher 역할로 처리 결과를 Subscriber에게 발행한다.

### 전반적인 플로우

{{<figure src="/images/spring-reactor/1.png" class="large" caption="출처 : https://engineering.linecorp.com/ko/blog/reactive-streams-with-armeria-1/">}}



1. Publisher.subscribe() 메서드를 통해 Subscriber가 Publisher에게 구독을 요청한다.
2. Publisher가 onSubscribe(subscription) 을 호출하여 Subscriber에게 Subscription을 전달한다.
3. Subscriber는 Subscription.request()를 통해 데이터 항목 수를 요청한다.
4. Publisher는 Subscription을 통해 Subscriber.onNext()로 데이터를 전달한다.
5. 전달이 성공적이면 onComplete, 오류가 발생한다면 onError를 호출한다.

## 리액터 기본 개념 및 구성

---

> 리액터의 두 가지 핵심 타입으로는 Mono와 Flux가 있는데, 모두 리액티브 스트림의 Publisher 인터페이스를 구현한 것이다.
> 
> - **Mono** : 하나의 데이터 항목만 가지는 데이터셋에 최적화된 리액티브 타입
> - **Flux** : 0, 1 또는 다수의 데이터를 가지는 파이프라인

### 리액티브 플로우

리액티브 플로우는 마블 다이어그램으로 나타낸다.

<aside>
마블 다이어그램의 상단은 <span class="ul">데이터의 타임라인</span>, 중앙에는 <span class="ul">오퍼레이션</span>, 하단에는 <span class="ul">결과로 생성되는 타임라인</span>을 나타낸다.<br/>

</aside>

**Mono의 기본적인 플로우를 보여주는 마블 다이어그램**

{{<figure src="/images/spring-reactor/2.png" class="large" >}}
<br/><br/>

**Flux의 기본적인 플로우를 보여주는 마블 다이어그램**

{{<figure src="/images/spring-reactor/3.png" class="large" >}}

**Mono**는 0 또는 하나의 데이터 항목과 에러를 가지지만 **Flux**는 다수의 데이터를 가지고 있는 것을 알 수 있다.

### 리액터 빌드 명세 추가

```groovy
implementation 'io.projectreactor:reactor-core'
testImplementation 'io.projectreactor:reactor-test'
```

스프링 부트가 dependency 관리를 자동으로 해주기 때문에 버전을 명시할 필요는 없다.<br/>
해당 dependency를 추가함으로써 Mono와 Flux를 사용하여 리액티브 파이프라인을 생성할 수 있다.

## 리액티브 오퍼레이션 적용

---

> Flux와 Mono가 제공하는 오퍼레이션은 두 타입을 함께 결합하여 데이터가 전달될 수 있는 파이프라인을 생성한다.<br/>
> 
> 500 개의 오퍼레이션이 존재하는데, 다음과 같이 분류할 수 있다.
> 
> - 생성 오퍼레이션 (creation)
> - 조합 오퍼레이션 (combination)
> - 변환 오퍼레이션 (transformation)
> - 로직 오퍼레이션 (logic)

### 리액티브 타입 생성

생성 오퍼레이션을 통해 데이터를 발행하는 리액티브 publisher를 생성할 수 있다.

**객체로부터 생성**

```java
@Test
public void createAFlux_just() {
    Flux<String> fruitFlux = Flux.just("Apple", "Orange", "Grape", "Banana");
    fruitFlux.subscribe(
      f -> System.out.println("Here's some fruit: " + f)
    );
}
```

Flux나 Mono의 **just() 오퍼레이션**을 사용하여 리액티브 타입을 생성할 수 있다. <br/>
Subscriber를 추가하기 위해 subscribe() 메서드를 호출한다. - <span class="gray">*호출 즉시 데이터가 전달*</span>
<br/><br/>

**컬렉션으로부터 생성**

```java
@Test
public void createAFlux_fromArray() {
    String[] fruits = new String[]{"Apple", "Orange", "Grape", "Banana"};
    Flux<String> fruitFlux = Flux.fromArray(fruits);
}
```

배열로부터 Flux를 생성하려면 **fromArray() 오퍼레이션**을 호출한다.<br/><br/>

```java
@Test
public void createAFlux_fromIterable() {
    List<String> fruitList = new ArrayList<>();
    fruitList.add("Apple");
    fruitList.add("Orange");
    fruitList.add("Grape");
    fruitList.add("Banana");
    Flux<String> fruitFlux = Flux.fromIterable(fruitList);
}
```

List, Set, Iterable의 다른 구현 컬렉션으로부터 Flux를 생성하려면 **fromIterable() 오퍼레이션**을 호출한다.<br/><br/>

```java
@Test
public void createAFlux_fromStream() {
    Stream<String> fruitStream = Stream.of("Apple", "Orange", "Grape", "Banana");
    Flux<String> fruitFlux = Flux.fromStream(fruitStream);
}
```

자바 Stream 객체를 사용하여 Flux를 생성하려면 **fromStream() 오퍼레이션**을 호출한다.<br/><br/>

**테스트 수행**

```java
StepVerifier.create(fruitFlux)
            .expectNext("Apple")
            .expectNext("Orange")
            .expectNext("Grape")
            .expectNext("Banana")
            .verifyComplete();
```

Flux가 지정되면 StepVerifier는 해당 리액티브 타입(fruitFlux)을 구독한 후 전달되는 데이터에 대해 assertion을 적용한다. 그리고 마지막으로 fruitFlux가 완전한지 검사를 수행한다. 해당 테스트코드는 상단의 모든 Flux 데이터를 검사할 수 있다.<br/><br/>

**카운터 생성**

```java
@Test
public void createAFlux_range() {
    Flux<Integer> intervalFlux = Flux.range(1, 5);

    StepVerifier.create(intervalFlux)
                .expectNext(1)
                .expectNext(2)
                .expectNext(3)
                .expectNext(4)
                .expectNext(5)
                .verifyComplete();
}
```

**range() 오퍼레이션**을 통해 1~5까지의 카운터 Flux를 생성하고 StepVerifier로 검사를 수행한다.<br/><br/>

```java
@Test
public void createAFlux_interval() {
    Flux<Long> intervalFlux = Flux.interval(Duration.ofSeconds(1)).take(5);

    StepVerifier.create(intervalFlux)
                .expectNext(0L)
                .expectNext(1L)
                .expectNext(2L)
                .expectNext(3L)
                .expectNext(4L)
                .verifyComplete();
}
```

**interval() 오퍼레이션**을 통해 매초마다 값을 증가시키는 카운터를 생성한다. <br/>
interval() 는 최대값이 지정되지 않으면 무한으로 실행되기 때문에 take() 오퍼레이션을 사용하여 5개의 항목까지만 실행되도록 제한하였다.

### 리액티브 타입 조합

조합 오퍼레이션을 통해 두 개의 리액티브 타입을 결합하거나 하나의 Flux를 두 개 이상의 리액티브 타입으로 분할할 수 있다.

**리액티브 타입 결합**

```java
@Test
public void mergeFluxes() {
  Flux<String> characterFlux = Flux.just("Garfield", "Kojak", "Barbossa")
                                   .delayElements(Duration.ofMillis(500));
  Flux<String> foodFlux = Flux.just("Lasagna", "Lollipops", "Apples")
                              .delaySubscription(Duration.ofMillis(250))
                              .delayElements(Duration.ofMillis(500));
  
  Flux<String> mergedFlux = characterFlux.mergeWith(foodFlux);

  StepVerifier.create(mergedFlux)
      .expectNext("Garfield")
      .expectNext("Lasagna")
      .expectNext("Kojak")
      .expectNext("Lollipops")
      .expectNext("Barbossa")
      .expectNext("Apples")
      .verifyComplete();
}
```

하나의 Flux를 다른 Flux와 결합하기 위해 **mergeWith() 오퍼레이션**을 사용한다.<br/>
delayElements() 오퍼레이션을 사용하여 500 밀리초마다 일정한 속도로 방출되도록 설정했기 때문에 두 Flux값은 번갈아가며 mergeFlux에 넣어진다. 또한 delaySubscription() 오퍼레이션으로 foodFlux가 250 밀리초 뒤에 구독을 시작했기 때문에 characterFlux 데이터가 먼저 방출되었다.

<span class="red">**→ mergeWith() 는 소스 Flux 값들이 완벽하게 번갈아 방출되게 보장할 수 없다.**</span><br/><br/>

```java
@Test
public void zipFluxes() {
  Flux<String> characterFlux = Flux
      .just("Garfield", "Kojak", "Barbossa");
  Flux<String> foodFlux = Flux
      .just("Lasagna", "Lollipops", "Apples");
  
  Flux<Tuple2<String, String>> zippedFlux = 
      Flux.zip(characterFlux, foodFlux);
  
  StepVerifier.create(zippedFlux)
        .expectNextMatches(p -> 
            p.getT1().equals("Garfield") && 
            p.getT2().equals("Lasagna"))
        .expectNextMatches(p -> 
            p.getT1().equals("Kojak") && 
            p.getT2().equals("Lollipops"))
        .expectNextMatches(p -> 
            p.getT1().equals("Barbossa") && 
            p.getT2().equals("Apples"))
        .verifyComplete();
}
```

**zip() 오퍼레이션**을 사용한다면 각 Flux 소스로부터 한 항목씩 번갈아 가져와 새로운 Flux를 생성한다.<br/>
순서대로 방출되기 때문에 **Tuple2**라는 두 개의 다른 객체를 전달하는 컨테이너 객체를 이용했다.

<aside>
<strong>두 Flux 항목을 문자열로 결합할 경우</strong>

```java
Flux<String> zippedFlux = 
        Flux.zip(characterFlux, foodFlux, (c, f) -> c + " eats " + f);
```

zip() 메서드에 전달되는 함수를 이용하여 두 Flux 항목을 결합할 수 있다.

</aside><br/><br/>

**리액티브 타입 방출 순서 변경**

```java
@Test
public void firstFlux() {
  Flux<String> slowFlux = Flux.just("tortoise", "snail", "sloth")
        .delaySubscription(Duration.ofMillis(100));
  Flux<String> fastFlux = Flux.just("hare", "cheetah", "squirrel");
  
  Flux<String> firstFlux = Flux.first(slowFlux, fastFlux);
  
  StepVerifier.create(firstFlux)
      .expectNext("hare")
      .expectNext("cheetah")
      .expectNext("squirrel")
      .verifyComplete();
}
```

**first() 오퍼레이션**을 통해 먼저 값을 방출하는 Flux 값을 선택할 수 있다.<br/>

### 리액티브 타입 변환

변환 오퍼레이션을 통해 리액티브 스트림을 통해 전달되는 데이터를 변환하거나 필터링할 수 있다.

**리액티브 타입 필터링**

```java
@Test
public void skipAFew() {
  Flux<String> countFlux = Flux.just(
      "one", "two", "skip a few", "ninety nine", "one hundred")
      .skip(3);
 
  StepVerifier.create(countFlux)
      .expectNext("ninety nine", "one hundred")
      .verifyComplete();
}
```

**skip() 오퍼레이션**을 통해 지정된 수(3)의 항목을 건너뛰고 나머지 항목을 방출할 수 있다.<br/>
<span class="ul">특정 시간이 경과할 때까지 기다렸다가</span> Flux 항목을 방출하려면 `skip(Duration.ofSeconds(4))`의 형태로 사용할 수 있다.<br/><br/>

```java
@Test
public void take() {
  Flux<String> nationalParkFlux = Flux.just(
      "Yellowstone", "Yosemite", "Grand Canyon", "Zion", "Acadia")
      .take(3);
 
  StepVerifier.create(nationalParkFlux)
      .expectNext("Yellowstone", "Yosemite", "Grand Canyon")
      .verifyComplete();
}
```

**take() 오퍼레이션**을 사용하면 지정된 수(3)의 항목만을 방출할 수 있다. - <span class="gray">*take() 오퍼레이션의 반대*</span><br/>
skip() 오퍼레이션과 마찬가지로 <span class="ul">특정 시간이 경과할 때까지 기다렸다가</span> Flux 항목을 방출하려면 `take(Duration.ofSeconds(4))`의 형태로 사용할 수 있다.<br/><br/>

```java
@Test
public void filter() {
  Flux<String> nationalParkFlux = Flux.just(
      "Yellowstone", "Yosemite", "Grand Canyon", "Zion", "Grand Teton")
      .filter(np -> !np.contains(" "));
 
  StepVerifier.create(nationalParkFlux)
      .expectNext("Yellowstone", "Yosemite", "Zion")
      .verifyComplete();
}
```

**filter() 오퍼레이션**을 통해 조건식(공백이 없는 문자)에 해당하는 항목만 선택적으로 방출할 수 있다.<br/><br/>

```java
@Test
public void distinct() {
  Flux<String> animalFlux = Flux.just(
      "dog", "cat", "bird", "dog", "bird", "anteater")
      .distinct();
 
  StepVerifier.create(animalFlux)
      .expectNext("dog", "cat", "bird", "anteater")
      .verifyComplete();
}
```

**distinct() 오퍼레이션**을 통해 발행된 적이 없는(고유한) Flux 항목만 방출할 수 있다.<br/><br/>

**리액티브 데이터 매핑**

```java
@Test
public void map() {
  Flux<Player> playerFlux = Flux
    .just("Michael Jordan", "Scottie Pippen", "Steve Kerr")
    .map(n -> {
      String[] split = n.split("\\s");
      return new Player(split[0], split[1]);
    });
  
  StepVerifier.create(playerFlux)
      .expectNext(new Player("Michael", "Jordan"))
      .expectNext(new Player("Scottie", "Pippen"))
      .expectNext(new Player("Steve", "Kerr"))
      .verifyComplete();
}
```

**map() 오퍼레이션**을 통해 발행된 항목을 다른 타입으로 매핑(변환)시킬 수 있다. <br/>
각 항목이 Flux로부터 발행될 때 <span class="red">**동기적으로**</span>(순차적으로) 매핑이 수행된다.<br/><br/>

```java
@Test
public void flatMap() {
  Flux<Player> playerFlux = Flux
    .just("Michael Jordan", "Scottie Pippen", "Steve Kerr")
    .flatMap(n -> Mono.just(n)
        .map(p -> {
            String[] split = p.split("\\s");
            return new Player(split[0], split[1]);
          })
        .subscribeOn(Schedulers.parallel())
      );
  
  List<Player> playerList = Arrays.asList(
      new Player("Michael", "Jordan"), 
      new Player("Scottie", "Pippen"), 
      new Player("Steve", "Kerr"));

  StepVerifier.create(playerFlux)
      .expectNextMatches(p -> playerList.contains(p))
      .expectNextMatches(p -> playerList.contains(p))
      .expectNextMatches(p -> playerList.contains(p))
      .verifyComplete();
}
```

**flatMap() 오퍼레이션**을 subscribeOn()과 함께 사용하면 리액터 타입 변환을 <span class="red">**비동기적으로**</span> 수행할 수 있다. 작업이 병행으로 수행되므로 어떤 작업이 먼저 끝날 지 보장이 되지 않는다.<br/>
<span class="gray">※ flatMap() 오퍼레이션은 수행 도중 생성되는 임시 Flux를 사용하여 변환을 수행하므로 비동기 변환이 가능하다.</span>

<aside>
<strong>subscribeOn()</strong>

각 구독이 병렬 스레드로 수행되어야 한다는 것을 나타낸다. subscribeOn()의 인자로는 사용하기 원하는 동시성 모델(Schedulers)을 지정할 수 있다.

</aside><br/><br/>

**Schedulers의 동시성 모델 종류**

| Schedulers 메서드 | 설명 |
| --- | --- |
| .immediate() | 현재 스레드에서 구독 실행 |
| .single() | 단일의 재사용 가능한 스레드에서 구독 실행. 모든 호출자에 대해 동일한 스레드를 재사용한다. |
| .newSingle() | 매 호출마다 전용 스레드에서 구독 실행 |
| .elastic() | 무한하고 신축성 있는 풀에서 가져온 작업 스레드에서 구독 실행. 필요 시 새로운 작업 스레드 생성 |
| .parallel() | 고정된 크기의 풀에서 가져온 작업 스레드에서 구독 실행. CPU 코어의 개수가 크기가 된다. |
<br/>

**리액티브 데이터 버퍼링**

```java
@Test
public void bufferAndFlatMap() {
    Flux.just(
        "apple", "orange", "banana", "kiwi", "strawberry")
        .buffer(3)
        .flatMap(x -> 
          Flux.fromIterable(x)
            .map(y -> y.toUpperCase())
            .subscribeOn(Schedulers.parallel())   
            .log()
        ).subscribe();

    StepVerifier
        .create(bufferedFlux)
        .expectNext(Arrays.asList("apple", "orange", "banana"))
        .expectNext(Arrays.asList("kiwi", "strawberry"))
        .verifyComplete();
}
```

**buffer() 오퍼레이션**을 통해 데이터 스트림을 분할하여 지정된 최대 크기의 리스트로 된 Flux를 생성한다.<br/>
flatMap()을 같이 사용하면 각 List 컬렉션을 별도의 스레드에서 병행으로 계속 처리할 수 있다.

❗  모든 항목을 List로 모을 경우에는 인자 없이 `buffer()`를 호출한다.<br/><br/>

```java
@Test
public void collectList() {
  Flux<String> fruitFlux = Flux.just("apple", "orange", "banana", "kiwi", "strawberry");
  
  Mono<List<String>> fruitListMono = fruitFlux.collectList();
  
  StepVerifier.create(fruitListMono)
               .expectNext(Arrays.asList(
                            "apple", "orange", "banana", "kiwi", "strawberry"))
               .verifyComplete();
}
```

**collection() 오퍼레이션**을 통해 입력한 Flux가 방출한 모든 메시지를 가지는 List 타입의 Mono를 생성할 수 있다.<br/><br/>

```java
@Test
public void collectMap() {
  Flux<String> animalFlux = Flux.just(
      "aardvark", "elephant", "koala", "eagle", "kangaroo");
  
  Mono<Map<Character, String>> animalMapMono = animalFlux.collectMap(a -> a.charAt(0));
  
  StepVerifier
      .create(animalMapMono)
      .expectNextMatches(map -> {
        return
            map.size() == 3 &&
            map.get('a').equals("aardvark") &&
            map.get('e').equals("eagle") &&
            map.get('k').equals("kangaroo");
      })
      .verifyComplete();
}
```

collectionMap() 오퍼레이션은 Map을 포함하는 Mono를 생성할 수 있다.<br/>
Flux가 방출된 메시지가 Map의 항목으로 지정되며, 각 항목의 키는 collectMap에서 설정한 값을 따른다.

### 로직 오퍼레이션 수행

Mono나 Flux가 발행한 항목이 어떤 조건과 일치하는지 알기 위해 사용한다.

**모든 항목 확인**

```java
@Test
public void all() {
  Flux<String> animalFlux = Flux.just(
      "aardvark", "elephant", "koala", "eagle", "kangaroo");
  
  Mono<Boolean> hasAMono = animalFlux.all(a -> a.contains("a"));
  StepVerifier.create(hasAMono)
              .expectNext(true)
              .verifyComplete();
  
  Mono<Boolean> hasKMono = animalFlux.all(a -> a.contains("k"));
  StepVerifier.create(hasKMono)
              .expectNext(false)
              .verifyComplete();
}
```

**all() 오퍼레이션**을 통해 모든 메시지가 조건에 충족하는지 확인할 수 있다.<br/>
결과는 Boolean 타입의 Mono로 생성되는데, 조건에 충족할 경우 true, 조건에 충족하지 않는다면 false를 방출한다.

**최소한 하나의 항목만 확인**

```java
@Test
public void any() {
  Flux<String> animalFlux = Flux.just(
      "aardvark", "elephant", "koala", "eagle", "kangaroo");
  
  Mono<Boolean> hasAMono = animalFlux.any(a -> a.contains("a"));
  
  StepVerifier.create(hasAMono)
              .expectNext(true)
              .verifyComplete();
  
  Mono<Boolean> hasZMono = animalFlux.any(a -> a.contains("z"));
  StepVerifier.create(hasZMono)
              .expectNext(false)
              .verifyComplete();
}
```

**any() 오퍼레이션**을 통해 최소한 하나의 메시지가 조건을 충족하는지 확인할 수 있다.<br/>
최소한 하나의 메시지가 조건에 충족한다면 true, 모두 조건에 충족하지 않는다면 false를 방출한다.