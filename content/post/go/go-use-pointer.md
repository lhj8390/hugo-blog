+++
author = "lhj8390"
title = "Golang - 포인터 사용하기"
date = "2023-02-20"
description = "golang에서 포인터의 개념 및 함수에서의 사용법, 주의사항에 대해 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++
## 포인터의 개념

---

> 포인터는 값이 저장된 메모리의 위치 값을 가지고 있는 변수이다.
> 

모든 변수는 하나 이상의 연속된 메모리 공간에 저장되는데, 그것을 **주소**라고 부른다. 포인터는 단순히 다른 변수가 저장된 주소를 내용으로 가지는 변수이다.

```go
var x int32 = 10
var y bool = true
pointerX := &x
pointerY := &y
var pointerZ *string
```

{{<figure src="/images/go-use-pointer/1.png" class="large">}}

서로 다른 타입은 서로 다른 수의 메모리를 차지하지만, 모든 포인터는 어떤 타입을 가리키던 간에 항상 같은 크기를 가진다. `pointerX` 의 값은 x의 주소인 1, `pointerY` 의 값은 y의 주소인 5, `pointerZ` 는 아무것도 가리키고 있지 않기 때문에 0의 값을 가진다. **포인터의 제로 값은 nil이다.**

<br/>

```go
x := 10
pointerToX := &x
fmt.Println(pointerToX)    // 메모리 주소 출력
fmt.Println(*pointerToX)   // 10

z := 5 + *pointerToX
fmt.Println(z)             // 15
```

`&` 는 주소 연산자로, 변수 앞에 `&`를 붙이면 해당 변수의 값이 저장된 메모리 위치의 주소를 반환한다.<br/>
`*`는 간접 연산자로, 포인터 타입 변수 앞에 붙이면 가리키는 값을 반환하고 <span class="red">역참조</span>라고 부른다.

<aside>
포인터 역참조 전에 포인터가 nil이 아닌지 확인이 필요하다. nil을 가진 포인터로 역참조를 시도하면 패닉을 일으킨다.<br/>

</aside>

### 포인터 타입

포인터가 어떤 타입을 가리키는지 나타낸다.

```go
x := 10
var pointerToX *int
pointerToX = &x
```

타입(int) 앞에 *를 사용하여 작성한다. 모든 타입을 기반으로 만들 수 있다.

<br/>

내장 함수 new는 포인터 변수를 생성한다. 

```go
var x = new(int)
fmt.Println(x == nil)  // false
fmt.Println(*x)        // 0
```

제공된 타입의 제로 값을 나타내는 포인터를 반환한다. 

### 기본 타입 리터럴에서의 포인터

기본 타입 리터럴(숫자, 불리언, 문자열)나 상수 앞에는 메모리 주소를 가지지 않기 때문에 &를 사용할 수 없다. 기본타입을 위한 포인터가 필요하면 다음과 같이 사용한다.

```go
var x string
y := &x
```

<br/>

하나의 구조체 내에 기본 타입을 가리키는 포인터가 있다면, 리터럴을 해당 항목에 직접 할당할 수 없다.
```go
type person struct {
  FirstName string
  LastName *string
}

p := person{
  FirstName "Pat",
  LastName: "Perry",  // 컴파일 에러
}
```

이 문제를 해결하려면 두 가지 방법이 존재한다.

1. 변수에 상수의 값을 가지게 한다.
2. 헬퍼 함수를 작성하여 불리언, 숫자, 문자열 타입을 파라미터로 받아 해당 타입의 포인터를 반환한다.

<br/>

```go
func stringp(s string) *string {
  return &s
}

p := person{
  FirstName: "Pat",
  LastName: stringp("Perry"),
}
```

상수를 함수로 전달했을 떄, 상수는 파라미터로 복사되고 변수가 되기 때문에 메모리에서 주소를 가지게 된다. 해당 함수는 결국 변수의 메모리 주소를 반환할 수 있게 된다.

## 함수에서의 사용

---

> Go는 값에 의한 호출을 사용하는 언어이기 때문에 기본타입, 구조체, 배열과 같은 비 포인터 타입들은 호출된 함수에서 원본의 불변성을 보장한다. 하지만 포인터가 함수로 전달되면 호출된 함수에서 원본 데이터를 수정할 수 있다.
> 

포인터를 함수에서 사용할 때 고려할 사항은 다음과 같다.

**1. nil 포인터를 함수로 전달했을 때, 해당 값을 nil이 아닌 값으로 만들 수 없다.**

```go
func failedUpdate(g *int) {
  x := 10
  g = &x
}

func main() {
  var f *int      // f = nil
  failedUpdate(f)
  fmt.Println(f)  // nil
```

예시에서 `f` 변수는 nil을 가지고 시작하는데, failedUpdate를 호출하면 내부에서 `g` 변수가 `x`를 가리키도록 변경되지,  main에 있는 `f` 변수를 변경하지는 않는다. 따라서 failedUpdate가 종료되고 main으로 돌아와도 `f` 변수는 여전히 nil을 가진다. <span class="red">즉, 포인터에 이미 할당된 값이 있는 경우에만 값을 재할당 할 수 있다.</span>

<br/>

**2. 포인터 파라미터에 할당된 값을 유지하려면 포인터를 <span class="red">역참조</span>하여 값을 설정해야 한다.**

```go
func failedUpdate(px *int) {
  x2 := 20
  px = &x2
}

func update(px *int) {
  *px = 20
}

func main() {
  x := 10
  failedUpdate(&x)
  fmt.Println(x)   // 10
  update(&x)
  fmt.Println(x)   // 20
}
```

failedUpdate에서는 **`px`** 포인터가 **`x2`** 의 주소를 가리키도록 변경되지만, 이는 **`x`** 의 주소와는 관련이 없다. 따라서 main으로 돌아왔을 때, main의 `x` 의 값은 변경되지 않는다. 하지만 <span class="red">update 에서는 `x` 의 주소를 가리키고 있는 `px` 파라미터를 역참조하여 값을 수정했기 때문에 main의 `x` 값이 변경된다.</span>

## 포인터 사용시 주의사항

---

> 포인터는 데이터 흐름을 이해하기 어렵게 만들며, 가비지 컬렉터에게 추가적인 작업을 준다.
> 

### 안티 패턴

```go
func MakeFoo(f *Foo) error {
  f.Field1 = "val"
  f.Field2 = 20
  return nil
}
```

구조체 전달을 포인트로 하여 항목을 채우는 것은 좋지 않은 코드이다.

### 모범 패턴

```go
func MakeFoo() (Foo, error) {
  f := Foo{
    Field1: "val",
    Field2: 20,
  }
  return f, nil
}
```

함수 내에서 구조체를 초기화하고 반환하는 것이 좋다.

### 포인터 파라미터를 사용하는 경우

포인터 파라미터를 사용해야 하는 경우는 함수가 해당 포인터를 인터페이스로 예상하는 경우에만 적용된다.

```go
f := struct {
  Name string `json:"name"`
  Age int `json:"age"`
}
err := json.Unmarshal([]byte(`{"name": "Bob", "age": 30}`), &f)
```

충분히 큰 구조체의 경우, 포인터를 이용하여 입력 파라미터 또는 반환값으로 구조체를 사용하여 성능을 향상시킬 수 있다. 모든 데이터 타입을 위한 포인터의 크기가 동일하기 때문이다. 대개의 경우, 포인터를 사용하는 것이 프로그램 성능에 영향을 주지 않지만, 함수 간에 메가바이트 이상의 데이터를 전달하는 경우 고려해볼만 하다.

### 슬라이스를 사용하는 경우

슬라이스는 길이, 수용력, 메모리 블록을 가리키는 포인터, 3개의 항목을 이루는 구조체로 이루어져 있다. 함수로 슬라이스를 넘길 경우 아래와 같은 사진처럼 복제된다. 길이와 수용력을 변경하는 것은 복사본만 변경되지 원본이 바뀔 수가 없다.

{{<figure src="/images/go-use-pointer/2.png" class="large">}}

즉, 슬라이스의 내용을 수정하는 것은 원본에 반영이 되지만 append를 통해 길이를 변경하는 것은 <span class="red">슬라이스의 수용력과 관계 없이 원본에 반영되지 않는다.</span> 기본적으로 함수로 넘겨진 슬라이스는 수정하지 않는 것이 권장된다.