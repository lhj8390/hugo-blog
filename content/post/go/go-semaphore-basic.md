+++
author = "lhj8390"
title = "semaphore 개념"
date = "2023-07-25"
description = "golang에서 semaphore의 개념 및 예제에 대해 설명한다."
tags = ["Go", "Golang", "semaphore"]
categories = ["language"]
subcategories = ["Go"]
+++

> Semaphore 란 여러 스레드 혹은 고루틴을 통해 공유 리소스에 대한 액세스를 제한하거나 제어하는 데에 사용할 수 있는 구성을 뜻한다.

<br/>

{{<figure src="/images/semaphore/1.png" class="large">}}

일반적으로 고루틴을 사용한다면 N 개의 요청을 병렬적으로 동시에 호출할 수 있을 것이다. 하지만 이러한 요청은 네트워크 I/O 성능을 낮출 수 있다.
이러한 상황을 해결하기 위해서는 동시에 실행하는 프로세스를 2개로 제한하고 프로세스를 N/2 번 반복할 수 있다. N 개의 요청을 동시에 처리하는 것보다 많은 시간이 걸리지만 네트워크가 한번에 2개의 요청만 처리하면 되기 때문에 I/O 네트워크 성능 부담을 줄일 수 있다.


<br/>

### In-N-Out

Semaphore 는 In-N-Out 개념을 활용한다. 
In-N-Out은 동시에 N 개의 자원에 대한 접근을 허용한다. 즉, N 개의 대기줄이 있고 각 줄은 FIFO 액세스를 허용하는 작은 대기열로 생각할 수 있다. Semaphore의 주요 관심사는 대기열의 작업이 아닌 N개의 접근 그 자체이다. 
한번에 N 개를 초과하는 요청이 들어왔을 경우에는 대기열에서 기다리도록 하여 동시에 실행할 수 있는 개수를 보장한다.

<br/>

## Mutex vs Semaphore

하나 이상의 스레드에서 자유롭게 액세스 할 수 있는 공유 리소스가 존재할 때, Mutex와 Semaphore 모두 공유 리소스에 대한 접근을 조절하는 동기화 기법이다.

- Semaphore : 공유된 자원의 데이터 혹은 임계영역(Critical Section) 등에 **여러** **Process 혹은 Thread**가 접근하는 것을 차단
- Mutex : 공유된 자원의 데이터 혹은 임계영역(Critical Section) 등에 **하나의** **Process 혹은 Thread**가 접근하는 것을 차단

즉, Semaphore 는 동기화 대상이 하나 이상이고 Mutex는 동기화 대상이 하나라고 볼 수 있다.

<br/>

## Semaphore package

go에서는 Semaphore 구현을 위한 패키지를 제공하고 있다.

https://pkg.go.dev/golang.org/x/sync/semaphore


<br/>

### Semaphore 사용

Semaphore package 를 import 한다.

```go
import “golang.org/x/sync/semaphore”
```

<br/>

Semaphore 을 생성하면서 최대 몇 개의 고루틴을 동시에 실행시킬 것인지 명시한다.

```go
// 동시에 실행할 고루틴 개수 지정
maxWorkers := runtime.GOMAXPROCS(0)
// 고루틴 개수를 사용하여 semaphore 객체 생성
sem := semaphore.NewWeighted(int64(maxWorkers))
```

<br/>

이제 Semaphore를 사용하여 최대 maxWorkers 개의 고루틴이 동시에 실행하도록 제한해보자.

```go
sem.Acquire(ctx, 1) // 한 개의 Semaphore 슬롯 획득
sem.Release(1) // 한 개의 Semaphore 슬롯 반환
```

Semaphore의 슬롯을 획득하는 `Acquire` 함수와 슬롯을 반환하는 `Release` 함수를 사용할 수 있다. 만약 슬롯이 가득 찼다면 `Acquire` 함수는 하나의 고루틴이 종료될 때까지 블록된다.

<br/>

마지막으로 남아있는 모든 고루틴이 종료될 때까지 블록되었다가 모든 슬롯을 획득한다.

```go
if err := sem.Acquire(ctx, int64(maxWorkers)); err != nil {
  log.Printf("Failed to acquire semaphore: %v", err)
}
```

위의 로직은 모든 작업이 무사히 종료되었는지 확인하는 용도로 `sync.waitgroup` 등을 사용한다면 대체할 수 있다.

<br/>

전체 코드는 아래 링크에서 확인할 수 있다.
[github 코드 보기](https://github.com/lhj8390/go-practice/blob/7176dad5085b19b9c56d539ce26f664c114aeaec/semaphore/main.go)

<br/><br/>



