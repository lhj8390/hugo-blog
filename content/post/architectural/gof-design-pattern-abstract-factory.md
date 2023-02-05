+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 추상 팩토리 (Abstract Factory) 패턴"
date = "2023-02-05"
description = "GoF 디자인 패턴 중 추상 팩토리 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "abstract Factory", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++
## 추상 팩토리 (Abstract Factory) 패턴

---

> <span class="red">생성(Creational) 패턴 중 하나</span>로 구체적인 서브 클래스에 의존하지 않고 서로 연관되거나 의존적인 객체의 조합을 만드는 인터페이스를 제공하는 패턴
> 

상속 관계에 있는 객체를 생성하지만 그 클래스 이름을 고정적으로 지정하지 않고 생성해야 할 때 유용하다.
{{< figure src="/images/gof-design-pattern-abstract-factory/1.png" alt="image" >}}

### 추상 팩토리 구성요소

> 추상 팩토리, 구체적인 팩토리, 추상 제품, 구체적인 제품으로 총 4개의 컴포넌트로 구성된다.
> 
- **추상 팩토리**(AbstractFactory) : 제품을 생성하는 메서드를 선언하는 인터페이스
- **구상 팩토리**(ConcreteFactory) : 추상 팩토리를 구현하고 구체적인 제품에 대한 객체를 생성하는 메소드를 구현
- **추상 제품**(AbstractProduct) : 개념적 제품 객체에 대한 인터페이스를 정의한다.
- **구상 제품**(ConcreteProduct) : 구체적으로 팩토리가 생성할 객체를 정의하고 추상 제품의 인터페이스를 구현한다.

### 구현 예제

우선 FactoryProducer 클래스를 만든다.

```java
public class FactoryProducer {

    static BaseProductFactory getFactory(ProductType type) {
        switch (type) {
            case COMPUTER:
                return new ComputerFactory();
            case TABLET:
                return new TabletFactory();
            case PHONE:
                return new PhoneFactory();
            default:
                throw new IllegalArgumentException();
        }
    }
}
```

추상 팩토리 패턴에서 중요한 역할을 하는 클래스로, 어떤 팩토리 구상 팩토리(ConcreteFactory)를 생성할지 결정하고, 그에 맞는 팩토리 객체를 생성하여 반환한다. <br/>
→ *“추상 팩토리와 구상 팩토리 간의 의존성을 감소시켜줌으로써 프로그램의 유연성을 높일 수 있다”*

<br/>

추상 팩토리(AbstractFactory) 클래스를 만들어준다.

```java
public abstract class BaseProductFactory {

    public abstract Product createProduct(String name);
}
```

각 <span class="red">구상 팩토리(ConcreteFactory)에서 구현해야 하는 메소드를 정의</span>한다. → <span class="gray">*예제에서는 createProduct() 메소드*</span>  추상 메소드를 제공함으로써 각 구상 팩토리(ConcreteFactory)에서는 제품의 생성 방법만 구현하면 된다. <br/>
→ *“구상 팩토리(ConcreteFactory) 간의 중복 코드를 줄이고 가독성을 높인다.”*

<br/>

구상 팩토리(ConcreteFactory) 클래스를 만들어준다.

```java
public class ComputerFactory extends BaseProductFactory {
    @Override
    public Product createProduct(String name) {
        return new Computer(name);
    }
}
```

BaseProductFactory 클래스(추상 팩토리)를 상속받아 createProduct() 메소드에서 구상 제품(ConcreteProduct) 객체를 반환한다. 즉, 구상 팩토리(ConcreteFactory) 클래스에서는 <span class="red">제품의 생성 방법을 정의하고 추상 팩토리(AbstractFactory)에서 제공하는 추상 메소드를 구현</span>한다. <br/>
→ *“프로그램의 구조를 깔끔하게 유지할 수 있다”*

<br/>

추상 제품(AbstractProduct) 클래스를 만들어준다.

```java
public abstract class Product {

    public abstract void use();

    public void buy() {
        System.out.println("제품을 구매하였습니다.");
    }
}
```

각 구상 제품(ConcreteProduct)에서 구현해야 하는 <span class="red">공통적인 메소드를 정의</span>한다. 예제에서는 use() 메소드와 buy() 메소드를 정의하였는데 구상 제품(ConcreteProduct) 클래스에서는 use() 메소드와 buy() 메소드의 구체적인 기능을 정의할 수 있다.<br/>
→ *“패턴의 일관성을 유지하면서도 제품의 종류에 따라 다양한 기능을 정의할 수 있다”*

<br/>

구상 제품(ConcreteProduct) 클래스를 만들어준다.

```java
public class Computer extends Product {
    private final String name;
    public Computer(String name) {
        this.name = name;
    }

    @Override
    public void use() {
        System.out.println(name + " 컴퓨터를 사용합니다.");
    }
}
```

Product 클래스(추상 제품)를 상속받아 <span class="red">추상 메소드에 대한 구체적인 기능을 구현</span>한다. 예제에서는 buy() 메소드를 추상 제품 클래스인 Product 클래스에서 정의한 그대로 사용하고 use() 메소드를 구체적으로 정의하였다.<br/>
→ *“제품의 종류에 따라 구체적인 클래스를 생성할 수 있어, 다양한 제품을 쉽게 구현할 수 있다”*


### 추상 팩토리 패턴 고려사항

> 클라이언트 구성 요소와 객체 생성 과정을 분리하는 데 유용하지만, 코드의 가독성이 떨어질 수 있기 때문에 상황에 따라 적절하게 사용해야 한다.<br/>
> 새로운 객체 생성 로직이 추가될 가능성이 높은 경우에 이 패턴을 적용해볼만 하다.
> 

<br/>

**추상 팩토리 패턴 적용 시 장점** 
1. 구체적인 클래스에 대한 결합도가 낮아진다. (안정성 ⬆️)
2. 객체 생성 과정이 추상화되어 클라이언트 구성 요소에 대한 의존성이 줄어든다.
3. 개발 및 구축 과정에서 새로운 객체 생성 로직 추가 및 변경이 쉽다.
<br/><br/>

**추상 팩토리 패턴 적용 시 단점**

1. 클래스 수가 증가하고, 코드의 복잡성이 증가한다.
2. 새로운 제품을 추가할 때, 팩토리 구조를 변경할 수도 있다.

<br/>