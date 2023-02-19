+++
author = "lhj8390"
title = "Golang - 함수 사용하기"
date = "2023-02-19"
description = "golang에서 함수를 선언하고 호출하는 방법, 클로저, defer에 대해 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++
## 함수 선언과 호출

---

> 함수 선언은 네 가지 부분으로 나뉘는데, 키워드 func, 함수의 이름, 입력 파라미터, 마지막으로 타입이다. 다른 언어와 마찬가지로 함수로부터 반환 값이 존재한다면 return을 반환하고 반환할 것이 없다면 return이 있을 필요가 없다.
> 

main 함수는 입력 파라미터와 반환 값이 없다.

```go
func main() {
  result := div(5, 2)
  fmt.Println(result)
}
```

### 이름이 지정된 파라미터와 선택적 파라미터 대응

함수를 위한 파라미터는 예외적인 경우를 제외하고는 호출 시 모두 넘겨져야 한다. 이름이 지정된 파라미터나 선택적 파라미터처럼 사용하고 싶다면 구조체로 만들어 함수로 넘겨줘야 한다.

```go
type MyFuncOpts struct {
  FirstName string
  LastName string
  Age int
}

func MyFunc(opts MyFuncOpts) error {
  // do something
}

func main() {
  MyFunc(MyFuncOpts {
    LastName: "Patel",
    Age: 50,
  })
  MyFunc(MyFuncOpts {
    FirstName: "Joe",
    LastName: "Smith",
  })
```

이름이 지정된 파라미터와 선택적 파라미터를 가지고 있지 않더라도 제한은 아니며, 주로 함수에 입력이 많을 때 유용하게 사용할 수 있다.

### 가변 입력 파라미터와 슬라이스

가변 파라미터는 반드시 입력 파마미터 리스트에 있는 마지막 파라미터여야 한다. 타입 전에 점(…) 3개를 이어붙이면 가변 파라미터가 된다. 

```go
func addTo(base int, vals ...int) []int {
  out := make([]int, 0, len(vals))
  for _, v := range vals {
    out = append(out, base+v)
  }
  return out
}
```

기본 숫자에 가변 파라미터의 각 숫자들을 더해 정수 슬라이스로 반환해주는 코드이다. 가변 파라미터는 슬라이스로 변환되기 때문에 **입력을 슬라이스로 제공**할 수 있다. 하지만 <span class="red">변수나 슬라이스 리터럴 뒤에 3개의 점(…)을 붙여줘야 한다.</span>

### 다중 반환값 허용

Go는 다중 반환값을 허용한다. 

```go
func divAndRemainder(numerator int, denominator int) (int, int, error) {
  if denominator == 0 {
    return 0, 0, errors.New("cannot divide by zero")
  }
  return numerator / denominator, numerator % denominator, nil
}
```

Go 함수가 여러 값을 반환할 때, 반환값의 타입들을 괄호 내에 쉼표(,)로 구분하여 나열한다. 다중 반환값을 사용하여 함수에서 문제가 발생한 경우에는 오류를 반환한다. 함수가 성공적으로 실행되면 오류 값은 nil이 된다. 

<br/>

```go
func main() {
  result, remainder, err := divAndRemainder(5, 2)
  if err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
  fmt.Println(result, remainder)
}
```

함수 호출의 결과로 3가지 다른 변수를 할당할 수 있다. 위의 코드는 결과로 반환되는 값을 `result`, `remainder`, `err` 변수에 할당하도록 했다. <span class="red">파이썬에서는 다중 반환값은 분해된 튜플이기 때문에 하나의 변수에 할당할 수 있지만, Go에서는 함수로부터 반환되는 각각의 값을 변수로 할당해야 한다.</span>

<br/>

암묵적으로 함수에서 반환되는 값들을 모두 무시할 수 있다. 

```go
result, _ := divAndRemainder(5, 2)
```

하나 혹은 그 이상을 읽고 싶지 않은 경우에는 _를 사용하여 명시적으로 무시한다.

### 반환값에 이름 지정

반환값에 대해 이름을 지정할 수 있다. 함수 내에서 반환값을 담기 위해 사용하는 변수를 미리 선언하는 것을 의미한다.

```go
func divAndRemainder(numerator int, denominator int) (result int, remainder int, err error) {
  // 반환값에 값 할당
  result, remainder = 20, 30

  return numerator / denominator, numerator % denominator, nil
}
```

코드를 명확하게 해준다는 장점이 있지만 1) **이름이 지정된 반환값이 섀도잉** 될 수 있고, 2) **해당 변수를 반환할 필요가 없다**는 단점이 존재한다.<br/>
이름이 지정된 반환 파라미터는 <span class="red">반환 값을 담기 위한 변수를 사용하는 것처럼 선언 방법을 제공한 것 뿐</span>이며, 이 값이 할당되지 않았다 해도 반환 시 강제적으로 값을 할당해준다.

<br/>

반환값에 이름을 지정한다면, 심각한 오류인 빈 반환(blank return)을 조심해야 한다. 다음은 빈 반환을 사용하는 예시이다.

```go
func divAndRemainder(numerator int, denominator int) (result int, remainder int, err error) {
  if denominator == 0 {
    err = errors.New("cannot divide by zero")
    return
  }

  result, remainder = numerator / denominator, numerator % denominator
  return
}
```

유효하지 않는 입력일 경우 result와 remainder 반환 값 에 할당된 값이 없기 때문에 해당 변수의 제로 값을 반환한다. 하지만 빈 반환을 사용하게 된다면 어떤 값이 반환되는지 파악하기 힘들기 때문에 좋은 코드가 아니다.

### 함수 타입 선언

함수 타입을 정의하는 데에 type 키워드를 사용할 수 있다.

```go
type opFuncType func(int,int) int
```

```go
var opMap = map[string]opFuncType {
  // ...
}
```

여러 번 참조하려는 경우 이름을 부여할 수 있고 문서화에 용이하다. 또한 인터페이스를 구현할 수 있도록 한다.

### 익명 함수

함수 내에 새로운 함수를 정의하여 변수에 할당할 수 있다. 이름이 없는 내부 함수를 **익명 함수**라고 한다.

```go
func main() {
  for i := 0; i < 5; i++ {
    func(j int) {
      fmt.Println("printing", j)
    }(i)
  }
}
```

func 키워드 뒤에 <span class="red">함수 이름을 제외한</span> 입력 파라미터, 반환값을 넣고 여는 중괄호를 사용하여 선언할 수 있다. <span class="gray">*함수 이름을 넣으면 컴파일 오류가 발생한다.*</span><br/>
익명 함수를 변수에 할당하지 않고 사용하는 경우에는 `defer` 문과 `고루틴`을 사용하는 경우가 유용하다. 코드가 실행되기 전에 함수를 실행하거나, 고루틴을 통해 비동기 작업을 수행할 때 유용하다.

## 클로저

---

> 함수 내부에 선언된 함수를 클로저(closure)라고 한다. 이는 외부 함수에서 선언한 변수를 함수 내부에 선언된 함수에서 접근하고 수정할 수 있다는 것을 의미한다.
> 

클로저를 사용해서 함수의 범위를 제한할 수 있다. 이는 패키지 레벨에 선언 수를 줄여, 사용되지 않는 이름을 쉽게 찾을 수 있도록 한다.

### 파라미터로 함수 전달

슬라이스를 활용하여 클로저를 사용할 수 있다.

```go
sort.Slice(people, func(i int, j int) bool {
  return people[i].LastName < people[j].LastName
})
fmt.Println(people)
```

`sort.Slice` 의 클로저는 두 개의 파라미터만 가지고 있지만, 클로저 내부에서는 외부 변수인 people 을 참조할 수 있다. people 은 클로저에 의해 **캡처**되었다고 표현한다.

### 함수에서 함수 반환

클로저를 사용해서 다른 함수로 어떤 함수의 상태를 넘겨줄 뿐만 아니라, 함수에서 클로저를 반환할 수도 있다.

```go
func makeMult(base int) func(int) int {
  return func(factor int) int {
    return base * factor 
  }
}
```

<br/>

클로저를 반환한 함수는 다음과 같이 사용한다.

```go
func main() {
  twoBase := makeMult(2)
  threeBase := makeMult(3)
  for i := 0; i < 3; i++ {
    fmt.Println(twoBase(i), threeBase(i))
  }
}

// 0 0
// 2 3
// 4 6
```

클로저는 슬라이스를 정렬하거나 효율적으로 검색할 때 유용하게 사용된다. 또한 `defer` 키워드를 통해 자원을 해제하는 로직을 구현할 때도 사용된다. 웹서버를 위한 미들웨어를 만들 때는 클로저를 반환하는 로직을 자주 볼 수 있다.

## defer

---

> 프로그램은 임시 자원을 만드는데, 이런 파일들의 정리는 함수가 성공적으로 완료되었는지 여부에 관계없이 이루어져야 한다. Go에서는 정리 코드에 `defer` 키워드가 들어가게 된다. 다른 언어의 try/catch 문 중 finally와 유사한 개념이다.
> 

defer를 사용하여 자원을 해제하는 예시이다.

```go
func main() {
  f, err := os.Open(os.Args[1])
  if err != nil {
    log.Fatal(err)
  }

  defer f.Close()
  data := make([]byte, 2048)
  for {
    count, err := f.Read(data)
    os.Stdout.Write(data[:count])
    if err != nil {
      if err != io.EOF {
        log.Fatal(err)
      }
      break
    }
  }
}
```

이 예시는 os 패키지의 Open 함수로 읽기 전용 파일 핸들을 얻어오고, 파일에 문제가 발생했다면 오류 메시지가 출력하고 프로그램을 종료한다. 함수가 어떤 식으로 종료되었던 간에 파일을 닫아줘야 하기 때문에 정리 코드 수행을 보장하기 위한 defer 키워드와 함수 호출을 바로 사용한다. <span class="red">defer는 호출하는 함수를 둘러싼 함수가 종료될 때까지 수행을 연기한다.</span>

defer는 Go 함수에서 여러 클로저를 지연시킬 수 있으며 후입선출(마지막이 제일 먼저) 순서로 실행된다.