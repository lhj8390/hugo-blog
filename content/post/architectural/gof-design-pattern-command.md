+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 명령 (Command) 패턴"
date = "2023-04-03"
description = "GoF 디자인 패턴 중 명령 (Command) 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Command", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++
## 명령 (Command) 패턴

---

> <span class="red">행동(behavioral) 패턴 중 하나</span>로, 어떤 작업을 수행하는 객체를 캡슐화하는 패턴이다.
> 

작업을 요청하는 객체와 작업을 수행하는 객체가 서로 분리된 형태로, 요청을 대기시키거나 되돌릴 수 있는 연산을 제공한다.

### 명령 패턴 구성요소

- Command : 작업을 수행하기 위한 인터페이스 정의
- ConcreteCommand : Command 인터페이스를 구현하는 클래스. 실제 작업을 수행한다.
- Invoker : Command 객체 생성 및 작업 요청
- Receiver : 작업을 실제로 수행하는 객체

### 구현 예제

작업을 수행하기 위한 Command 인터페이스를 정의한다.

```java
public interface Command {
    void execute();
}
```

작업을 수행할 execute 메서드를 정의하였다.

<br/>

Command 인터페이스를 구현하는 ConcreteCommand 클래스를 만든다.

```java
public class BuyCommand implements Command {
    private final Order order;

    public BuyCommand(Order order) {
        this.order = order;
    }

    @Override
    public void execute() {
        order.buy();
    }
}

public class CancelCommand implements Command {
    private final Order order;

    public CancelCommand(Order order) {
        this.order = order;
    }

    @Override
    public void execute() {
        order.cancel();
    }
}
```

Command 인터페이스를 구현하는 BuyCommand와 CancelCommand 클래스를 만들었다. 
각각의 클래스는 Order(<span class="gray">*Receiver*</span>) 객체를 생성자를 통해 주입받으며 실제 작업을 수행하는 Order(<span class="gray">*Receiver*</span>) 객체를 사용하여 Command 인터페이스의 execute() 메서드를 구현한다.<br/>
→ *“작업을 요청하는 객체(<span class="gray">Invoker</span>)와 작업을 수행하는 객체(<span class="gray">Receiver</span>)를 분리하여 결합도를 낮춘다.”*

<br/>

작업을 수행하는 객체인 Receiver 클래스를 만든다.

```java
public class Order {

    private final ProductType productType;
    private final int price;

    public Order(ProductType productType, int price) {
        this.productType = productType;
        this.price = price;
    }

    public void buy() {
        System.out.printf("%s 을 %d 가격으로 구매합니다.%n", productType, price);
    }

    public void cancel() {
        System.out.printf("%s 구매를 취소합니다.%n", productType);
    }
}
```

비즈니스 로직을 담고 있는 객체로, 모든 Command 클래스는 Receiver 객체의 메서드에게 실행을 위임한다. 여기서 Order 클래스는 buy(), cancel() 작업에 대한 구체적인 구현을 담당한다.<br/>
→ *“객체 수정에 대한 유연성을 제공하며 확장 가능한 코드를 작성할 수 있다.”*

<br/>

마지막으로 작업을 요청하는 Invoker 클래스를 구현한다.

```java
public class OrderCommand {

    private final List<Command> commandList = new ArrayList<>();

    public void setCommand(Command command) {
        commandList.add(command);
    }

    public void executeAll() {
        for (Command c: commandList) {
            c.execute();
        }
    }
}
```

`setCommand` 메서드를 통해 Command 객체를 추가하고 `executeAll` 메서드를 통해 모든 Command 객체를 실행하는 역할을 수행한다. <br/>
→ *“작업의 구체적인 내용을 알 필요가 없기 때문에 유연하고 확장 가능한 코드를 작성할 수 있다.”*

<br/>

명령 패턴을 사용하는 Client 코드이다.

```java
Order computerOrder = new Order(ProductType.COMPUTER, 1000000);
Order phoneOrder = new Order(ProductType.PHONE, 500000);

OrderCommand order1 = new OrderCommand();
OrderCommand order2 = new OrderCommand();

order1.setCommand(new BuyCommand(computerOrder));
order1.setCommand(new CancelCommand(computerOrder));
order1.executeAll();

order2.setCommand(new BuyCommand(phoneOrder));
order2.setCommand(new CancelCommand(phoneOrder));
order2.executeAll();
```

Command 객체는 작업에 대한 요청을 캡슐화한다. 각 요청마다 별도의 Command 객체를 생성하고 매개 변수화할 수 있다. 즉, <span class="red">명령을 쉽게 추가하거나 제거할 수 있으며 다른 요청으로 바꿀 수 있다.</span>

### 명령 패턴 고려사항

> 명령 패턴은 로깅과 오류 처리를 구현할 때, 실행 대기열을 구현해야 할 경우에 유용하게 사용할 수 있다. 하지만 간단한 작업을 처리하는 경우나 실행할 명령이 고정적인 경우에는 사용하지 않는 것이 좋다.
> 

<br/>

**명령 패턴 사용 시 장점**

1. 유연성과 확장성이 높아진다.
2. 명령을 캡슐화하므로 디버깅이 쉬워진다.
3. 실행 취소/다시 실행 같은 기능을 구현할수 있다.
4. 간단한 명령들을 조합하여 복잡한 명령을 만들 수 있다.

<br/>

**명령 패턴 사용 시 단점**

1. 많은 Command 객체를 만들어야 하기 때문에 복잡성이 증가한다.
2. 요청을 캡슐화하기 때문에 메모리 사용량이 증가할 수 있다.


<br/>
<br/>
<br/>