+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 방문자 (visitor) 패턴"
date = "2023-05-14"
description = "GoF 디자인 패턴 중 방문자 (visitor) 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "visitor", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

> <span class="red">행동(behavioral) 패턴</span> 중 하나로, 객체 구조는 그대로 두고, 객체에 대한 연산(기능)을 별도의 객체로 분리해서 캡슐화하는 패턴이다.

방문자 패턴을 사용하면 기능을 추가하면서 객체를 수정하지 않고도 기능을 확장할 수 있다.

{{< figure src="/images/gof-design-pattern-visitor/1.png" alt="image" >}}

### 방문자 패턴 구성요소

-   Visitor : ConcreteElement 객체를 방문할 때 호출되어야 하는 visit 메소드 정의
-   ConcreteVisitor : Visitor 인터페이스를 구현한 클래스. ConcreteElement 객체가 실제로 수행할 작업 정의
-   Element : 방문자가 방문할 수 있는 객체 정의
-   ConcreteElement : Element 인터페이스를 구현한 클래스. Visitor 객체가 수행해야 하는 작업 수행
-   Object Structure : ConcreteElement 객체들의 집합

<br/>

### 구현 예제

우선 Visitor 인터페이스를 정의한다.

```java
public interface Visitor {
    void visit(Computer computer);
    void visit(Phone phone);
}
```

visit 메서드가 호출되는 객체인 Element의 구현체들을 방문하기 위한 메서드를 제공한다.

<br/>

Visitor 인터페이스를 구현한 ConcreteVisitor 클래스를 만든다.

```java
public class OfflineOrder implements Visitor {

    @Override
    public void visit(Computer computer) {
        System.out.println("매장에서 " + computer.getName() + " 을 주문합니다." );
    }

    @Override
    public void visit(Phone phone) {
        System.out.println("매장에서 " + phone.getName() + " 을 주문합니다." );
    }
}

public class OnlineOrder implements Visitor {

    @Override
    public void visit(Computer computer) {
        System.out.println("온라인으로 " + computer.getName() + " 을 주문합니다." );
    }

    @Override
    public void visit(Phone phone) {
        System.out.println("온라인으로 " + phone.getName() + " 을 주문합니다." );
    }
}
```

ConcreteVisitor 클래스는 각 Element 객체를 방문할 때 해당 객체에 맞는 특정 작업을 수행한다. 즉, Element 객체의 구체적인 구현과는 독립적으로 작업을 수행할 수 있다. <br/>
→ _“객체 구조와 작업을 분리함으로써, 새로운 작업을 추가하거나 기존 작업을 변경하기가 더욱 쉬워진다.”_

<br/>

Visitor 객체가 방문할 객체들이 구현해야 하는 인터페이스인 Element 인터페이스를 정의한다.

```java
public interface Product {
    void accept(Visitor visitor);
}
```

accept 메서드는 Visitor 객체를 인자로 받아들이고, Visitor 객체가 해당 Product 객체를 방문할 때 호출된다. accept 메서드를 호출함으로 인해 Element 객체는 Visitor 객체에 자신을 전달할 수 있게 된다.

<br/>

Element 인터페이스를 구현한 ConcreteElement 클래스를 생성한다.

```java
public class Computer implements Product {

    private final String name;

    public Computer(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

public class Phone implements Product {

    private final String name;

    public Phone(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}
```

Element 인터페이스의 accept 메서드를 오버라이드하여, 인자로 전달된 Visitor 객체의 visit 메서드를 호출하면서 this(자기 자신)을 전달한다. 이를 통해 Visitor 객체는 자신이 수행해야 할 작업을 Element 객체에 적용할 수 있게 된다.<br/>
→ _“Visitor 객체에 대한 의존성을 갖지 않으면서도 Visitor 객체가 자신을 방문할 수 있도록 구성할 수 있다.”_

<br/>

마지막으로 Element 인터페이스를 구현한 객체들을 관리하는 Object Structure 클래스를 만든다.

```java
public class ObjectStructure {
    private final List<Product> productList = new ArrayList<>();

    public void attach(Product product) {
        productList.add(product);
    }

    public void detach(Product product) {
        productList.remove(product);
    }

    public void accept(Visitor visitor) {
        productList.forEach(p -> p.accept(visitor));
    }
}
```

attach 메서드를 사용하여 객체를 추가하고, detach 메서드를 사용하여 객체를 제거할 수 있다. 또한, accept 메서드를 사용하여 모든 Element 객체에 대해 accept 메서드를 호출하여 Visitor 객체가 방문할 수 있게 한다.<br/>
→ _“ObjectStructure 클래스의 코드를 변경하지 않고도 새로운 작업을 추가할 수 있다.”_

<br/>

방문자 패턴을 사용하는 클라이언트 코드이다.

```java

ObjectStructure products = new ObjectStructure();

Computer computer = new Computer("삼성 컴퓨터");
Phone galaxy = new Phone("Galaxy S23");
Phone iphone = new Phone("아이폰 14");

products.attach(computer);
products.attach(galaxy);
products.attach(iphone);

products.accept(new OfflineOrder());

// product 제거
products.detach(iphone);
products.accept(new OnlineOrder());

```

ObjectStructure 클래스를 통해 Product 객체들을 관리하고 모든 Product 객체들에 대해 Visitor의 visit 메서드를 호출하여 Visitor가 처리할 수 있도록 한다. OfflineOrder와 OnlineOrder 클래스는 Visitor 인터페이스를 구현하는 데 각각의 클래스에 따라 각각 다른 동작을 수행하게 된다.

<br/>

### 방문자 패턴 고려사항

> 객체 구조와 작업을 분리해야 하는 경우, 구조를 변경하지 않고 새로운 동작을 추가해야 할 일이 많은 경우에는 방문자 패턴을 유용하게 사용할 수 있다. 하지만 구조가 자주 변경되거나 객체의 타입이 많은 경우에는 방문자 패턴을 고려해보아야 한다.

<br/>

**방문자 패턴 사용 시 장점**

1.  다형성을 적극 활용하므로 유연하고 확장성이 뛰어나다.
2.  구조를 변경하지 않고도 새로운 작업을 추가할 수 있다.
3.  여러 객체에 대해 연산을 수행하는 알고리즘을 구현할 때 유용하다.

<br/>

**방문자 패턴 사용 시 단점**

1.  객체의 구조가 변경되면 Visitor 인터페이스의 모든 구현체를 수정해야 한다.
2.  구현체가 많아지면 코드가 복잡해질 수 있다.
3.  한 번에 처리해야 하는 요소의 수가 많은 경우 성능 문제가 발생할 수 있다.


<br/><br/><br/>