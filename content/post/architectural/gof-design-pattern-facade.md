+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 퍼사드 (Facade) 패턴"
date = "2023-03-11"
description = "GoF 디자인 패턴 중 추상 팩토리 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Facade", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

## 퍼사드 (Facade) 패턴

---

> <span class="red">구조(structural) 패턴 중 하나</span>로, 한 서브시스템 내의 인터페이스 집합에 대한 확일화된 하나의 인터페이스를 제공하는 패턴이다.

서브시스템의 복잡도를 숨기고, 클라이언트가 서브시스템과의 상호작용을 단순화하기 때문에 소프트웨어의 유지보수성을 높일 수 있다.

### 퍼사드 패턴 구성요소

- Facade : 클라이언트가 서브시스템을 사용하기 쉽게 하기 위한 인터페이스를 제공
- Subsystem : 시스템의 기능을 구현하는 클래스나 모듈
- Client : Facade를 사용하여 서브시스템을 조작하는 클라이언트

### 구현 예제

시스템의 기능을 구현하는 Subsystem 클래스를 만들어준다.

```java
public class AccountService {

    public void login(String email) {
        System.out.println(email + "로 로그인합니다.");
    }
}

public class PaymentService {

    public void payment() {
        System.out.println("결제를 진행합니다.");
    }
}

public class ShippingService {

    public void startShipping() {
        System.out.println("배송이 시작되었습니다.");
    }
}
```

3개의 Service 클래스를 만들어주었다. 각각은 Account, Payment, Shipping 기능을 수행한다.

<br/>

서브 시스템을 더 쉽게 사용할 수 있는 단순한 인터페이스를 제공하는 Facade 클래스를 만든다.

```java
public class ShopFacade {

    private final AccountService accountService;
    private final PaymentService paymentService;
    private final ShippingService shippingService;

    public ShopFacade() {
        accountService = new AccountService();
        paymentService = new PaymentService();
        shippingService = new ShippingService();
    }

    public void buyProduct(String email) {
        accountService.login(email);
        paymentService.payment();
        shippingService.startShipping();
    }
}
```

클라이언트는 각각의 서비스를 일일이 호출할 필요 없이 ShopFacade의 buyProduct 메소드만 호출하면 된다.<br/>
→ *“서브 시스템의 복잡성을 감추고, 서비스의 구현이 변경되어도 클라이언트에게 영향을 주지 않는다”*

<br/>

Facade 패턴을 사용하는 Client 코드이다.

```java
ShopFacade shopFacade = new ShopFacade();
shopFacade.buyProduct("tmp@gmail.com");
```

Client에서는 ShopFacade 객체를 생성한 후, buyProduct() 메서드만 호출하여 서브시스템의 내부 구현과 상관없이 내부 로직을 간단하게 처리한다.<br/>
→ *“코드의 복잡성을 줄이고 유지 보수성을 향상시킨다.”*

### 퍼사드 패턴 고려사항

> 시스템의 일부 기능을 숨겨야 하거나, 사용자가 직접 접근해서는 안되는 경우에 퍼사드 패턴을 주로 사용한다. 하지만 일부 기능에 대해 <span class="red">사용자가 직접 접근할 필요가 있거나 새로운 기능이 추가될 때마다 퍼사드 클래스 자체를 변경해야 하는 경우</span>에는 사용을 고려해 보아야 한다.
> 

<br/>

**퍼사드 패턴 사용 시 장점**

1. 서브시스템 내부를 캡슐화하고 단순한 인터페이스만 노출할 수 있다.
2. 서브시스템과 클라이언트 사이의 의존성을 줄여준다.

<br/>

**퍼사드 패턴 사용 시 단점**

1. 퍼사드 객체가 많은 역할을 가져가게 된다면 클래스가 복잡해지고 유지보수가 어려워진다.
2. 과도하게 사용 시, 서브시스템 내부 구성 요소에 대한 접근성이 감소하게 된다. → <span class="gray">*서브시스템 제어 및 변경이 어려움*</span>

<br/>