+++
author = "lhj8390"
title = "Golang - 채널 버퍼링"
date = "2023-05-03"
description = "golang에서 unbuffered channel, buffer channel 개념과 차이점을 설명한다."
tags = ["Go", "Golang", "channel"]
categories = ["language"]
subcategories = ["Go"]
+++

>go 에서 채널은 unbuffered channel, buffer channel 두 종류가 있다. 기본적으로 채널은 Unbuffered 이다. 

## Unbuffered channel

Unbufferd 채널은 송신 고루틴과 수신 고루틴이 동시에 실행되지 않고, 송신 고루틴이 데이터를 보낸 후에는 수신 고루틴이 데이터를 가져갈 때까지 대기한다. 수신자가 데이터를 받을 때까지 송신자는 block이 걸리기 때문에 동기화가 보장된다.

### example

```go
func main() {  
    wg := sync.WaitGroup{}  
    chanFoo := make(chan string)  
    wg.Add(2)  
	
    go func() {  
        defer wg.Done()  
        chanFoo <- "Hello"  
        time.Sleep(1 * time.Second)
        for i := 0; i < 3; i++ {  
            fmt.Println("wait" + strconv.Itoa(i))  
        }  
    }()  
	  
    go func() {  
        defer wg.Done()  
        time.Sleep(2 * time.Second)
        fmt.Println(<-chanFoo)  
    }()  
	
    wg.Wait()  
}
```

송신 고루틴일 경우 1초간 대기하고, 수신 고루틴은 2초간 대기하는 코드를 작성해보았다. 만약 block이 수행되지 않았다면 `wait` 관련 반복문이 콘솔에 먼저 찍힐 것이다. 하지만 상단의 코드는 다음과 같은 결과를 보여준다.

```
Hello
wait0
wait1
wait2
```

채널에 버퍼가 존재하지 않아 데이터를 수신하기 전까지 송신 고루틴 부분에 block이 걸리기 때문에 Hello (<font color="#7f7f7f">수신 고루틴이 데이터를 읽는 부분</font>) 가 먼저 콘솔에 찍힌 것을 확인할 수 있다.

<br/>

### deadlock이 발생하는 경우

```go
func main() {  
    chanFoo := make(chan string)  
    chanFoo <- "Hello"  
    for i := 0; i < 3; i++ {  
        fmt.Println("wait" + strconv.Itoa(i))  
    }  
    fmt.Println(<-chanFoo)  
}
```

main 고루틴에서 버퍼가 없는 채널에 대해 쓰기 및 읽기를 수행하는 코드이다. 데이터를 보낸 후 채널은 데이터를 수신할 때까지 무한정 대기 상태로 들어가게 되고 `chanFoo <- "Hello"` 코드 부분부터 block 된다. 이후 로직을 수행할 수 없기 때문에 하단처럼 deadlock이 걸리게 된다.

```
fatal error: all goroutines are asleep - deadlock!
```

<br/>

## buffered channel

Buffered 채널은 수신 고루틴이 받을 준비가 되어 있지 않아도 지정된 버퍼만큼 데이터를 보내고 계속 다른 일을 수행할 수 있기 때문에 비동기적으로 동작한다는 특징이 있다.

### example

```go
func main() {  
    wg := sync.WaitGroup{}  
    chanFoo := make(chan string, 1)  
    wg.Add(2)  
	  
    go func() {  
        defer wg.Done()  
        chanFoo <- "Hello"  
        time.Sleep(1 * time.Second)  
        for i := 0; i < 3; i++ {  
            fmt.Println("wait" + strconv.Itoa(i))  
        }  
    }()  
	  
    go func() {  
        defer wg.Done()  
        time.Sleep(2 * time.Second)  
        fmt.Println(<-chanFoo)  
    }()  
	  
    wg.Wait()  
}
```

버퍼 size가 하나이고 버퍼 개수에 맞게 채널에 데이터를 하나만 보내는 코드를 작성해보았다. 만약 block이 수행된다면 `fmt.Println(<-chanFoo)` 부분이 먼저 수행될 것이다. 

```
wait0
wait1
wait2
Hello
```

데이터를 버퍼에 저장하고 다음 작업을 수행할 수 있기 때문에 `chanFoo <- "Hello"` 이후의 코드인 `wait` 반복문이 먼저 노출된다.

<br/>

### deadlock이 발생하는 경우

```go
func main() {  
    chanFoo := make(chan string, 1)  
	  
    for i := 0; i < 3; i++ {  
        chanFoo <- "Hello"  
        fmt.Println("wait" + strconv.Itoa(i))  
    }  
	  
    for i := 0; i < 3; i++ {  
        fmt.Println(<-chanFoo)  
    }  
}
```

버퍼가 다 차면 Unbufferd 채널과 마찬가지로 버퍼에 빈 자리가 생길 때까지 대기한다. 반복문을 이용하여 버퍼 크기보다 많은 데이터를 보내려고 시도했기 때문에 block 이 수행 되어 deadlock이 발생한다.

```
fatal error: all goroutines are asleep - deadlock!
```

<br/>
<br/>