+++
author = "lhj8390"
title = "Golang - 오류 처리하기"
date = "2023-03-04"
description = "golang에서 오류를 처리하는 방법과 오류 래핑, 패닉과 복구에 대해 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++
## 오류 처리 기초

---

> Go는 함수에 마지막 반환 값으로 error 타입의 값을 반환하여 오류를 처리한다. 함수가 예상했던 대로 수행이 되면 error 파라미터로 nil이 반환된다. 만약 문제가 있다면 오류 값이 반환된다.

함수의 마지막 반환값을 이용하여 오류를 처리하는 예시이다.

```go
func calcRemainderAndMod(numerator, denominator int) (int, int, error) {
  if denominator == 0 {
    return 0, 0, errors.New("denominator is 0")
  }
  return numerator / denominator, numerator % denominator, nil
}
```

새로운 오류는 error 패키지에 있는 New 함수를 호출하면서 문자열과 함께 생성된다. 예외가 있는 언어들과는 달리, Go는 오류가 반환되는 것을 검출하는 특별한 구문이 없다.

<br/>

error 는 단일 메서드를 정의하는 내장 인터페이스이다.

```go
type error interface {
  Error() string
}
```

해당 인터페이스를 구현하는 모든 것은 오류로 간주된다. 

### 오류를 생성하는 방법

Go의 표준 라이브러리는 문자열로 오류를 생성하는 두 가지 방법을 제공한다.

**`errors.New`  함수 사용**

```go
func doubleEven(i int) (int, error) {
  if i % 2 != 0 {
    return 0, errors.New("only even numbers are processed")
  }
  return i % 2, nil
}
```

이 문자열은 반환된 오류 인스턴스의 Error 메서드를 호출했을 때 반환된다. → <span class="gray">*오류를 fmt.Println로 넘기면 자동으로 Error 메서드를 호출한다.*</span>

<br/>

**`fmt.Errorf` 함수 사용**

```go
func doubleEven(i int) (int, error) {
  if i % 2 != 0 {
    return 0, fmt.Errorf("%d isn't an even number", i)
  }
  return i % 2, nil
}
```

이 함수는 fmt.Printf에 대한 모든 포매팅 동사를 사용하여 오류를 생성할 수 있다. `errors.New` 처럼 반환된 오류 인스턴스의 Error 메서드를 호출할 때 문자열이 반환된다.

## 센티넬 오류

---

> 센티넬 오류는 패키지 레벨에 선언된 몇 가지 변수 중 하나이다. 센티넬 오류는 대개 처리를 시작하거나 지속할 수 없음을 나타낼 때 사용한다.
> 

더 이상 처리가 가능하지 않고 오류 상태를 설명할 필요가 없는 특정 상태에 도달했을 때 센티넬 오류로 처리하는 것이 적합하다.

```go
package consterr

type Sentinel string

func(s Sentinel) Error() string {
  return string(s)
}
```

<br/>

위와 같은 package가 있을 경우 다음과 같이 센티널 오류를 사용한다.

```go
package mypkg

const (
  ErrFoo = consterr.Sentinel("foo error")
  ErrBar = consterr.Sentinel("bar error")
)
```

문자열 리터럴을 error 인터페이스를 구현하는 타입으로 전환했다. ErrFoo와 ErrBar의 값 변경은 불가능하다.

## 자신만의 오류 정의

---

> 오류는 인터페이스이기 때문에 로깅이나 오류 처리를 위한 정보를 포함아여 자신만의 오류를 정의할 수 있다. 예를 들어 오류의 일부로 상태 코드를 포함할 수 있다.

<br/>

다음은 커스텀 오류를 정의하는 예시이다.

```go
type Status int

const (
  InvalidLogin Status = iota + 1
  NotFound
)
```

상태 코드를 나타내는 열거형을 정의한다.

<br/>

해당 값을 가지는 StatusErr를 정의한다.

```go
type StatusErr struct {
  Status Status
  Message string
}

func (se StatusErr) Error() string {
  return se.Message
}
```
<br>

다음과 같이 StatusErr를 사용할 수 있다.

```go
func LoginAndGetData(uid, pwd, file string) ([]byte, error) {
  err := login(uid, pwd)
  if err != nil {
    return nil, StatusErr{
      Status: InvalidLogin,
      Message: fmt.Sprintf("invalid credentials for user %s", uid),
    }
  }
  data, err := getData(file)
  if err != nil {
    return nil, StatusErr{
      Status: NotFound,
      Message: fmt.Sprintf("file %s not found", file),
    }
  }
  return data, nil
}
```

사용자 정의 오류 타입을 정의하는 경우에도 반환 타입으로 error를 사용하는 것이 좋다. 함수에서 다양한 오류 타입을 반환할 수 있고 함수 호출자가 특정 오류 타입에 의존하지 않도록 선택할 수 있다.

### 주의할 점

자신만의 오류 타입을 반환할 때 초기화되지 않은 인스턴스를 반환하지 않도록 주의하자.

```go
func GenerateError(flag bool) error {
  var genErr StatusErr
  if flag {
    genErr = StatusErr{
      Status: NotFound,
    }
  }
  return genErr
}

func main() {
  err := GenerateError(true)
  fmt.Println(err != nil)    // true
  err = GenerateError(false)
  fmt.Println(err != nil)    // true
```

err 가 nil 이 아닌 이유는 error는 인터페이스이기 때문이다. 인터페이스가 nil 로 간주되려면 기본 타입과 값이 반드시 nil 이어야 한다. 

<br/>

해당 문제를 고치려면 명시적으로 error 값을 nil 로 반환해주면 된다.

```go
func GenerateError(flag bool) error {
  if flag {
    return StatusErr{
      Status: NotFound,
    }
  }
  return nil
}
```

반환문에 error 변수가 제대로 정의되었는지 코드를 읽을 필요가 없다는 장점이 있다.

<br/>

지역 변수를 error 타입으로 지정해도 해결할 수 있다.

```go
func GenerateError(flag bool) error {
  var genErr error  // error 타입 지정
  if flag {
    genErr = StatusErr{
      Status: NotFound,
    }
  }
  return genErr
}
```

## 오류 래핑

---

> 오류가 코드를 통해 다시 전달될 때, 추가 정보를 넣으면서 오류를 유지하는 것을 **오류 래핑**이라고 한다. 일련의 오류 래핑을 가질 때, **오류 체인**이라고 부른다.
> 

Go 표준 라이브러리에는 오류를 래핑하는 함수인 `fmt.Errorf` 와 오류를 언래핑하기 위한 errors 패키지의 `Unwrap` 함수가 있다.

두 개의 함수를 사용하여 오류를 래핑하고 다시 언래핑하는 예시이다.

```go
func fileChecker(name string) error {
  f, err := os.Open(name)
  if err != nil {
    return fmt.Errorf("in fileChecker: %w", err) // 오류 래핑
  }
  f.Close()
  return nil
}

func main() {
  err := fileChecker("not_exist.txt")
  if err != nil {
    fmt.Println(err)
    if wrapperdErr := errors.Unwrap(err); wrappedErr != nil {
      fmt.Println(wrappedErr) // 오류 언래핑
    }
  }
}

// in fileChecker: open not_exist.txt: no such file or directory
// open not_exist.txt: no such file or directory
```

<br/>

사용자 지정 타입으로 오류를 래핑하려면 Unwrap 메서드를 구현해야 한다.

```go
type StatusErr struct {
  Status Status
  Message string
  err error
}

func (se StatusErr) Error() string {
  return se.Message
}

func (se StatusError) Unwrap() error {
  return se.err
}
```

구현한 Unwrap 메서드는 파라미터가 없고 error를 반환한다.

<br/>

이제 StatusErr로 기본 오류를 래핑할 수 있다.

```go
func LoginAndGetData(uid, pwd, file string) ([]byte, error) {
  err := login(uid, pwd)
  if err != nil {
    return nil, StatusErr {
      Status: InvalidLogin,
      Message: fmt.Sprintf("invalid credentials for user %s", uid),
      Err: err,
    }
  }
  data, err := getData(file)
  if err != nil {
    return nil, StatusErr {
      Status: NotFound,
      Message: fmt.Sprintf("file %s not found", file),
      Err: err,
    }
  }
  return data, nil
}
```

### IS와 AS

오류 래핑은 오류와 관련하여 추가 정보를 얻기 위한 유용한 방법이지만 센티넬 오류가 래핑되었을 때 확인을 위해 `==` 를 사용할 수 없으며 타입 단언이나 타입 스위치를 사용할 수도 없다. Go는 이러한 문제를 해결하기 위해 **IS**와 **AS**를 제공한다.

<br/>

반환된 오류나 래핑된 모든 오류가 센티넬 오류와 일치하는지 확인하려면 `errors.Is` 를 사용한다.

```go
func fileChecker(name string) error {
  f, err := os.Open(name)
  if err != nil {
    return fmt.Errorf("in fileChecker: %w", err)
  }
  f.Close()
  return nil
}

func main() {
  err := fileChecker("not_exist.txt")
  if err != nil {
    if errors.Is(err, os.ErrNotExist) {
      fmt.Println("That file doesn't exist")
    }
  }
}
```

`errors.Is` 함수는 확인될 오류와 대응되는 인스턴스를 파라미터로 받고 센티넬 오류와 일치하는 오류 체인에 해당 오류가 있다면 true를 반환한다.

<br/>

사용자 정의 오류 타입을 지정했을 때 비교 불가능한 타입일 경우 Is 메서드를 구현해야 한다.

```go
type MyErr struct {
  Codes []int
}

func (me MyErr) Error() string {
  return fmt.Sprintf("codes: %v", me.Codes)
}

func (me MyErr) Is(target error) bool {
  if me2, ok := target.(MyErr); ok {
    return reflect.DeepEqual(me, m2)
  }
  return false
}
```

`reflect.DeepEqual` 메서드는 슬라이스를 포함한 모든 것을 비교할 수 있다. 자체 Is 메서드는 동일한 인스턴스가 아닌 오류에 대해서 비교가 가능하게 한다.

<br/>

`errors.As` 함수는 반환된 오류가 특정 타입과 일치하는지 확인할 수 있다.

```go
err := AFunctionThatReturnsAnError()

var myErr MyErr
if errors.As(err, &myErr) {
  fmt.Println(myErr.Code)
}
```

`errors.As` 함수는 두 개의 파라미터를 가지는데, 첫 번째는 검사할 오류이고 두 번째는 찾고자 하는 타입의 변수를 가리키는 포인터이다. 함수가 true를 반환하면 일치하는 오류가 두 번째 파라미터로 할당된다. → <span class="gray">*에러 인스턴스를 해당 에러 타입으로 캐스팅한다.*</span>

결론적으로 특정 에러 인스턴스나 특정 값을 찾을 때 `errors.Is` 를 사용하고 특정 에러 타입을 찾을 때는 `errors.As` 를 사용한다.

### defer 사용

defer를 사용하여 오류를 래핑할 수 있다.

```go
func DoSomeThings(val1 int, val2 string) (_ string, err error) {
  defer func() {
    if err != nil {
      err = fmt.Errorf("in DoSomeThings: %w", err)
    }
  }()

  var3, err := doThing1(val1)
  if err != nil {
    return "", err
  }
  var4, err := doThing2(val2)
  if err != nil {
    return "", err
  }
  return doThing3(val3, val4)
```

지연된 함수에서 err를 참조할 수 있도록 반환 값에 이름을 부여하였다. <span class="red">같은 메시지로 래핑된 모든 오류를 처리할 경우</span> 유용하게 사용할 수 있다.

## 패닉과 복구

---

> Go는 다음에 무슨 일이 일어날지 알 수 없는 상황이 생겼을 경우 패닉을 발생시킨다. 프로그래밍 오류나 환경적인 문제로부터 발생할 수 있다.
> 

패닉이 발생하면 함수는 즉시 종료되고 현재 함수에 연결된 defer 함수가 실행된다. defer가 main 함수에 도달할 때까지 실행되다가 메시지와 스택 트레이스 출력과 함께 종료된다.

### 패닉 생성

프로그램이 복구될 수 없는 상황이라면 직접 패닉을 생성할 수 있다. 

```go
func doPanic(msg string) {
  panic(msg)
}

func main() {
  doPanic(os.Args[0])
}
```

내장 함수 panic 은 어떤 타입이든 하나의 파라미터만 받고 문자열을 사용한다. 

### 패닉 복구

패닉이 발생한 후 프로그램이 비정상적으로 종료되는 것을 막을 수 있다.

```go
func div60(i int) {
  defer func() {
    if v := recover(); v != nil {
      fmt.Println(v)
    }
  }()
  fmt.Println(60 / i)
}

func main() {
  for _, val := range []int{1, 2, 0, 6} {
    div60(val)
  }
}
```

내장 함수 recover는 패닉을 확인하기 위한 defer 내부에서 호출된다. defer 함수를 등록하고 if 문 내에서 recover 함수를 호출하고 nil이 아닌 값인지 확인한다.

<aside>
패닉이 발생하면 defer로 등록된 함수만 실행할 수 있기 때문에 recover는 반드시 defer 내부에서 호출해야 한다.<br/>

</aside>

<br/><br/>