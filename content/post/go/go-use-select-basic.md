+++
author = "lhj8390"
title = "Select 문 효과적으로 사용하기"
date = "2023-05-20"
description = "golang의 select 문을 사용하는 이유, 장점, select 문 패턴, select 문 사용 시 주의사항에 대해 설명한다."
tags = ["Go", "Golang", "select", "channel", "close"]
categories = ["language"]
subcategories = ["Go"]
+++

## Select 문의 용도

1.  **비동기 처리** : 여러 개의 채널을 동시에 감시하고, 도착한 값을 처리할 수 있다. 비동기적인 이벤트 처리를 구현할 수 있다.
2.  **블로킹 없는 통신** : 감시하는 채널 중 하나에 값이 도착할 때까지 블로킹되며, 그렇지 않은 경우 다른 채널을 계속해서 감시할 수 있다. 이를 통해 블로킹 없이 효율적인 채널 통신을 구현할 수 있다.
3.  **타임아웃 처리** : 타임아웃 채널과 함께 사용하여 일정 시간 동안 채널 값의 도착을 기다리다가 타임아웃이 발생하면 특정 동작을 수행할 수 있다.


## Select 문의 이점

select를 사용하여 여러 개의 채널을 관리하면 다음과 같은 이점이 있다고 한다.

1.  **Improved Performance**(성능 향상): By using multiple channels with the select statement, you can improve the performance of your Golang applications. This is because you can process data from multiple channels simultaneously, reducing the amount of time it takes to complete operations.
2.  **Reduced Resource Consumption**(리소스 감소): Multiple channels allow you to break down large tasks into smaller, more manageable ones. This reduces the amount of resources needed to perform each task and improves the overall efficiency of your application.
3.  **Enhanced Concurrency**(동시성 증가): With Golang select multiple channels, you can achieve higher levels of concurrency in your code. This is because multiple channels allow you to perform multiple tasks simultaneously, reducing the need for expensive synchronization primitives like locks.
4.  **Flexible Communication**(유연한 커뮤니케이션): Multiple channels with select statements offer more flexibility in communication between goroutines. You can use the select statement to handle different types of communication, such as reading from multiple channels or sending data to multiple channels.

출처 : https://hackthedeveloper.com/golang-select-multiple-channels/
<br/>

## Select 문 주요 패턴들

### 1. 추가 신호 채널로 채널 닫기

송신자가 N이고 수신자가 하나인 경우에는 값 전송을 중단해도 된다고 알릴 수 있는 추가적인 채널을 만들어 송신자 측에게 신호를 보낸다. (<font color="#7f7f7f">닫힌 채널에 데이터를 송신할 경우 패닉이 발생하기 때문에 기본적으로 송신자만 채널을 닫을 수 있다</font>.)

```go
stopChan := make(chan struct{}) // 추가적인 신호 채널 생성

wg := &sync.WaitGroup{}  
wg.Add(3)  

// sender1  
go func() {  
    defer wg.Done()  
    chan1 <- true  
    for {  
        select {  
        case <-stopChan:  
            fmt.Println("close chan1")  
            close(chan1)  
            return
        }  
    }  
}()

// sender2  
go func() {  
    defer wg.Done()  
    chan2 <- true  
    for {  
        select {  
        case <-stopChan:  
            fmt.Println("close chan2")  
            close(chan2)  
            return
        }  
    }  
}()

// receiver
go func() {  
    defer wg.Done()  
    var a, b, ok bool  
    for {  
        select {  
        case a, ok = <-chan1:  
            if !ok {  
                chan1 = nil  
                continue  
            }  
            fmt.Println(a)  
        case b, ok = <-chan2:  
            if !ok {  
                chan2 = nil  
                continue  
            }  
            fmt.Println(b)  
        default:  
            if a && b {  
                close(stopChan)  // stop 신호 
                return  
            }  
        }  
    }  
}()  
  
wg.Wait()

```

chan1 과 chan2 에서 값을 정상적으로 읽으면 `close(stopChan)` 이 실행되고, 각각 송신자의 select 문의 case 문이 통과되면서 채널을 닫을 수 있다.

**Output**

```
true
true
close chan1
close chan2
```


### 2. sync.Once 사용 패턴

여러 송신자가 하나의 채널을 공유할 경우 sync.Once 를 사용할 수 있다.

```go
chan1 := make(chan bool)  
once := &sync.Once{}

// sender1  
go func() {  
    chan1 <- true  
    time.Sleep(time.Second)  
    once.Do(func() {  
        close(chan1)  
    })  
}()

// sender2  
go func() {  
    chan1 <- true  
    time.Sleep(time.Second)  
    once.Do(func() {  
        close(chan1)  
    })  
}()

for {  
    select {  
    case a, ok := <-chan1:  
        if !ok {  
            chan1 = nil  
            break  
        }  
        fmt.Println(a)  
    }  
    if chan1 == nil {  
        break  
    }  
}
```

이미 닫은 채널을 또 닫으려고 시도할 경우 `close of closed channel panic` panic이 발생하지만 sync.Once 를 통해 한번만 수행되도록 보장할 수 있다.


### 3. 시간 초과 패턴

select 문에 time 패키지를 추가해 지정된 시간이 지날 경우 채널 읽기를 종료하도록 할 수 있다.

```go
func main() {  
  
    chan1 := make(chan bool)  
    chan2 := make(chan bool)  
	  
    go func() {  
        chan1 <- true  
        time.Sleep(time.Second * 4)
        close(chan1)  
    }()
	    
    select {  
    case a := <-chan1:  
        fmt.Println(a)
    case <-time.After(time.Second * 2):  
        fmt.Println("timeout!!!!")
    }  
}

```


**Output**

```
timeout!!!!
```

<br/>

## Select 문 사용 시 주의점

select 문은 여러 개의 채널을 다루는 데에 있어 최적화되어 있지만 그만큼 잘못 사용하면 메모리 누수가 발생하는 원인이 될 수 있다.


### 1. 닫힌 채널에 대한 예외처리

닫힌 채널에 대해 예외처리를 수행하지 않으면 지속적으로 select의 case 문 내부에 접근한다.

```go
func main() {  
  
    chan1 := make(chan bool)  
    chan2 := make(chan bool)  
	  
    go func() {  
        chan1 <- true  
        close(chan1)  
    }()  
	  
    go func() {  
        chan2 <- true  
        close(chan2)  
    }()  
	  
    for {  
        select {  
        case a := <-chan1:  
            fmt.Println(a)  
        case b := <-chan2:  
            fmt.Println(b)  
        }  
    }  
}
```

chan1 과 chan2 채널에 각각 데이터를 한 번만 전달하고 채널을 닫았음에도 불구하고 채널의 제로 값이 각각의 변수에 할당된다. 다른 case에서 데이터를 기다리더라도 닫힌 채널이 우선적으로 선택된다.


**Output**

```
true
true
false
false
.
.
. // memory leak!!!
```

<br/>

**해결방안**

```go
for {  
    select {  
    case _, ok := <-chan1:  
        if !ok {  
            chan1 = nil  
            break  
        } 
    case _, ok := <-chan2:  
        if !ok {  
            chan2 = nil  
            break  
        }
    }  
    if chan1 == nil && chan2 == nil {  
        break  
    }  
}
```

ok 문을 사용해서 채널이 닫혔는지 여부를 확인한 후 닫힌 채널이면 채널 변수를 nil로 설정한다. nil로 설정된 채널은 더 이상 값을 반환하지 못하기 때문에 case 문이 실행되지 않는다.



### 2. default 문 사용

```go
for {  
    select {  
    case _, ok := <-chan1:  
        if !ok {  
            chan1 = nil  
            // break or continue?  
        }  
    }
    if chan1 == nil { break }
}
```

닫힌 채널에 대해 채널 변수를 nil로 설정할 때, default 문의 여부에 따라 break와 continue 사용이 달라진다.

<br/>


```go
for {  
    select {  
    case _, ok := <-chan1:  
        if !ok {  
            chan1 = nil  
            // continue // deadlock 발생!
            break  
        }  
    }
    if chan1 == nil { break }
}

```

continue 문이 실행되면 select 문을 벗어나지 않고 다른 case문을 검사하게 되는데 default case문이 존재하지 않아 select 문이 무한히 실행되기 때문에 deadlock 이 발생한다. 따라서 continue 문 대신 break 문을 사용하여 select 문을 종료하거나 default 문을 추가해야 한다.

<br/>

```go
for {  
    select {  
    case a, ok := <-chan1:  
        if !ok {  
            chan1 = nil  
            continue  
        }  
        fmt.Println(a)  
    case b, ok := <-chan2:  
        if !ok {  
            chan2 = nil  
            continue  
        }  
        fmt.Println(b)  
    default:  
        if chan1 == nil && chan2 == nil {  
            return  
        }  
    }  
}
```

다음과 같은 예제일 경우 continue도 사용 가능하다. chan1 과 chan2 채널 변수가 nil일 경우 default case 문을 통과하게 되고 nil 체크 후 select 문을 종료할 수 있다.

<br/><br/><br/>