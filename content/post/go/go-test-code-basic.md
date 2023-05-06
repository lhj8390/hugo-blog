+++
author = "lhj8390"
title = "Golang - 테스트 작성하기"
date = "2023-05-06"
description = "golang에서 테스트 코드를 작성하는 기초에 대해 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++
## Go의 테스팅

---

> Go의 표준 라이브러리인 testing 패키지는 테스트를 위한 타입과 함수를 제공하고 go test 도구는 테스트를 실행하고 보고서를 만든다. 다른 언어들과는 다르게 프로덕션 코드와 같은 디렉터리와 패키지에 배치된다.
> 

### 기본 테스트 예제

간단한 함수를 테스트하는 예제이다.

```go
func addNumber(x, y int) int {
    return x + x
}
```
<br/>

이에 대응되는 테스트 코드는 adder/adder_test.go에 있다.


```go
func Test_addNumbers(t *testing.T) {
    result := addNumber(2, 3)
    if result != 5 {
        t.Error("incorrect result: expected 5, got", result)
    }
}
```

모든 테스트는 이름이 `_test.go` 로 끝나는 파일에 작성된다. foo.go라는 파일의 테스트를 작성하려면 foo_test.go 이름의 파일에 작성해야 한다.<br/>
테스트 함수는 Test라는 단어로 시작하고 단일 파라미터로 *test.T 타입을 받는다. `go build` 후 현재 디렉터리의 테스트를 실행하려면 `go test` 명령어를 사용한다.

<br/>

### 테스트 실패 보고

테스트 실패 메시지를 생성하기 위해 Printf 스타일 포멧을 선호한다면 Errorf 메서드를 사용한다.

```go
t.Errorf("incorrect result: expected %d, got %s", 5, result)
```

여러 개의 독립적인 항목을 한 번에 테스팅할 경우 Error/Errorf 를 사용한다.

<br/>

실패가 발견되는 대로 <span class="red">진행을 멈춰야 한다면</span> Fatal과 fatalf 메서드를 사용한다.

```go
t.Fatal("Test failed.")
```

실패 확인 후에도 계속 실패하거나 테스트가 패닉을 유발한다면 Fatal/Fatalf 를 사용한다.

<br/>

### 설정 및 해제

테스트 실행 전 초기화 및 테스트 실행 후 정리 작업을 수행하려면 TestMain 함수를 사용한다.

```go
var testTime time.Time

func TestMain(m *testing.M) {
	fmt.Println("Set up stuff")
	testTime = time.Now()
	exitVal := m.Run()  // 테스트 함수 수행
	fmt.Println("clean up stuff")
	os.Exit(exitVal) // Run()에서 반환된 종료 코드와 함께 반드시 호출해야 한다.
}

func TestFirst(t *testing.T) {
	fmt.Println("TestFirst uses stuff", testTime)
}

func TestSecond(t *testing.T) {
	fmt.Println("TestSecond uses stuff", testTime)
}
```

TestMain은 개별 테스트 이전과 이후에 호출되는 것이 아니라 한 번만 실행된다. 또한 패키지 별로 단 하나만 가질 수 있다. TestMain은 보통 데이터베이스와 같은 데이터 설정이 필요한 경우, 패키지 레벨 변수에 의존적인 코드가 테스트될 때 주로 사용한다.

<br/>

*testing.T의 Cleanup 메서드는 단일 테스트를 위해 생성된 임시 자원을 정리하기 위해 사용한다.

```go
func createFile(t *testing.T) (string, error) {
	f, err := os.Create("tempFile")
	if err != nil {
		return "", err
	}

	t.Cleanup(func() {
		os.Remove(f.Name())
	})
	return f.Name(), nil
}

func TestFileProcessing(t *testing.T) {
	fName, err := createFile(t)
	if err != nil {
		t.Fatal(err)
	}
	// 테스트 수행. 
}
```

테스트가 완료되면 수행된다. 해당 예제와 같이 <span class="red">샘플 데이터를 설정하기 위한 헬퍼 함수에 의존적일 때</span> 유용하게 사용할 수 있다. Cleanup은 여러 번 호출되어도 괜찮고 defer와 같이 마지막에 추가된 것이 처음 호출된다.

## 테이블 테스트

---

> 여러 개의 입력 값을 묶어서 테스트하는 방식이다. 코드의 커버리지를 높이고 중복을 줄일 수 있다.
> 

### 사용 예시

먼저 익명 구조체의 슬라이스를 선언한다. 슬라이스의 각 항목은 서로 다른 테스트를 나타낸다.

```go
data := []struct {
		name     string
		num1     int
		num2     int
		op       string
		expected int
		errMsg   string
}{
		{"addition", 2, 2, "+", 4, ""},
		{"subtraction", 2, 2, "-", 0, ""},
		{"multiplication", 2, 2, "*", 4, ""},
		{"division", 2, 2, "/", 1, ""},
		{"bad_division", 2, 0, "/", 0, `division by zero`},
}

for _, d := range data {
  t.Run(d.name, func(t *testing.T) {
    result, err := DoMath(d.num1, d.num2, d.op)
    if result != d.expected {
      t.Errorf("Expected %d, got %d", d.expected, result)
    }
    var errMsg string
    if err != nil {
      errMsg = err.Error()
    }
    if errMsg != d.errMsg {
      t.Errorf("Expected error message `%s`, got `%s`", d.errMsg, errMsg)
    }
  })
}
```

data 내에 테스트 케이스를 순회하여 매번 Run 메서드를 호출한다. 함수 내부에서는 data의 항목들을 사용하고 동일한 로직을 반복적으로 사용하여 DoMath를 호출한다.

## 코드 커버리지 확인

---

> 코드 커버리지는 테스트 코드 중 어떤 경우를 놓쳤는지를 알 수 있도록 해주는 도구이다. 100% 코드 커버리지에 도달했다고 하더라도 코드에 버그가 없다는 것을 보장하지는 않는다.
> 

go test 명령어에 -cover 플래그를 붙여 커버리지 정보를 계산할 수 있다.

```go
go test -v -cover -coverprofile=c.out
```

-coverprofile 플래그를 포함하면 파일에 커버리지 정보를 저장할 수 있다.

<br/>

cover 도구를 사용하여 커버리지 분석 결과를 HTML 포맷으로 출력할 수 있다.

```go
go tool cover -html=c.out
```

이 명령어를 실행하면 웹 브라우저가 열리고 테스트 된 코드 라인과 테스트 되지 않은 코드 라인을 한눈에 파악할 수 있다.

## 벤치마크

---

> 코드의 성능이나 코드 처리 시간 등을 파악하려면 테스팅 프레임워크에 내장된 벤치마크를 사용할 수 있다. 벤치마크를 작성하여 함수 또는 코드 조각의 처리 시간, 메모리 사용량 등을 측정할 수 있다.
> 

Go에서 벤치마크는 Benchmark라는 단어로 시작하고 타입 *testing.B의 단일 파라미터를 받는다. *testing.B 타입은 *testing.T의 기능 모두와 벤치마크를 위한 추가적인 기능을 제공한다.

```go
var blackhole int

func BenchmarkFileLen1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		result, err := FileLen("testdata/data.txt", 1)
		if err != nil {
			b.Fatal(err)
		}
		blackhole = result
	}
}
```

모든 Go 벤치마크 함수는 0에서 b.N만큼 순회하여 루프를 수행한다. 예시에서 blackhole 변수를 패키지 레벨에 작성하였는데, 이는 컴파일러가 코드를 최적화하여 벤치마크를 망치는 상황을 방지한다.

<br/>

벤치마크를 실행하는 명령어이다.

```go
go test -bench=.
```

## 스텁

---

> 테스트하려는 코드가 다른 코드와 의존적일 경우 스텁이라는 실제 기능을 대체하는 가짜 함수를 만들 수 있다. 실제 함수와 같은 인터페이스를 가지며 미리 정해진 결과를 반환한다.
> 

테스트를 하려는 ProcessExpression 메서드를 정의했다.

```go
type Processor struct {
	Solver MathSolver
}

type MathSolver interface {
	Resolve(ctx context.Context, expression string) (float64, error)
}

func (p Processor) ProcessExpression(ctx context.Context, r io.Reader)(float64, error) {
	curExpression, err := readToNewLine(r)
	if err != nil {
		return 0, nil
	}
	if len(curExpression) == 0 {
		return 0, errors.New("no expression to read")
	}
	answer, err := p.Solver.Resolve(ctx, curExpression)
	return answer, err
}
```

<br/>

다음은 ProcessExpression을 스텁을 사용하여 테스트하는 코드이다. 테스트를 위해 Resolve 메서드를 간단하게 구현한다.

```go
type MathSolverStub struct {}

func (ms MathSolverStub) Resolve(ctx context.Context, expr string) (float64, error) {
	switch expr {
	case "2 + 2 * 10":
		return 22, nil
	case "(2 + 2) * 10":
		return 40, nil
	case "(2 + 2 * 10":
		return 0, errors.New("invalid expression: (2 + 2 * 10")
	}
	return 0, nil
}

func TestProcessorProcessExpression(t *testing.T) {
	p := Processor{MathSolverStub{}}
	in := strings.NewReader(`2 + 2 * 10
(2 + 2) * 10
(2 + 2 * 10`)
	data := []float64{22, 40, 0, 0}
	for _, d := range data {
		result, err := p.ProcessExpression(context.Background(), in)
		if err != nil {
			t.Error(err)
		}
		if result != d {
			t.Errorf("Expected result %f, got %f", d, result)
		}
	}
}
```

MathSolver 인터페이스를 구현하는 MathSolverStub 구조체를 사용하여 스텁을 구현하였다. 이를 통해 MathSolver 인터페이스의 실제 구현체를 작성하지 않고도 Processor 객체가 예상한 대로 동작하는지를 검증할 수 있다.

## httptest

---

> Go에서는 표준 라이브러리인 httptest 패키지를 사용하여 HTTP 핸들러 함수를 테스트할 수 있다.
> 

HTTP 서비스를 호출하는 MathSolver의 구현체이다.

```go
type RemoteSolver struct {
	MathServerURL string
	Client        *http.Client
}

func (rs RemoteSolver) Resolve(ctx context.Context, expression string) (float64, error){
	req, err := http.NewRequestWithContext(ctx, http.MethodGet,
		rs.MathServerURL+"?expression="+url.QueryEscape(expression),
		nil)
	if err != nil {
		return 0, err
	}
	resp, err := rs.Client.Do(req)
	if err != nil {
		return 0, err
	}
	defer resp.Body.Close()
	contents, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return 0, err
	}
	if resp.StatusCode != http.StatusOK {
		return 0, errors.New(string(contents))
	}
	result, err := strconv.ParseFloat(string(contents), 64)
	if err != nil {
		return 0, err
	}
	return result, nil
}
```

이 코드에 대한 테스트 코드를 작성할 예정이다. 

<br/>


<span class="red">함수에 전달된 데이터가 서버에 도착하는지 확인</span>하는 것이 목표이다.

```go
type info struct {
	expression string
	code       int
	body       string
}

var io info
```

테스트 함수에 입력과 출력을 저장하기 위해 info 타입과 입력과 출력이 할당되는 io 변수를 선언한다.

<br/>

가상의 원격 서버를 설정하고 이를 사용하여 RemoteSolver의 인스턴스를 구성한다.

```go
server := httptest.NewServer(
  http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
  expression := req.URL.Query().Get("expression")
  if expression != io.expression {
    rw.WriteHeader(http.StatusBadRequest)
    rw.Write([]byte("invalid expression: " + io.expression))
    return
  }
  rw.WriteHeader(io.code)
  rw.Write([]byte(io.body))
}))
defer server.Close()
rs := RemoteSolver{
  MathServerURL: server.URL,
  Client: server.Client(),
}
```

`NewServer` 함수를 사용하여 임의의 사용하지 않는 포트의 HTTP 서버를 생성하고 시작한다. 또한 요청을 처리하기 위한 http.Handler를 구현한다.

<br/>

테이블 테스트를 사용하여 테스트를 수행한다.

```go
data := []struct {
  name   string
  io     info
  result float64
  errMsg string
}{
  {"case1", info{"2 + 2 * 10", http.StatusOK, "22"}, 22, ""},
  {"case2", info{"( 2 + 2 ) * 10", http.StatusOK, "40"}, 40, ""},
  {"case3", info{"( 2 + 2 * 10", http.StatusBadRequest,
    "invalid expression: ( 2 + 2 * 10"},
	  0, "invalid expression: ( 2 + 2 * 10"},
}
for _, d := range data {
  t.Run(d.name, func(t *testing.T) {
    io = d.io
    result, err := rs.Resolve(context.Background(), d.io.expression)
    if result != d.result {
      t.Errorf("io `%f`, got `%f`", d.result, result)
    }
    var errMsg string
    if err != nil {
      errMsg = err.Error()
    }
    if errMsg != d.errMsg {
      t.Errorf("io error `%s`, got `%s`", d.errMsg, errMsg)
    }
  })
}
```

data 변수에 테스트 케이스를 정의하고 이를 순회하며 RemoteSolver.Resolve 메소드를 호출하고 결과를 검증한다.

<br/>
<br/>
<br/>
<br/>
