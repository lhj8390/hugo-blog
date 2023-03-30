+++
author = "lhj8390"
title = "Golang - 인터페이스 사용하기"
date = "2023-03-01"
description = "golang에서 인터페이스의 개념과 사용법, 예제를 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++

## 인터페이스

---

> 인터페이스는 구체화된 객체가 아닌 유일한 추상 타입으로, 타입이 구현해야 하는 메서드들을 정의한다.

인터페이스를 사용하면 코드를 보다 유연하게 하고 사용중인 기능을 정확하게 선언할 수 있다.

<br/>

인터페이스를 선언하는 예시이다.

```go
type Stringer interface {
  String() string
}
```

인터페이스에 의해 정의된 메소드는 인터페이스의 메서드 세트를 호출한다. 인터페이스는 이름 마지막에 `er` 을 붙인다.

### 인터페이스의 특징

Go의 인터페이스는 **암묵적**으로 구현이 된다. 구체 타입은 구현하는 인터페이스를 선언하지 않는다. 이 암묵적인 행동은 인터페이스가 <span class="red">타입 안정성</span>과 <span class="red">디커플링</span>을 가능하게 한다. → <span class="gray">*정적 및 동적 언어의 기능을 연결한다.*</span>

```go
type LogicProvider struct {}

func (lp LogicProvider) Process(data string) string {
  // business logic
}

type Logic interface {
  Process(data string) string
}

type Client struct {
  L Logic
}

func (c Client) Program() {
  // get data rm somewhere
  c.L.Process(data)
}

main() {
  c := Client{
    L: LogicProvider{},
  }
  c.Program()
}
```

Go에서는 인터페이스가 있을 뿐만 아니라 Client 가 해당 사항을 알고 있다. 인터페이스 형태를 만족하는 LogicProvider에게 선언된 것은 없다.

### 임베딩과 인터페이스

구조체에 타입을 임베딩할 수 있는 것처럼 인터페이스에 인터페이스를 임베딩할 수 있다.

```go
type Reader interface {
  Read(p []byte) (n int, err error)
}

type Closer interface {
  Close() error
}

type ReadCloser interface {
  Reader
  Closer
}
```

구조체 내에 인터페이스를 임베딩할 수도 있다.

### 구조체 반환

구조체 대신 인터페이스를 반환한다면 암묵적 인터페이스의 주요 장점인 <span class="red">디커플링을 잃게 된다</span>. 코드가 인터페이스뿐만 아니라 모듈에 영구적으로 의존하기 때문에 향후 유연성을 제한한다. 또한, 인터페이스 대신 구체 타입이 반환된다면 기존 코드에 영향을 주지 않고 새 메서드와 항목을 추가할 수 있다. 하지만 인터페이스에 새로운 메서드를 추가하게 된다면 <span class="red">모든 구현을 업데이트하거나 코드를 망가뜨릴 수 있다.</span>

각기 다른 인터페이스의 인스턴스를 반환하는 단일 팩토리 함수보다는, 각 구체 타입에 맞는 분리된 팩토리 함수를 작성하는 것이 좋다.

<aside>
<b>디커플링</b> : 인터페이스 등을 활용하여 모듈간 의존도를 최소화하여 개발하는 방법<br/>
</aside>

### nil 사용

인터페이스가 nil이라는 것은 타입과 값 모두 nil이어야 한다.

```go
var s *string
fmt.Println(s == nil)  // true
var i interface{}
fmt.Println(i == nil)  // true
i = s
fmt.Println(i == nil)  // false
```

타입 없이 변수를 가질 수 없듯이, 값이 존재하는 포인터 변수가 nil 이 아니라면 해당 타입의 포인터는 항상 nil 이 아니다 즉, <span class="red">타입이 nil이 아닌 한 인터페이스도 nil이 아니다.</span>

인터페이스가 nil이 아니면 해당 인터페이스가 가지는 메서드를 실행할 수 있다. 타입이 nil이 아니더라도 인터페이스와 연관된 값이 nil인지 여부를 확인하기 힘들기 때문에 리플렉션을 사용하여 인터페이스의 값이 nil인지 확인해야 한다.

## 타입 단언 및 타입 스위치

---

> 인터페이스 변수가 특정 구체 타입을 가지고 있거나 구체 타입이 다른 인터페이스를 구현한 것을 확인하는 방법으로는 **타입 단언** 또는 **타입 스위치**가 있다.
> 

### 타입 단언

인터페이스를 구현한 구체 타입의 이름을 지정하거나 인터페이스 기반인 구체 타입에 의해 구현된 다른 인터페이스의 이름을 지정한다.

```go
type MyInt int

func main() {
  var i interface{}
  var mine MyInt = 20
  i = mine
  i2 := i.(MyInt)  // 타입 단언
  fmt.Println(i2 + 1)
}
```

타입 단언이 잘못된다면 `panic: interface conversion: interface {} is main.MyInt, not ''`  같은 패닉을 볼 수 있다. 타입이 기본 타입을 공유하더라도 타입 단언은 기본 값의 타입과 반드시 일치해야 한다. → <span class="gray">*`i.(int)` 도 패닉이 발생한다.*</span>

### 타입 스위치

인터페이스를 여러 가능한 타입 중 하나로 사용할 때, 타입 스위치를 사용한다.

```go
func doThings(i interface{}) {
  switch j := i.(type) {
  case nil:
    // i = nil, j의 타입은 interface{}
  case int:
    // j int
  case MyInt:
    // j MyInt
  default:
    // 무슨 타입인지 알 수 없기 때문에 j의 타입은 interface{}
  }
}   
```

인터페이스가 연관된 타입이 없다는 것을 보기 위해 case 중 하나에 nil을 사용할 수 있다.

### 사용처

타입 단언을 유용하게 사용할 수 있는 방법으로는 인터페이스를 구현한 구체 타입이 다른 인터페이스로도 구현되어 있는지 확인하는 것이다. 이는 **선택적 인터페이스**를 지정할 수 있게 한다. 선택적 인터페이스는 API를 발전시킬 때 효과적으로 사용할 수 있다.

타입 스위치는 주어진 인터페이스 값이나 변수가 구현한 타입에 따라 다른 처리를 할 수 있게 해주는 기능을 제공한다. 인터페이스에 제공될 수 있는 특정 타입이 있는 경우에 가장 유효하다.

## 인터페이스 연결

---

> Go는 사용자 정의 함수 타입을 포함하여 모든 사용자 정의 타입에 메서드를 허용한다. 이것은 함수가 인터페이스를 구현할 수 있도록 한다.

일반적으로 HTTP 처리를 할 때 사용할 수 있다.

```go
type Handler interface {
  ServeHTTP(http.ResponseWriter, *http.Request)
}

type HandlerFunc func(http.ResponseWriter, *http.Request)

func (f HandlerFunc) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  f(w, r)
}
```

타입 변환을 사용하여 http.HandlerFunc로 변환하여 `func(http.ResponseWriter, *http.Request)` 시그니처를 가지는 모든 함수를 http.Handler로써 사용이 가능하다.<br/>
단일 함수 다른 함수나 다른 상태에 의존적인 것 같다면, 인터페이스 파라미터를 사용하고 함수 타입을 선언하여 함수와 인터페이스를 연결하는 것이 낫다.

## 암묵적 인터페이스

---

> **의존성 주입**은 디커플링을 쉽게 하기 위해 개발된 기술 중 하나로, 해당 작업 수행이 필요한 기능을 명시적으로 코드에 지정할 수 있다. Go의 암묵적 인터페이스는 의존성 주입을 쉽게 만들 수 있다.

의존성 주입을 구현하는 예제이다.

```go
func LogOutput(message string) {
  fmt.Println(message)
}

type SimpleDataStore struct {
  userData map[string]string
}

func (sds SimpleDataStore) UserNameForID(userID string) (string, bool) {
  name, ok := sds.userData[userID]
  return name, ok
}

func NewSimpleDataStore() SimpleDataStore {
  return SimpleDataStore{
    userData: map[string]string{
      "1": "Fred",
      "2": "Mary",
    },
  }
}
```

`LogOutput` 이라는 로그를 노출하는 함수와 `SimpleDataStore`이라는 데이터 저장소를 생성했다. 비즈니스 로직이 실행되면 로그를 남겨야 하는데, 나중에 다른 로거나 데이터 저장소를 사용할 수 있기 때문에 의존성을 제거하고 싶다.

<br/>

`Logger` 와 `DataStore` 인터페이스를 정의한다. 또한 `LogOutput` 함수가 Logger 인터페이스를 충족할 수 있도록 메서드와 함수 타입을 정의한다.

```go
type DataStore interface {
  UserNameForID(userID string) (string, bool)
}

type Logger interface {
  Log(message string)
}

type LoggerAdapter func(message string)

func (lg LoggerAdapter) Log(message string) {
  lg(message)
}
```

`LoggerAdapter`와 `SimpleDataStore`는 비즈니스 로직에 필요한 인터페이스를 충족하지만 두 타입은 일치하는지 알 수 없다.

<br/>

비즈니스 로직을 구현한 코드이다.

```go
type SimpleLogic struct {
  l Logger
  ds DataStore
}

func (sl SimpleLogic) SayHello(userID string) (string, error) {
  sl.l.Log("Say Hello for " + userID)
  name, ok := sl.ds.UserNameForID(userID)
  if !ok {
    return "", errors.New("unknown user")
  }
  return "Hello, " + name, nil
}

func NewSimpleLogic(l Logger, ds DataStore) SimpleLogic {
  return SimpleLogic{
    l: l,
    ds: ds,
  }
}
```

Logger와 DataStore 항목을 가지는 구조체를 정의했다. `SimpleLogic` 에는 <span class="red">구체 타입을 언급하는 내용이 없으므로 의존성이 없다.</span> 나중에 새로운 구현으로 바꾸더라도 공급자가 해당 인터페이스와 연관이 없기 때문에 아무런 문제가 없다.

<br/>

main 함수에서 모든 컴포넌트를 구성한다.

```go
func main() {
  l := LoggerAdapter(LogOutput)
  ds := NewSimpleDataStore()
  logic := NewSimpleLogic(l, ds)
  fmt.Println(logic.SayHello("1"))
}
```

main 함수는 <span class="red">모든 구체 타입이 실제 무엇인지 알고 있는 유일한 부분</span>이다. 만약 다른 구현으로 교체하려면 main 함수에서만 변경하면 된다. 의존성 주입은 테스트를 더 쉽게 만들 수 있는 훌륭한 패턴이다.

<br/>