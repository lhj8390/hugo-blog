+++
author = "lhj8390"
title = "Golang - 타입과 메서드 사용하기"
date = "2023-02-21"
description = "golang에서 타입과 메서드의 개념과 사용하는 방법에 대해 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++
## 타입

---

> Go는 내장 타입과 사용자 정의 타입을 가지는 정적 타입 언어이다. 다른 언어들처럼 메서드에 타입을 추가할 수 있고, 타입 추상화도 가능하다.
> 

구조체 타입을 정의한 예시이다.

```go
type Person struct {
  FirstName string
  LastName string
  Age int
}
```햣

구조체 리터럴의 기본 타입을 가지는 Person 이름의 사용자 정의 타입을 선언하였다.

기본타입 / 복합타입 리터럴을 사용하여 구체적인 타입을 정의한 예시이다.

```go
type Score int
type Cnverter func(string)Score
type TeamsScores map[string]Score
```

### 타입은 상속되지 않는다

다른 사용자 정의 타입에 기반한 사용자 정의 타입을 선언할 수 있다.

```go
type HighScore Score
type Employee Person
```

다른 타입 기반으로 타입을 선언하는 것은 두 타입의 기본 타입은 동일하지만 두 타입 간의 계층은 존재하지 않는다. 

Go에서는 명시적 타입 변경 없이 타입 HighScore 인스턴스를 Score 변수로 혹은 그 반대로 할당할 수 없다. 

```go
// 타입이 정해지지 않는 상수를 할당하는 것은 유효하다
var i int = 300
var s Score = 100
var hs HighScore = 200
hs = s                      // 컴파일 오류
s = i                       // 컴파일 오류
s = Score(i)
hs = HighScore(s)
```

## 메서드

---

> Go는 사용자 정의 타입에 대한 메서드를 지원한다.
> 

Go는 구조체에 대해 각 항목에 접근하는 것을 권장하기 때문에 비즈니스 로직(한번의 수행으로 여러 항목 업데이트 등) 을 위해 메서드를 사용한다.

메서드를 선언한 예시이다.

```go
type Person struct {
  FirstName string
  LastName string
  Age int
}

func (p Person) String() string {
  return fmt.Sprintf("%s %s, age %d", p.FirstName, p.LastName, p.Age)
}
```

메서드 선언은 함수 선언과 비슷하지만 리시버(Receiver) 를 명시해야 한다. 리시버는 func 키워드와 메서드 이름 사이에 들어간다. 관례적으로 리시버 이름은 타입 이름의 짧은 약어인 첫 문자를 사용한다. 또한 메서드는 연관된 타입과 동일한 패키지 내에 선언되어야 한다. 

### 포인터 리시버와 값 리시버

리시버의 타입 사용을 결정할 때 규칙이 있다.

1. 메서드가 리시버를 수정한다면 반드시 포인터 리시버를 사용해야 한다.
2. 메서드가 nil 인스턴스를 처리할 필요가 있다면 반드시 포인터 리시버를 사용해야 한다.
3. 메서드가 리시버를 수정하지 않는다면, 값 리시버를 사용할 수 있다.

포인터와 값 리시버를 사용하는 예시이다.

```go
type Counter struct {
  total int
  lastUpdated time.Time
}

func (c *Counter) Increment() {
  c.total++
  c.lastUpdated = time.Now()
}

func (c Counter) String() string {
  return fmt.Sprintf("total: %d, last Updated: %v", c.total, c.lastUpdated)
}
```

```go
var c Counter
fmt.Println(c.String())  // total: 0, last Updated: 0001-01-01
c.Increment()
fmt.Println(c.String())  // total: 1, last Updated: 2023-02-19
```

값 타입인 지역 변수를 포인터 리시버와 함께 사용하면 Go는 자동으로 지역변수 (여기서는 c) 를 포인터 타입으로 변환한다. → `*c.Increment()` 가 `(&c).Increment()` 로 변환된다.*

### nil 인스턴스를 위한 메서드 작성

대부분의 언어에서는 nil 인스턴스로 메서드를 호출했을 때 오류를 발생시킨다. 하지만 Go는 실제 메서드를 실행하려고 노력한다. 메서드가 값 리시버를 가진다면 패닉이 발생하지만, 포인터 리시버를 가질 때 해당 메서드가 nil 인스턴스의 가능성을 처리한다면 제대로 동작한다.

```go
type IntTree struct {
  val int
  left, right *IntTree
}

func (it *IntTree) Insert(val int) *IntTree {
  if it == nil {
    return &IntTree{val: val}
  }
  if val < it.val {
    it.left = it.left.Insert(val)
  } else if val > it.val {
    it.right = it.right.Insert(val)
  }
  return it
}

func (it *IntTree) Contains(val int) bool {
  switch {
  case it == nil:
    return false
  case val < it.val:
    return it.left.Contains(val)
  case val > it.val:
    return it.right.Contains(val)
  default:
    return true
  }
}
```

포인터 리시버는 포인터 함수 파라미터와 동일하게 동작하기 때문에 포인터의 복사본이 변경되지, 원본은 변경이 되지 않는다. 즉, nil을 처리하고 원본 포인터를 nil이 아닌 것으로 바꾸는 메서드를 작성할 수 없다. nil을 확인하고 오류를 반환하는 로직이 이상적이다.

## 객체 구성을 위한 임베딩

---

> 소프트웨어 엔지니어링에서는 클래스 상속보다는 객체 구성을 선호하라고 조언한다. Go는 상속을 가지지 않지만, 구성과 승격을 위한 내장 지원을 통해 코드 재사용을 권장한다.
> 

다음은 임베딩을 하는 예시이다.

```go
type Employee struct {
  Name string
  ID string
}

func (e Employee) Description() string {
  return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}

type Manager struct {
  Employee
  Reports []Employee
}

func (m Manager) FindNewEmployees() []Employee {
  // do business logic
}
```

Manager 가 Employee 타입의 항목을 포함하고 있지만, 해당 항목에 이름이 지정되어 있지 않다. 이를 Employee를 **임베딩**했다고 표현한다. 

임베딩한 항목과 메서드는 승격되어 구조체를 포함하고 바로 실행할 수 있다.

```go
m := Manager{
  Employee: Employee{
    Name: "Bob Bobson",
    ID: "12345",
  },
  Reports: []Employee{},
}

fmt.Println(m.ID)  // 12345
fmt.Println(m.Description()) // Bob Bobson (12345)
```

포함하는 구조체(예를 들어 Manager) 가 동일한 이름의 항목 또는 메소드를 가지면, 임베딩된 항목의 타입을 사용하여 가려진 항목이나 메서드를 참조해야 한다.

```go
type Inner struct {
  X int
}

type Outer struct {
  Inner
  X int
}

o := Outer{
  Innter: Inner{
    X: 10,
  },
  X: 20,
}

fmt.Println(o.Inner.X)  // 10
```

명시적으로 Inner를 지정함으로써 Inner의 X만 접근 가능하다.