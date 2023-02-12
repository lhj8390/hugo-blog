+++
author = "lhj8390"
title = "Golang - 제어문 사용하기"
date = "2023-02-12"
description = "golang에서 제어문의 사용법, 예제를 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++

## if 문
---

> 다른 프로그래밍 언어의 if 문과 매우 유사하다. 가장 큰 차이점은 조건을 감싸는 괄호가 없다는 것이다.
> 

### if 문 선언

기본적으로 다음과 같이 선언한다.

```go
n := rand.Intn(10)

if n == 0 {
  fmt.Println("That's too low")
} else if n > 5 {
  fmt.Println("That's too big:", n)
} else {
  fmt.Println("That's a good number:", n)
} 
```

if 혹은 else 문의 중괄호 내에 선언된 모든 변수는 블록 내에서만 유효하다. 
<br/><br/>

<span class="red">if, else 블록 범위내에서만 사용 가능한 변수</span>를 설정할 수 있다.

```go
if n := rand.Int(10); n == 0 {
  fmt.Println("That's too low")
} else if n > 5 {
  fmt.Println("That's too big:", n)
} else {
  fmt.Println("That's a good number:", n)
} 
```

if/else 문이 마무리되면 n은 더 이상 접근이 되지 않는다. 

## for 문

---

> 다른 언어와 마찬가지로 루프를 위해 for 문을 사용한다. 차이점은 Go에서는 **for가 유일한 반복문 키워드**이다.
> 

### 완전한 for 문

다른 언어의 for 문과 유사하다. if 문 처럼 감싸는 괄호는 없다.

```go
for i := 0; i < 10; i++ {
  fmt.Println(i)
}
```

변수를 초기화하기 위해서는 반드시 := 를 사용해야 한다. → <span class="gray">*var 키워드는 허용하지 않는다.*</span>

### 조건식만 사용하는 for 문

for 문에서 초기값과 증감식을 생략할 수 있다.

```go
for i < 100 {
  fmt.Println(i)
  i = i * 2
}
```

다른 언어의 while 문을 사용하는 것과 비슷하다.

### for 문을 이용한 무한루프

조건식도 사용하지 않고 for 문을 사용하면 무한루프가 된다.

```go
for {
  fmt.Println("Hello")
  if !CONDITION {
    break
  }
}
```

즉시 루프에서 빠져나오려면 break 문을 사용하고 다음 루프로 넘기려면 continue 를 사용한다.

### for-range 문

Go의 어떤 내장 타입의 요소를 순회하며 루프를 수행하는 for 문이다. for-range 문의 각각의 값들은 복합 타입 변수의 값을 복사한 값이므로 <span class="red">for-range 문 내에서 값을 수정하더라도 원본이 변경되지 않는다.</span>

슬라이스를 사용하여 for-range 루프를 만든 코드이다.

```go
evenVals := []int{2, 4, 6}
for i, v:= range evenVals {
  fmt.Println(i, v)
}

// 0 2
// 1 4
// 2 6
```

for-range 루프는 두 개의 변수를 얻을 수 있다. 첫 번째는 현재 순회 중인 값의 위치이고, 두 번째는 해당 위치의 값이다.

<br/>

맵을 사용하여 for-range 루프를 만든 코드이다.

```go
m := map[string]int{
  "a": 1,
  "c": 3,
  "b": 2,
}

for k, v := range m {
  fmt.Println(k, v)
}
```

출력은 <span class="red">맵의 순서와 관계 없이 매번 다르게 나온다.</span> 그 이유는 Go 팀이 해시 도스 공격 같은 문제를 해결하기 위해 해시 알고리즘을 수정하여 맵 변수가 생성될 때마다 무작위 숫자를 포함시켰고, for-range 순회 순서를 루프가 반복될 때마다 조금씩 달라지게 했기 때문이다.

<br/>

문자열을 사용하여 for-range 루프를 만든 코드이다.

```go
samples := "a_π!"
for i, r := range samples {
	fmt.Println(i, r, string(r))
}

// 0 97 a
// 1 95 _
// 2 960 π
// 4 33 !
```

index가 3을 건너뛴 것을 알 수 있다. for-range 루프로 문자열을 순회할 때는 룬을 순회하기 때문에 오프셋은 룬이 가지고 있는 바이트 수에 따라 증가한다.

<br/>

잘 사용하지는 않지만 중첩된 순회문이 있을 경우 바까쪽 루프의 순회를 종료하고 싶을 경우 레이블을 사용한다.

```go
outer:
  for _, outerVal := range outerValues {
    for _, innerVal := rnage outerVal {
      // innerVal 처리
      if invalidSituation(innerVal) {
        continue outer
      }
    }
    // innerVal 처리가 성공적으로 처리되었을 때 수행되는 부분 
  } 
```

## Switch 문

---

> 다른 언어들은 break 문을 명시하지 않는다면 아래의 case 문도 함께 실행되어 switch 문의 사용을 선호하지 않는다. 하지만 Go는 다른 언어와 다르게 break 문을 사용하지 않아도 다음 case 문을 실행하지 않는다는 것을 기본으로 한다.
> 
<br/>

```go
words := []string{"a", "smile", "octopus", "anthropologist"}
for _, word := range words {
  switch size := len(word); size {
  case 1, 2, 3, 4:
    fmt.Println(word, "is a short word!")
  case 5:
    wordLen := len(word)
    fmt.Println(word, "is exactly the right length:", wordLen)
  case 6, 7, 8, 9:
  default:
    fmt.Println(word, "is a long word!")
  }
}  

// a is a short word!
// smile is exactly the right length: 5
// anthropologist is a long word!
```

if 문과 마찬가지로 비교가 되는 값을 감싸는 괄호는 넣을 필요가 없다. 또한 switch 문 내에 어디든 접근 가능한 변수를 선언할 수 있다. 비어 있는 case 문에 대해서는 아무 일도 일어나지 않는다.

<br/>

for문 안에 switch 문이 들어가 있을 경우 for 루프를 빠져나와야 한다면 레이블을 활용할 수 있다.

```go
loop:
  for i := 0; i < 10; i++ P
    switch {
			// 생략..
		case 1:
			break loop // for 문 루프 빠져나오기
    }
```

<br/>

switch 문에서 비교가 되는 값을 명시하지 않는 <span class="red">공백 switch 문</span>을 사용할 수도 있다. 


```go
words := []string{"a", "smile", "octopus", "anthropologist"}
for _, word := range words {
  switch wordLen := len(word); {
  case wordLen < 5:
    fmt.Println(word, "is a short word!")
  case wordLen > 10:
    fmt.Println(word, "is a long word!")
  default:
    fmt.Println(word, "is exactly the right length.")
  }
}  
```

각 case 문에 boolean 결과를 내는 비교는 모두 가능하다.