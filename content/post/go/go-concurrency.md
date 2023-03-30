+++
author = "lhj8390"
title = "Golang - 동시성 사용하기"
date = "2023-03-30"
description = "golang에서 동시성 사용과 함께 고루틴, 채널, 뮤텍스를 사용하는 방법를 설명한다."
tags = ["Go", "Golang"]
categories = ["language"]
subcategories = ["Go"]
+++

## 동시성

---

> 동시성은 단일 프로세스를 독립적인 컴포넌트로 분리하고 해당 컴포넌트가 안전하게 데이터를 공유하는 방법을 지정한다. 대부분의 언어는 Lock을 획득하여 공유 데이터에 접근하지만 Go는 CSP(순차적 프로세스 통신)에 기반한다.
> 

### 동시성 사용 시점

프로그램에서 동시성 사용의 여부는 단계에 따른 데이터 흐름, 문제의 복잡도, 실행 시간 등에 따라 결정된다. 독립적으로 수행할 수 있는 여러 처리로부터 데이터를 결합시키려면 동시성을 사용하는 것이 좋은 방법이다. <span class="red">하지만, 동시성은 불필요한 오버헤드를 발생시키기 때문에, 동시에 실행되는 시간이 매우 짧을 때는 동시성을 사용하는 것은 좋지 않다.</span> 이 경우에는 일반적인 순차적인 처리 방식으로 구현하는 것이 더 효율적일 수 있다.

보통 동시성은 I/O 작업을 위해 사용된다. 디스크나 네트워크로부터 읽거나 쓰기는 인메모리 처리보다 천 배정도 느리기 때문이다.

## 고루틴

---

> 고루틴은 Go 런타임에서 관리하는 가벼운 **프로세스**로 Go 동시성 모델의 핵심 개념이다.
> 

Go 런타임은 여러 스레드를 생성하고 프로그램을 실행하기 위해 단일 고루틴을 시작한다. Go 런타임 스케줄러는 해당 고루틴을 실행하기 위해 적절한 스레드를 할당하며, 고루틴이 종료될 때는 해당 스레드를 반환한다.

### 고루틴의 장점

1. 운영체제 자원을 생성하지 않기 때문에 스레드 생성보다 생성 속도가 빠르다.
2. 고루틴은 메모리를 더 효율적으로 사용할 수 있다.
3. 고루틴 간의 전환은 프로세스 내부에서만 일어나기 때문에 스레드 사이의 전환보다 빠르다.
4. 스케줄러는 Go 프로세스 일부이기 때문에 스케줄링 결정을 최적화할 수 있다.

## 채널

---

> 고루틴은 채널을 통해 통신한다. make 함수를 사용하여 생성할 수 있는 내장 타입이다.
> 

채널을 생성하는 예시이다.

```go
ch := make(chan int)
```

참조 타입이기 때문에 채널을 함수로 전달하면 실제로 채널에 대한 포인터를 전달한다. 제로 값은 nil 이다.

### 읽기, 쓰기

`<-` 연산자를 사용하여 채널과 상호작용할 수 있다. 

```go
a := <- ch  // ch 에서 값을 읽는다.
ch <- b     // ch 에 b의 값을 쓴다.
```

채널 변수 왼쪽에 `<-` 연산자를 두면 <span class="orange">*채널로부터 데이터를 읽고*</span> 채널 변수 오른쪽에 `<-` 연산자를 두면 <span class="orange">*채널에 데이터를 쓸 수 있다.*</span>

<aside>
💡 <span class="red">하나의 고루틴으로 같은 채널을 읽고 쓰는 것은 불가능하다.</span> 읽기 전용이라는 것을 나타내기 위해 chan 키워드 앞에 `<-` 연산자를 (<span class="gray">ch ←chan int</span>), 쓰기 전용이라는 것을 나타내기 위해 chan 키워드 뒤에 `<-` 연산자를 (<span class="gray">ch chan← int</span>) 사용한다.<br/>

</aside>

### 버퍼링

기본적으로 채널은 <span class="red">버퍼가 없다</span>. 버퍼가 없는 채널에서는 데이터를 보내는 고루틴과 데이터를 받는 고루틴이 동시에 실행되지 않고, 보내는 고루틴이 데이터를 보낸 후에는 받는 고루틴이 데이터를 가져갈 때까지 대기한다.

버퍼가 있는 채널은 블로킹(<span class="gray">작업이 완료될 때까지 대기</span>) 없이 제한된 쓰기의 버퍼를 가진다. 버퍼가 다 채워지면, 채널이 읽어질 때까지 쓰기 고루틴은 일시 중지된다.

```go
ch := make(chan int, 10)
```

버퍼가 있는 채널을 생성하는 예시이다. 버퍼의 수용력을 지정하여 만들 수 있다.

### for-range

for-range 루프를 사용하여 채널의 값을 읽을 수 있다.

```go
for v := range ch {
  fmt.Println(v)
}
```

채널이 닫힐 때까지나 break, return 문에 도달할 때까지 루프는 지속된다.

### 채널 닫기

채널에 쓰기를 완료했을 때 close 내장 함수를 이용하여 채널을 닫을 수 있다.

```go
close(ch)
v, ok := <-ch  // 채널에서 값을 읽을 때 ok로 채널의 닫힘 유무 확인 가능
```

닫힌 채널에 쓰기를 시도하거나 다시 닫으려 하면 패닉이 발생하지만, <span class="red">읽기를 시도하는 것은 가능하다.</span> 채널이 버퍼가 없거나 버퍼가 있는 채널에 값이 없다면 제로 값을 반환한다. 채널에서 값을 읽을 때는 채널이 닫혀있는지 여부를 확인하는 것이 좋다.

### 채널 동작 방식

|  | 버퍼 X, 열림 | 버퍼 X, 닫힘 | 버퍼 O, 열림 | 버퍼 O, 닫힘 | Nil |
| --- | --- | --- | --- | --- | --- |
| 읽기 | 써질 때까지 일시중지 | 제로 값 반환 | 버퍼가 빌 때까지 <br/>일시중지 | 버퍼에 남은 값을 반환하고 <br/> 비어 있으면 제로 값 반환 | 무한정 대기 |
| 쓰기 | 읽을 때까지 일시중지 | 패닉 | 버퍼가 가득 찰 때까지  <br/>일시중지 | 패닉 | 무한정 대기 |
| 닫기 | 동작 | 패닉 | 동작 | 패닉 | 패닉 |

## select 문

---

> 여러 채널 작업을 모니터링하고, 그 중에서 준비된 작업을 수행하는 방법을 제공하는 제어 구문이다.
> 

select 문을 사용하는 예시이다.

```go
select {
case v := <-ch:
  fmt.Println(v)
case v := <-ch2:
  fmt.Println(v)
case ch3 <- x:
  fmt.Println("wrote", x)
case <-ch4:
  fmt.Println("got value on ch4, but ignored it")
}
```

select 문의 각 case는 채널에 읽기나 쓰기를 수행하는데, 읽기나 쓰기가 가능한 case에서 블록이 실행된다. select 문 내에서 여러 채널이 읽거나 쓸 수 있다면 <span class="red">순서와 상관없이 여러 case 중 하나를 임의로 선택한다.</span>

<br/>

select 문의 장점은 교착상태를 방지할 수 있다는 것에 있다.

```go
func main() {
  ch1 := make(chan int)
  ch2 := make(chan int)
  go func() {
    v := 1
    ch1 <- v
    v2 := <-ch2
    fmt.Println(v, v2)
  }()

  v := 2
  var v2 int
  select {
  case ch2 <- v:
  case v2 = <-ch1:
  }
  fmt.Println(v, v2)
}
```

select 구문은 여러 채널에서 대기하다가 준비된 작업이 있으면 해당 작업을 처리하므로, 교착 상태를 방지할 수 있다.

## 동시성 사례와 패턴

---

> 동시성은 Go 언어의 강력한 기능 중 하나이며, 이를 올바르게 활용함으로써 효율적인 프로그램을 작성할 수 있다.동시성을 적용한 프로그램을 작성할 때 자주 사용되는 Go의 동시성 패턴과 사례를 설명한다.
> 

### 루프 변수 캡처 문제

고루틴을 사용할 때 for 루프의 값이나 인덱스를 캡처하면 예상치 못한 오류가 발생할 수 있다.

```go
func main() {
  a := []int{2, 4, 6}
  for _, v := range a {
    go func() {
      fmt.Println(v)
    }()
  }
}

// 6
// 6
// 6
```

고루틴이 시작할 때 이미 v의 값이 6으로 변경되어 있기 때문에 이런 결과가 출력된다. for 루프뿐만 아니라 값이 바뀔 수 있는 변수에 의존하고 있는 고루틴이라면 이런 오류가 생길 수 있다.

<br/>

이 문제를 해결하려면 1) <span class="ul">루프 내에 값을 섀도잉하거나</span> 2) <span class="ul">파라미터로 값을 넘겨줘야 한다.</span>

```go
// 값 섀도잉
for _, v := range a {
  v := v
  go func() {
    fmt.Println(v)
  }()
}

// 파라미터로 값 전달
for _, v := range a {
  go func(val int) {
    fmt.Println(val)
  }(v)
}
```

### 고루틴 정리

고루틴이 종료되지 않는다면 스케줄러는 해당 고루틴에게 주기적으로 시간을 할당하므로 프로그램이 느려질 수 있다. (<span class="gray">*고루틴 leak*</span>) 일반적으로 값들의 모두를 사용하는 경우 고루틴은 종료한다.

다음은 일반적으로 고루틴이 종료되는 예시이다.

```go
func countTo(max int) <-chan int {
  ch := make(chan int)
  go func() {
    for i := 0; i < max; i++ {
      ch <- i
    }
  }()
  return ch
}

func main() {
  for i := range countTo(10) {
    fmt.Println(i)
  }
}
```

for-range 문에서 모든 값들을 순회하여 고루틴을 사용하기 때문에 순회가 끝난 후 고루틴은 정상 종료된다.

<br/>

메모리 누수가 발생하는 예시이다.

```go
func countTo(max int) <-chan int {
  ch := make(chan int)
  go func() {
    for i := 0; i < max; i++ {
      ch <- i
    }
  }()
  return ch
}

func main() {
  for i := range countTo(10) {
    if i > 5 {
      break  // 루프를 빠르게 종료
    }
    fmt.Println(i)
  }
}
```

루프를 빠르게 종료하여 고루틴은 채널에서 값을 읽기 위해 영원히 기다리게 된다.

### Done 채널 패턴

Done 채널 패턴은 처리를 종료해야 하는 시점을 고루틴에게 알려준다. 

```go
func searchData(s string, searchers []func(string) []string) []string {
  done := make(chan struct{})
  result := make(chan []string)
  for _, searcher := range searchers {
    go func(searcher func(string) []string) {
      select {
      case result <- searcher(s):
      case <-done:
      }
    }(searcher)
  }
  r := <-result
  close(done)
  return r
}
```

여기서 done 채널은 searchData 함수를 호출한 측에서 함수의 실행을 강제로 종료시키기 위한 용도로 사용된다.

해당 코드는 다음과 같이 작동한다.

1. 고루틴은 searcher 함수를 실행하고 검색 결과를 result 채널에 보내거나 done 채널이 닫힐 때까지 기다린다.
2. 가장 먼저 결과를 반환한 searcher 함수의 검색결과를 r 변수에 저장한다.
3. 모든 고루틴이 종료되도록 done 채널을 닫는다. (<span class="gray">*select 문에서 <-done을 수신하고 모든 고루틴은 종료된다*</span>)
4. 검색 결과를 슬라이스(r)로 반환한다.

<span class="red">여러 검색 함수(searchers) 를 동시에 실행하고 가장 먼저 결과를 반환한 검색 함수(searcher) 의 결과만 반환</span>한다. 이를 통해 검색 속도를 높일 수 있다. 

### 고루틴 종료를 위한 취소 함수 사용

done 패턴을 사용하여 취소 함수를 함께 반환하는 패턴을 구현할 수도 있다.

```go
func countTo(max int) (<-chan int, func()) {
  ch := make(chan int)
  done := make(chan struct{})
  cancel := func() {
    close(done)
  }
  go func() {
    for i := 0; i < max; i++ {
      select {
      case <-done:
        return
      default:
        ch <- i
      }
    }
    close(ch)
  }()
  return ch, cancel
}

func main() {
  ch, cancel := countTo(10)
  for i := range ch {
    if i > 5 {
      break
    }
    fmt.Println(i)
  }
  cancel()
}
```

countTo 함수의 두 개의 채널은 1) <span class="ul">데이터를 반환</span>하고, 2) <span class="ul">종료 시그널을 보낸다.</span> done 채널을 직접 반환하는 것보다 done 채널을 닫는 클로저를 생성하여 대신 반환하는 것이 더 낫다. 

### 버퍼 사용 시점

버퍼가 있는 채널은 얼마나 많은 고루틴이 실행될 지를 알고 있을 때, <span class="red">실행시킬 고루틴의 개수를 제한하거나 대기 중인 작업의 양을 제한하려는 경우</span>에 유용하다. 시스템에서 대기중인 작업의 양을 관리하여 서비스가 뒤처지거나 과부하가 걸리는 것을 방지한다.

```go
func processChannel(ch chan int) []int {
  const conc = 10
  results := make(chan int, conc)
  for i := 0; i < conc; i++ {
    go func() {
      results <- process(v)
    }()
  }
  var out []int
  for i := 0; i < conc; i++ {
    out = append(out, <-results)
  }
  return out
}
```

정확히 몇 개의 고루틴이 실행되어야 하는지 알고 있고 각 고루틴은 작업이 끝나는 대로 종료된다. 값을 채널에 보낼 때마다 버퍼가 차지 않았다면 송신자는 계속해서 값을 보낼 수 있지만, 버퍼가 가득 찼다면 송신자는 채널이 값을 수신할 때까지 블록된다. 따라서, 채널에 대한 송수신 작업이 서로 다른 속도로 발생하는 경우에도 송신자가 계속해서 값을 보낼 수 있다.

### 배압 (backpressure)

버퍼가 있는 채널과 구현될 수 있는 다른 기술은 **배압**이다. 버퍼가 있는 채널을 사용하고 select문으로 시스템에 동시에 들어오는 요청 수를 제한할 수 있다.

```go
type PressureGauge struct {
  ch chan struct{}
}

func New(limit int) *PressureGauge {
  ch := make(chan struct{}, limit)
  for i := 0; i < limit; i++ {
    ch <- struct{}{}
  }
  return &PressureGauge {
    ch: ch,
  }
}

func (pg *PressureGauge) Process(f func()) error {
  select {
  case <-pg.ch:
    f()
    pg.ch <- struct{}{}
    return nil
  default:
    return errors.New("no more capacity")
  }
}

func main() {
  pg := New(10)
  http.HandleFunc("/request", func(w http.ResponseWriter, r *http.Request) {
    err := pg.Process(func() {
      w.Write([]byte("done"))
    })
    if err != nil {
      w.WriteHeader(http.StatusTooManyRequests)
      w.Write([]byte("Too many request"))
    }
  })
  http.ListenAndServe(":8080", nil)
}
```

고루틴은 Process 함수를 사용하길 원하고 select는 채널로부터 토큰을 읽으려고 한다. 토큰을 읽는다면 함수를 실행하고 토큰이 반환된다. 토큰을 읽을 수 없으면 토큰 대신 오류를 반환한다.

### select에서 case 문 해제

select case 중 하나가 닫힌 채널을 읽는다면 항상 성공하고 제로 값을 반환한다. 따라서 값이 유효한지 검사를 해야하고 case를 건너뛰어야 한다. select에서 case를 비활성화하기 위해서는 nil 채널을 활용할 수 있다.

```go
for {
  select {
  case v, ok := <-in:
    if !ok {
      in = nil // 해당 case는 더 이상 성공할 수 없음
      continue
    }
  case v, ok := <-in2:
    if !ok {
      in2 = nil // 해당 case는 더 이상 성공할 수 없음
      continue
    }
  case <-done:
    return
  }
}
```

채널이 닫혔다는 것을 감지했을 때 채널 변수를 nil로 설정한다. 연관된 case 문은 nil 채널에서 값을 반환하지 않기 때문에 더 이상 실행되지 않는다.

### 타임아웃 처리

동시성 처리를 통해 하나의 요청을 수행하는 시간을 관리할 수 있다. 

```go
func timeLimit() (int, error) {
  var result int
  var err error
  done := make(chan struct{})
  go func() {
    result, err = doSomeWork()
    close(done)
  }()
  select {
  case <-done:
    return result, err
  case <-time.After(2 * time.Second):
    return 0, errors.New("work timed out")
  }
}
```

result와 err에 값을 할당하고 done 채널을 닫기 위해 클로저를 고루틴으로 사용했다. done 채널이 닫히면, done 채널로부터 값을 성공적으로 읽게 된다. doSomeWork 함수가 끝나기 전에 `time.After(2 * time.Second)` case 값이 읽힌다면 timeLimit은 타임 오류를 반환한다.

### WaitGroup 사용

단일 고루틴을 기다리려면 done 채널 패턴을 사용하지만 몇몇 고루틴을 기다려야 한다면 표준 라이브러리 sync 패키지의 **WaitGroup**을 사용한다.

```go
func processAndGather(in <-chan int, processor func(int) int, num int) []int {
  out := make(chan int, num)
  var wg sync.WaitGroup
  wg.Add(num) // 고루틴의 수를 미리 알림
  for i := 0; i < num; i++ {
    go func() {
      defer wg.Done() // 고루틴이 패닉에 빠지더라도 호출되는 것을 보장하기 위해 defer를 사용
      for v := range in {
        out <- processor(v)
      }
    }()
  }
  go func() {
    wg.Wait()  // 모든 고루틴이 완료될 때까지 대기
    close(out)
  }()
  var result []int
  for v := range out {
    result = append(result, v)
  }
  return result
}
```

여러 고루틴이 같은 채널에 쓰기를 하는 경우 쓰기를 위한 채널은 단 한번만 닫혀지는 것을 보장해야 한다. 이를 위한 해결책이 WaitGroup이다.

WaitGroup이 제공하는 메서드는 다음과 같다.

- Add : 대기할 고루틴의 카운터 증가
- Done : 완료 시점에 고루틴에 의해 호출되어 카운터 감소
- Wait : 카운터가 0이 될 때까지 고루틴 일시중지

<aside>
💡 WaitGroup은 사용하기 쉽지만 고루틴을 설계할 때 무조건 사용해서는 안된다. 작업 고루틴이 종료하고 나서 정리할 필요가 있을 경우에만 사용하는 것이 좋다.<br/>

</aside>

### 한 번만 코드 실행

지연 로딩을 하거나 초기화 코드를 정확히 한 번만 호출하고 싶을 경우 sync 패키지의 Once 타입을 사용한다.

```go
type SlowComplicatedParser interface {
  Parse(string) string
}

var parser SlowComplicatedParser
var once sync.Once

func Parse(dataToParse string) string {
  once.Do(func() {
    parser = initParser()
  })
  return parser.Parse(dataToParse)
}

func initParser() SlowComplicatedParser {
  // 설정 또는 로딩 수행
}
```

WaitGroup과 같이 sync.Once의 인스턴스를 설정할 필요는 없다. 예제에서는 once의 Do 메서드로 전달된 클로저 내부에서 parser 값을 설정했다. Parse가 한 번 이상 호출된다면 once.Do는 클로저를 다시 실행하지 않는다.

## 뮤텍스 사용

---

> 뮤텍스는 공유된 데이터의 접근이나 어떤 코드의 동시 실행을 제한한다. 뮤텍스를 사용하는 일반적인 경우는 고루틴이 공유된 값을 읽거나 쓰는 경우이지만 값을 처리하지는 않는다.
> 

### 채널 사용 예시

채널을 사용하여 공유된 값을 읽거나 쓰는 예시이다.

```go
func scoreboardManager(in <-chan func(map[string]int), done <-chan struct{}) {
  scoreboard := map[string]int{}
  for {
    select {
    case <-done:
      return
    case f := <-in:
      f(scoreboard)
    }
  }
}
```

하나의 채널로 맵을 읽거나 수정하는 함수를 기다리고 두번째 채널에서 종료 시점을 알린다.

<br/>

```go
type ChannelScoreboardManager chan func(map[string]int)

func NewChannelScoreboardManager() (ChannelScoreboardManager, func()) {
  ch := make(ChannelScoreboardManager)
  done := make(chan struct{})
  go scoreboardManager(ch, done)
  return ch, func() {
    close(done)
  }
}

func (csm ChannelScoreboardManager) Update(name string, val int) {
  csm <- func(m map[string]int) {
    m[name] = val
  }
}
```

업데이트 메서드는 굉장히 직관적이지만 값을 읽기 위해서는 scoreboardManager에 전달되는 함수를 기다리는 done 패턴을 사용해야 한다.<br/>
해당 코드는 번거롭고 한 번에 하나의 읽기만 허용된다는 단점이 있다. 이를 개선하기 위해서는 뮤텍스를 사용한다.

### 뮤텍스 사용 예시

sync 패키지에는 두 개의 뮤텍스 구현(<span class="gray">Mutex, RWMutex</span>)이 있다. 
- Mutex : Lock과 Unlock 메서드 존재
- RWMutex : 읽기 잠금(RLock, RUnlock), 쓰기 잠금(Lock, Unlock) 존재

```go
type MutexScoreboardManager struct {
	l          sync.RWMutex
	scoreboard map[string]int
}

func NewMutexScoreboardManager() *MutexScoreboardManager {
	return &MutexScoreboardManager{
		scoreboard: map[string]int{},
	}
}

func (msm *MutexScoreboardManager) Update(name string, val int) {
	msm.l.Lock()
	defer msm.l.Unlock()
	msm.scoreboard[name] = val
}

func (msm *MutexScoreboardManager) Read(name string) (int, bool) {
	msm.l.RLock()
	defer msm.l.RUnlock()
	val, ok := msm.scoreboard[name]
	return val, ok
}
```

뮤텍스 잠금을 획득하고자 할 때, 잠금이 해제되었는지 확인해야 한다. Lock과 RLock이 호출된 후에 즉시 Unlock 호출을 위해 defer문을 사용하였다.

### 주의사항

채널 또는 뮤텍스는 다음과 같은 경우에 사용하는 것이 좋다.

- 고루틴을 조정하거나 고루틴에 의해 변경되는 값을 추적할 경우에는 **채널**
- 구조체에 항목을 공유하여 접근하는 경우에는 **뮤텍스**
- 채널을 사용할 때 성능의 문제가 있고 이슈가 고쳐지지 않을 경우에는 **뮤텍스**

데이터가 HTTP 서버나 데이터베이스 같은 외부 서비스에 저장되어 있다면 뮤텍스를 사용하지 않는 것이 좋다. 또한 잠금과 잠금 해제의 쌍을 올바르게 유지하지 않으면 프로그램이 교착 상태에 빠질 수 있으니 주의해야 한다.

<br/><br/><br/>