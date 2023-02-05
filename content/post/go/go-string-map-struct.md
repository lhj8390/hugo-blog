+++
author = "lhj8390"
title = "Golang - 복합타입 (맵, 구조체) 사용하기"
date = "2023-02-05"
description = "golang에서 복합타입 중 맵, 구조체의 개념과 사용법, 예제를 설명한다."
tags = ["Go", "Golang"]
categories = [
    "language"
]
subcategories = ["Go"]
+++
## 문자열

---

> 문자열은 바이트 단위로 표현되는데, 문자열의 각 문자는 글자 집합(character set)에 의해 결정된 바이트 크기로 저장된다. Go에서 기본적으로 UTF-8 글자 집합을 사용하여 문자열을 처리한다.
> 
> 
> 또한 문자열은 읽기 전용(immutable)이며, 문자열 내용을 변경하려면 새로운 문자열을 생성해야 한다.
> 

### 문자열 선언

문자열은 다음과 같이 선언할 수 있다.

```go
var s string = "Hello"
var b byte = s[6]

var s2 string = s[:3]  // "Hel"
```

배열이나 슬라이스에서 단일 값을 추출하는 것처럼, 문자열도 인덱스 표현으로 단일 값을 꺼내올 수 있다. 

### 문자열 변환

단일 룬 혹은 바이트는 문자열로 변환이 가능하다.

```go
var a rune    = 'x'
var s string  = string(a)
var b byte    = 'y'
var s2 string = string(b)
```

## 맵

---

> 맵(map)은 키(key)와 값(value)의 쌍으로 구성된 자료구조이다. 보통 엄격하게 증가하는 순서가 아닌 값들을 구성하는 데이터가 있을 때 사용한다.<br/>
> 특정 종류의 데이터를 저장할 때는 편리하지만, 특정 키만 허용하도록 제한하는 방법이 없기 때문에 API에서는 사용하지 않고, 맵의 값으로 동일한 타입을 사용해야 하는 한계점이 있다.
> 
<br/>
맵은 다음과 같은 특징이 있다.

1. 맵의 키는 비교 가능한 타입이 될 수 있다. → 맵의 키는 슬라이스나 맵이 될 수 없다. 
2. 맵은 비교 불가능하다. 맵의 값이 동일한지 비교하기 위해 `==` 혹은 `!=` 를 사용할 수 없다.

### 맵 선언

문자열 타입의 키와 정수를 값으로 가지는 맵을 선언한다.

```go
var map1 map[string]int

// 비어있는 맵 리터럴 할당
map2 := map[string]int{}

// 기본 크기 지정
map3 := make(map[int][]string, 10)

// 비어있지 않은 맵 리터럴
map4 := map[string][]string {
  "Orcas": []string{"Fred", "Ralph", "Bijou"},
  "Lions": []string{"Sarah", "Peter", "Billie"},
  "Kittens": []string{"Waldo", "Raul", "Ze"},
}
```

map1 의 제로 값은 nil이 되고 길이가 0이 된다. map1 에 값을 쓰려고 하면 panic이 발생한다. <br/>
비어있는 맵 리터럴인 map2 는 길이는 0이지만 비어있는 맵 리터럴이 할당된 맵을 읽고 쓸 수 있다.

### 맵 응용

다음과 같이 맵을 읽고 쓸 수 있다.

```go
map1 := map[string]int{}
map1["Orcas"] = 1          // 맵 쓰기
map1["Lions"] = 2
fmt.Println(map1["Orcas"]) // 맵 읽기
```

아직 설정하지 않은 맵 키에 할당된 값을 읽으려고 시도할 경우에는 맵의 값이 되는 타입의 제로 값을 반환한다.<br/><br/>

맵에 키가 있는지 확인하고 싶을 때는 다음과 같이 사용한다.

```go
m := map[string]int{
  "hello": 5,
  "world": 0,
}

v, ok := m["hello"]  // 5 true
v, ok = m["noData"]  // 0 false
```
<br/>

맵에서 값을 삭제하는 코드이다.

```go
m := map[string]int{
  "hello": 5,
  "world": 10,
}

delete(m, "hello")
```

delete 함수는 맵과 키를 받아 해당 키에 해당하는 키-값 쌍을 제거한다. 키가 맵에 없거나 맵이 nil인 경우에는 아무일도 일어나지 않는다.<br/><br/>

맵을 셋(Set)처럼 이용하려면 다음과 같이 사용할 수 있다.

```go
intSet := map[int]bool{}
vals := []int{5, 10, 2, 5, 8, 7, 3, 9, 1, 2, 10}

for _, v := range vals {
	intSet[v] = true
}

fmt.Println(len(vals), len(intSet))
```

Go는 셋을 지원하지 않지만, 맵을 이용해서 셋처럼 사용할 수 있다. 값을 boolean으로 설정하여 키에 대응되는 값으로 true를 설정한다.

## 구조체

---

> 맵의 한계점인 <span class="gray">*“맵의 값으로 동일한 타입만 사용해야 한다”*</span> 를 해결하고 여러 데이터 타입을 함께 구성하고자 할 때 사용한다.
> 

### 구조체 선언

type 키워드로 구조체 타입의 이름을 지정하고 중괄호로 구조체를 정의한다.

```go
type person struct {
  name string
  age  int
  pet  string
}

// 구조체 타입으로 변수 선언
var person1 person
// 구조체 리터럴
person2 := person{}
```

맵 리터럴과는 다르게 항목들 간에 구분을 위한 콤마를 사용하지 않는다.  person1, person2 두 변수 모두 구조체 내에 존재하는 모든 항목들이 <span class="red">각 타입에 맞는 제로 값으로 설정</span>된다.

### 구조체 응용

구조체 타입 이름을 지정하지 않고 구조체 타입을 구현하는 **익명 구조체**를 선언할 수 있다.

```go
var person struct {
  name string
  age  int
  pet  string
}

person.name = "bob"
person.age = 50
person.pet = "dog"

pet := struct {
  name string
  kind string
}{
  name: "Fido",
  kind: "dog",
}
```

person과 pet 변수는 익명 구조체이다. 익명 구조체는 **1)** 외부 데이터를 구조체로 전환 혹은 구조체를 외부 데이터로 전환할 때, **2)** 테스트 작성할 때 주로 사용한다. <br/><br/>

Go에서 다른 기본 타입의 변수들 간의 비교를 허용하지 않는 것처럼, 다른 타입 구성의 구조체의 비교도 허용하지 않는다.

```go
type firstPerson struct {
  name string
  age  int
}

type secondPerson struct {
  name string
  age  int
}

firstPerson == secondPerson // 비교 불가능!!
```

firstPerson 구조체와 secondPerson 구조체는 서로 다른 타입이기 때문에 ==를 사용하여 비교는 불가능하다. 하지만 <span class="red">익명 구조체이면서 두 구조체 모두 같은 이름, 순서, 타입을 가진다면 타입 변환 없이 비교가 가능하다.</span>