+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 팩토리 메서드 (Factory Method) 패턴"
date = "2023-02-26"
description = "GoF 디자인 패턴 중 팩토리 메서드 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "factory method", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++
## 팩토리 메서드(Factory Method)

---

> <span class="red">생성(Creational) 패턴 중 하나</span>로 객체를 생성하기 위해 인터페이스를 정의하지만, 어떤 클래스의 인스턴스를 생성할지는 서브클래스가 결정하도록 하는 패턴

객체 생성의 책임을 몇 개의 보조 서브클래스 가운데 하나에게 위임하고, 어떤 서브클래스가 위임자인지에 대한 정보를 국소화시키고 싶을 때 주로 사용한다.

{{<figure src="/images/gof-design-pattern-factory-method/1.png" class="large">}}

### 팩토리 메서드 구성요소

> 4가지 구성 요소들이 상호 작용하여 객체 생성을 캡슐화한다.
> 
- **생성자**(Creator) : 제품 객체를 생성하는 추상 클래스 또는 인터페이스
- **구체적인 생성자**(ConcreteCreator) : Creator 클래스를 상속하여 구체적인 제품을 생성하는 클래스. 팩토리 메서드를 구현하여 실제 객체를 생성
- **제품**(Product) : 팩토리 메서드로 생성되는 객체의 공통 인터페이스
- **구상 제품**(ConcreteProduct) : Product 클래스에 정의된 인터페이스를 실제로 구현

### 구현 예제

Creator 추상 클래스를 만들어준다.

```java
public abstract class Creator {

    public abstract Product createProduct();

    void useProduct() {
        Product product = createProduct();
        product.use();
    }
}
```

제품 객체를 생성하는 추상 메서드를 구현한다. 클라이언트 코드에서는 <span class="red">구체적인 제품 객체를 직접 생성하지 않고 추상화된 인터페이스를 통해 제품 객체를 생성하고 사용</span>할 수 있다.<br/>
→ “*객체 생성을 추상화하여 클라이언트 코드와 구체 클래스(Concrete Creator) 간의 의존성을 줄여주고 유지보수성과 확장성을 높일 수 있다.*”

<br/>

ConcreteCreator 클래스를 만든다.

```java
public class ComputerCreator extends Creator {
    @Override
    public Product createProduct() {
        return new Computer();
    }
}
```

Creator 추상 클래스를 상속받은 구체적인 클래스로 Computer 객체를 생성하는 구체적인 생성 로직을 구현한다.<br/>
→ “*Creator 클래스와 Product 인터페이스에만 의존하면서 실제 생성되는 객체의 타입을 유연하게 변경할 수 있다.*”

<br/>

Product 인터페이스를 구현한다.

```java
public interface Product {

    void use();
}
```

팩토리 메서드 패턴에서 생성될 구체적인 제품 객체들의 공통 기능을 선언한다. 

<br/>

마지막으로 ConcreteProduct 클래스를 만들어준다.

```java
public class Computer implements Product {
    @Override
    public void use() {
        System.out.println("컴퓨터를 사용합니다.");
    }
}
```

Product 인터페이스에서 정의된 use() 메서드를 구현하여 실제로 동작하는 방식을 구현한다.<br/>
→ “*팩토리 메서드 패턴에서 실제로 사용되는 제품을 나타내는 역할을 한다*”


### 팩토리 메서드 패턴 고려사항

> 객체 지향 설계 원칙 중 하나인 개방-폐쇄 원칙(OCP)을 준수하여 기능 확장에 유리하지만, 단순한 객체 생성이나 생성자 인자의 조합으로 생성되는 객체의 경우에는 구조가 복잡해지기 때문에 사용하지 않는 편이 좋다.
> 
> 
> 객체 생성 과정이 복잡하거나 다양할 때 이 패턴을 적용해볼만 하다.
> 

<br/>

**팩토리 메서드 사용 시 장점**

1. 제품을 생성하는 코드와 제품을 사용하는 코드를 분리하여 유연성이 높다.
2. 객체를 생성하는 공통적인 인터페이스를 제공하여 코드의 중복을 줄일 수 있다.
3. 코드의 가독성과 유지보수성이 올라간다.

<br/>

**팩토리 메서드 사용 시 단점**

1. 클래스의 수가 늘어나면 코드의 복잡도가 증가한다.
2. 팩토리 클래스와 제품 클래스를 별도로 작성해야 하기 때문에 복잡할 수 있다.

<br/>