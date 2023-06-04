+++
author = "lhj8390"
title = "Golang - context 개념 및 기본 예제"
date = "2023-06-03"
description = "golang에서 context 패키지의 개념과 context를 사용하는 방법에 대해 설명한다."
tags = ["Go", "Golang", "context"]
categories = ["language"]
subcategories = ["Go"]
+++


> HTTP 서버는 마이크로 서비스의 요청 체인을 식별하기 위해 추적 ID를 원하거나 너무 오래 걸린다면 요청을 중단하는 타이머를 설정할 수 있다. 하지만 Go에서는 고루틴이 고유한 식별자를 가지고 있지 않기 때문에 요청 메타 데이터를 처리하기 힘들다. 이에 Go는 컨텍스트라는 구성을 통해 요청 메타 데이터 문제를 해결했다.

컨텍스트는 요청 메타 데이터와 함께 고루틴 간의 작업 취소, 타임아웃, 값 전달 등과 같은 고루틴의 문맥관리를 제공하는 인터페이스이다.

<br/>

## 기본 예제

컨텍스트는 단순히 context 패키지에 정의된 Context 인터페이스를 만족하는 인스턴스이다.

```go
func logic(ctx context.Context, info string) (string, error) {
  // do something
  return "", nil
}
```

함수에서 마지막에 반환하는 것은 오류라는 관례가 있는 것처럼, 프로그램을 통해 명시적으로 전달되는 함수의 첫 번째 파라미터로써 컨텍스트를 사용한다.

<br/>

context 패키지에는 컨텍스트를 생성하고 래핑하기 위한 여러 팩토리 함수가 포함된다.
기존 컨텍스트가 없는 경우에는 `context.Background` 함수를 사용하여 빈 초기 컨텍스트를 만든다.

```go
ctx := context.Background()
result, err := logic(ctx, "a string")
```

빈 컨텍스트는 기존 컨텍스트를 래핑하는 시작점이 되고, 메타 데이터를 추가하기 위해서는 팩토리 함수를 사용해 기존 컨텍스트를 래핑하여 새로운 컨텍스트를 생성해야 한다.

<br/>

HTTP 서버를 작성할 때, 미들웨어 계층에서 최상위 http.Handler로 컨텍스트를 전달하거나 획득하는 예시이다.

```go
func Middleware(handler http.Handler) http.Handler {
  return http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
    ctx := req.Context()  // 1
    // 컨텍스트로 작업을 감싼다.
    req = req.WithContext(ctx) // 2
    handler.ServeHTTP(rw, req)
  })
}
```

1. Context는 요청과 연관된 context.Context를 반환한다.
2. WithContext는 생성된 context.Context와 결합된 request 와 함께 새로운 http.Request를 반환한다.

<br/>

HTTP 서버의 핸들러 함수이다.

```go
func handler(rw http.ResponseWriter, req *http.Request) {
    ctx := req.Context()
    err := req.ParseForm()
    if err != nil {
        rw.WriteHeader(http.StatusInternalServerError)
        rw.Write([]byte(err.Error()))
        return
    }
    data := req.FormValue("data")
    result, err := logic(ctx, data) // 1
    if err != nil {
        rw.WriteHeader(http.StatusInternalServerError)
        rw.Write([]byte(err.Error()))
        return
    }
    rw.Write([]byte(result))
}
```

요청에서 Context 메서드를 사용하여 컨텍스트를 추출하고 1) 첫 번째 파라미터로 컨텍스트와 함께 비즈니스 로직(Logic 메서드)를 호출한다.

<br/>

WithContet 메서드를 다음과 같은 상황에서도 사용할 수 있다.

```go
type ServiceCaller struct {
    client *http.Client
}

func (sc ServiceCaller) callAnotherService(ctx context.Context, data string) (string, error) {
    req, err := http.NewRequest(http.MethodGet, "<http://example.com?data=>"+data, nil)

    if err != nil {
        return "", err
    }
    req = req.WithContext(ctx)
    resp, err := sc.client.Do(req)
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    if resp.StatusCode != http.StatusOK {
        return "", fmt.Errorf("unexpected status code %d", resp.StatusCode)
    }

    // 응답 처리
    id, err := processResponse(resp.Body)
    return id, err
}
```

다른 HTTP 서비스로 HTTP 호출을 만드는 예시이다. client.Do(req) 를 사용하여 요청을 전송하고, 서버로부터의 응답을 수신한다. 미들웨어를 통해 컨텍스트를 전달하는 경우와 같이 WithContext를 사용하여 외부로 나가는 요청에 컨텍스트를 설정할 수 있다.

<br/>

## Cancel

> 여러 개의 고루틴이 동시에 실행되고 있을 때, 하나의 서비스가 유효한 결과를 반환하지 않는다면 다른 고루틴을 처리할 필요가 없다. 이런 상황에서 컨텍스트를 사용하여 작업을 취소할 수 있다.

취소 가능한 컨텍스트를 생성하려면 WithCancel 함수를 사용한다.

```go
ctx, cancel := context.WithCancel(context.Background())
```

파라미터로 context.Context를 받고 context.Context와 context.CancelFunc를 반환한다.
- context.Context : 함수로 전달된 컨텍스트와 동일하지 않는 컨텍스트를 반환받는다.
- context.CancelFunc : 컨텍스트를 취소하여 잠재적으로 취소를 대기하는 모든 코드에 중지할 시점을 알려준다.
<br/>

```go
func slowServer() *httptest.Server {
    s := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        time.Sleep(2 * time.Second)
        w.Write([]byte("Slow response"))
    }))
    return s
}

func fastServer() *httptest.Server {
    s := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if r.URL.Query().Get("error") == "true" {
            w.Write([]byte("error"))
            return
        }
        w.Write([]byte("ok"))
    }))
    return s
}
```

```go
var client = http.Client{}

func callBoth(ctx context.Context, errVal string, slowURL string, fastURL string) {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()
    var wg sync.WaitGroup
    wg.Add(2)
    go func() {
        defer wg.Done()
        err := callServer(ctx, "slow", slowURL)
        if err != nil {
            cancel()
        }
    }()
    go func() {
        defer wg.Done()
        err := callServer(ctx, "fast", fastURL+"?error="+errVal)
        if err != nil {
            cancel()
        }
    }()
    wg.Wait()
    fmt.Println("done with both")
}

func callServer(ctx context.Context, label string, url string) error {
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
    if err != nil {
        fmt.Println(label, "request err:", err)
        return err
    }
    resp, err := client.Do(req)
    if err != nil {
        fmt.Println(label, "response err:", err)
        return err
    }
    data, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println(label, "read err:", err)
        return err
    }
    result := string(data)
    if result != "" {
        fmt.Println(label, "result:", result)
    }
    if result == "error" {
        fmt.Println("cancelling from", label)
        return errors.New("error happened")
    }
    return nil
}
```


<br/><br/><br/><br/>