+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 중재자 (Mediator) 패턴"
date = "2023-06-14"
description = "GoF 디자인 패턴 중 중재자 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Mediator", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

> <span class="red">행동(behavioral) 패턴 중 하나</span>로, 객체 간의 복잡한 상호작용을 캡슐화하여 객체들 간의 결합도를 낮추는 디자인 패턴이다.

중재자 패턴은 객체 간의 직접적인 의존성을 없애고, 대신 중재자를 통해 상호작용을 수행한다.

{{< figure src="/images/gof-design-pattern-mediator/1.png" alt="image" >}}

### 중재자 패턴의 구성요소

-   **Mediator** : 객체 간의 상호작용을 조정하는 역할을 정의하는 인터페이스
-   **ConcreteMediator :** Mediator 인터페이스의 구현체**.** Mediator 역할을 실제로 수행
-   **Colleague** : 중재자 (Mediator)와 상호작용하는 객체로 인터페이스로 정의.
-   **Concrete Colleague** : Colleague 인터페이스의 구현체. Mediator 객체와 통신할 수 있는 메서드를 구현

<br/>

### 구현 예제

Mediator 인터페이스를 선언한다.

```java
public interface Mediator {
    void notify(User user, String message);
}
```

선언된 notify 메서드는 Mediator 객체에서 동료 객체들에게 메시지를 전달하기 위해 사용된다.

<br/>

Mediator 인터페이스를 구현한 ConcreteMediator 클래스를 만든다.

```java
public class ChatMediator implements Mediator {

    private final List<User> userList = new ArrayList<>();

    public void addClient(User user) {
        userList.add(user);
    }

    @Override
    public void notify(User user, String message) {
        if (userList.contains(user)) {
            user.receive(message);
        } else {
            System.out.println("주의 : 해당 유저는 없습니다.");
        }
    }
}
```

Mediator 인터페이스를 구현하여 객체 간의 상호작용을 조정하는 역할을 수행한다. 여기서 notify 메서드는 userList에 사용자 객체가 등록되어 있는 경우 해당 사용자 객체에게 메시지를 전달한다.<br/>
→ _“객체 간의 결합도를 낮추고, 객체 간의 상호작용을 중앙 집중적으로 관리할 수 있다.”_

<br/>

Colleague 인터페이스를 만든다.

```java
public interface User {
    void send(User user, String message);
    void receive(String message);
}
```

Colleague 객체들이 구현해야 하는 메서드를 정의한다. send 메서드는 Colleague 객체가 다른 Colleague 객체에게 메시지를 보낼 때 사용되고, receive 메서드는 Colleague 객체가 다른 Colleague 객체로부터 메시지를 받았을 때 호출되는 메서드이다.

<br/>

Colleague 인터페이스를 구현한 Concrete Colleague 클래스를 만든다.

```java
public class Client implements User {

    private final String name;
    private final Mediator chatMediator;

    public Client(String name, Mediator chatMediator) {
        this.name = name;
        this.chatMediator = chatMediator;
    }

    @Override
    public void send(User user, String message) {
        System.out.println(name + " 유저가 메시지를 보냅니다....");
        chatMediator.notify(user, message);
    }

    @Override
    public void receive(String message) {
        System.out.println(name + " 유저가 메시지를 받았습니다 : " + message);
    }
}
```

생성자에서는 Mediator 객체를 인자로 받아 다른 Colleague 객체들과 소통할 수 있다. 또한 send 메서드와 receive 메서드를 구현하여 객체 간의 메시지 송수신 기능을 구현한다.<br/>
→ _“Colleague 객체가 Mediator 객체와 직접 통신하지 않고, 중간에 Mediator 객체를 거쳐서 다른 객체와 상호작용하기 때문에 객체 간의 결합도를 낮출 수 있다.”_

<br/>

중재자 패턴을 사용하는 클라이언트 코드이다.

```java

ChatMediator mediator = new ChatMediator();

User user1 = new Client("user1", mediator);
User user2 = new Client("user2", mediator);
User user3 = new Client("user3", mediator);
User user4 = new Client("user4", mediator);

mediator.addClient(user1);
mediator.addClient(user2);
mediator.addClient(user3);

user1.send(user3, "안녕하세요. user3");
user3.send(user1, "안녕하세요.");
user2.send(user4, "안녕하세요. user4");

```

채팅방의 역할을 수행하는 ChatMediator 객체와 4개의 Client 객체를 생성한다. Client 객체가 send 메서드를 호출할 경우 ChatMediator 객체를 통해 다른 Client 객체에게 메세지를 전달할 수 있게 된다.

<br/>

### 중재자 패턴 고려사항

> 객체 간의 상호작용이 복잡하고 각각에게 강한 의존성이 있어 결합도를 낮춰야 하는 경우에 중재자 패턴을 사용한다. 하지만 객체 간의 강한 결합도가 필요하거나 객체 간의 상호작용이 한 두 개 뿐이며 변경이 적고 중재자가 병목 현상을 유발할 가능성이 있는 경우에는 적합하지 않다.

<br/>

**중재자 패턴 사용 시 장점**

1.  객체 간의 결합도를 줄일 수 있어 유지보수성이 높아진다.
2.  객체 간의 의존성을 최소화하여, 객체의 재사용성을 높일 수 있다.
3.  객체 간의 상호작용을 중앙에서 관리하므로, 복잡한 상호작용을 간소화 할 수 있다.

<br/>

**중재자 패턴 사용 시 단점**

1.  중재자가 많은 역할을 담당하게 되어, 중재자 클래스의 복잡도가 증가할 수 있다.
2.  중재자에게 많은 역할이 부여될 경우, 단일 책임 원칙(SRP)을 위배하게 될 수 있다.
3.  중재자 클래스가 추가될 경우, 전체적으로 변경사항이 발생할 가능성이 있다.

<br/><br/><br/>