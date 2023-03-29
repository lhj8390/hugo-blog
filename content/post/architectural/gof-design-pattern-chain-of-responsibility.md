+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 책임 연쇄 (Chain of Responsibility) 패턴"
date = "2023-03-29"
description = "GoF 디자인 패턴 중 책임 연쇄 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Chain of Responsibility", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++
## 책임 연쇄 (Chain of Responsibility) 패턴

---

> <span class="red">행동(behavioral) 패턴 중 하나</span>로, 여러 개의 객체 중에서 요청을 처리할 수 있는 객체를 찾아서 처리하도록 하는 패턴이다.
> 

요청을 처리하는 객체(핸들러)들이 연결된 체인을 이루고 있으며, 요청을 처리할 수 있는 핸들러를 찾을 때까지 체인 상의 모든 핸들러가 요청을 받아서 처리할 수 있는지를 순차적으로 검사한다.

### 책임 연쇄 패턴 구성요소

- Handler: 요청을 처리하는 메서드를 가진 인터페이스나 추상 클래스.
- ConcreteHandler: 실제로 요청을 처리하는 구현 클래스.
- Client: 요청을 보내는 객체

### 구현 예제

요청을 처리하는 메서드를 선언하는 Handler 인터페이스를 선언한다.

```java
public abstract class Manager {
    protected String name;
    protected Manager superior;

    public Manager(String name) {
        this.name = name;
    }

    public void setSuperior(Manager superior) {
        this.superior = superior;
    }

    public abstract void requestApplications(Request request);
}
```

각각의 Manager 객체는 superior 변수를 통해 자신의 상사 매니저 객체를 참조하도록 되어 있다. Manager 객체는 스스로 해당 요청을 처리할 수 있는지를 판단하고, 처리할 수 없다면 상사 매니저(<span class="gray">superior</span>) 객체에게 해당 요청을 전달한다. <br/>
→ *“요청을 처리하는 객체들 간의 연결고리 역할을 수행한다.”*

<br/>

실제로 요청을 처리하는 구현 클래스인 ConcreteHandler 클래스를 만든다.

```java
public class GeneralManager extends Manager {

    public GeneralManager(String name) {
        super(name);
    }

    @Override
    public void requestApplications(Request request) {
        if (
            request.getRequestType() == RequestType.PURCHASE &&
            request.getAmount() < 10
        ) {
            System.out.println("GeneralManager 가 처리중입니다 : ");
        } else {
            superior.requestApplications(request);
        }
    }
}

public class MiddleManager extends Manager {

    public MiddleManager(String name) {
        super(name);
    }

    @Override
    public void requestApplications(Request request) {
        if (request.getRequestType() == RequestType.PURCHASE) {
            System.out.println("MiddleManager 가 처리중입니다 : ");
        } else {
            superior.requestApplications(request);
        }
    }
}

public class SuperManager extends Manager {

    public SuperManager(String name) {
        super(name);
    }

    @Override
    public void requestApplications(Request request) {
        System.out.println("SuperManager 가 처리중입니다 : ");
    }
}
```

각각의 매니저 객체가 requestApplications 메서드를 사용하여 자신이 처리할 수 있는 요청을 처리하고 처리할 수 없을 경우 `superior.requestApplications(request);` 와 같이 상사 매니저 객체에게 요청을 전달한다.<br/>
→ *“객체 간의 결합도를 낮출 수 있고, 코드의 유연성과 확장성을 높일 수 있다.”*

<br/>

책임 연쇄 패턴을 사용하는 Client 코드이다.

```java
Manager manager = new GeneralManager("일반관리자");
Manager middleManager = new MiddleManager("중간관리자");
Manager superManager = new SuperManager("슈퍼관리자");

// 다음으로 request 를 넘길 manager 설정
manager.setSuperior(middleManager);
middleManager.setSuperior(superManager);

Request purchase = new Request(RequestType.PURCHASE, 1000);
Request cancel = new Request(RequestType.CANCEL, 10);
manager.requestApplications(purchase);
manager.requestApplications(cancel);
```

setSuperior() 메서드를 사용하여 다음 Manager에게 요청을 전달할 수 있도록 연결고리를 설정한다. GeneralManager 객체의 requestApplications 메소드가 호출되면서 요청을 처리하지 못하면 자신의 상사 매니저에게 요청을 전달한다.<br/>
→ *“객체 간의 연결고리 설정을 통해 코드의 유연성과 확장성이 높아지며, 객체 간의 결합도를 낮출 수 있다.”*

### 책임 연쇄 패턴 고려사항

> 객체 간의 결합도를 낮춰야 하거나 요청을 처리할 객체를 동적으로 결정할 때 책임 연쇄 패턴을 사용하면 좋다. 다만 <span class="red">모든 요청을 처리해야 하는 경우나 책임 연쇄가 너무 길어진다면</span> 책임 연쇄 패턴을 사용해서는 안된다.
> 

<br/>

**책임 연쇄 패턴 사용 시 장점**

1. 객체 간의 결합도를 낮출 수 있다.
2. 객체 간의 연결고리를 동적으로 설정하기 때문에 요청 처리를 유연하게 할 수 있다.

<br/>

**책임 연쇄 패턴 사용 시 단점**

1. 처리할 수 있는 객체가 없는 경우 요청이 처리되지 않을 수 있다. -<span class="gray"> *요청 처리 보장 X*</span>
2. 요청을 처리할 객체를 찾기 위해 여러 객체를 검색하는 과정에서 오버헤드가 발생할 수 있다.


<br/><br/>