+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 단일체 (Singleton) 패턴"
date = "2023-02-28"
description = "GoF 디자인 패턴 중 단일체, Singleton 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Singleton", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

## 단일체 (Singleton 패턴

---

> <span class="red">생성(Creational) 패턴 중 하나</span>로, 오직 한 개의 클래스 인스턴스만을 갖도록 보장하고 이에 대한 전역적인 접근점을 제공하는 패턴

또 다른 인스턴스가 생성되지 않도록 할 수 있고, 클래스 자신이 그 인스턴스에 대한 접근 방법을 제공할 수 있다.

### 복합체 패턴 구성요소

- **Singleton** : `Instance()` 연산을 정의하여, 유일한 인스턴스로 접근할 수 있도록 한다.

### 구현 예제

> 단일체(Singleton) 패턴을 구현하는 방식은 여러가지가 있다. Eager loading, Lazy loading, Thread-safe, Double-checked locking 싱글톤 등이 있다.

<span class="bg_b wide">**우선 Eager loading 방식을 구현한 예제이다.**</span>

클래스가 로드될 때 즉시 인스턴스를 생성하는 방식이다.

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

인스턴스를 처음 요청할 때까지 대기하지 않아야 하므로 더 빠른 응답 시간을 제공할 수 있다.

<br/>

<span class="bg_b wide">**Lazy loading 방식을 구현한 예제이다.**</span>

인스턴스가 필요한 경우에만 인스턴스를 생성하는 방식이다.

```java
public class LazySingleton {
    private static LazySingleton INSTANCE;

    public static LazySingleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new LazySingleton();
        }
        return INSTANCE;
    }

}
```

처음으로 인스턴스가 요청될 때까지 인스턴스를 생성하지 않아 시스템 자원을 절약할 수 있다.

<br/>

<span class="bg_b wide">**Thread-safe 방식을 구현한 예제이다.**</span>

멀티 스레딩 환경에서 안전하게 사용할 수 있는 방식이다.

```java
public class ThreadSafeSingleton {
    private static ThreadSafeSingleton INSTANCE;

    public static synchronized ThreadSafeSingleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new ThreadSafeSingleton();
        }
        return INSTANCE;
    }
}
```

인스턴스를 생성하는 메서드에 대한 락(lock)을 설정하여 한 번에 하나의 스레드만 인스턴스를 생성하도록 보장한다.

### 단일체 패턴 고려사항

> 자원 공유가 필요하거나 전역 상태를 유지해야 하는 경우 주로 사용되지만, 상태를 번경하는 작업이 많은 경우 싱글톤이 다른 클래스에 의존하고 있어 테스트 시 가짜 객체로 대체하는 것이 어려울 경우 단일체 패턴을 사용하면 안된다.

<br/>

**단일체 패턴 사용 시 장점**

1. 하나의 인스턴스를 공유하기 때문에 메모리 사용량이 줄어들고, 성능을 향상시킬 수 있다.
2. 전역 상태를 쉽게 유지할 수 있다.

<br/>

**단일체 패턴 사용 시 단점**

1. 인스턴스를 생성하는 방법이 제한되기 때문에 테스트에서 문제가 발생할 수 있다.
2. 다른 클래스에 의존하고 있을 수 있으므로, 의존성 문제가 발생할 수 있다.
3. 인스턴스가 하나만 존재하므로, 상태 변경 문제가 발생할 수 있다.

<br/>