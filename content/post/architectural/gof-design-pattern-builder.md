+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 빌더 (builder) 패턴"
date = "2023-02-23"
description = "GoF 디자인 패턴 중 빌더 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "builder", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++
## 빌더 (builder) 패턴

---

> <span class="red">생성(Creational) 패턴 중 하나</span>로 복잡한 객체의 생성 과정과 표현 방법을 분리하여, 서로 다른 표현이더라도 동일한 절차로 생성하는 방법을 제공한다.
> 

객체를 생성하는 과정이 복잡할 때 깔끔하게 관리할 수 있고, 객체를 생성할 때 필요한 데이터만을 골라 호출할 수 있다.

{{< figure src="/images/gof-design-pattern-builder/1.png" alt="image" >}}

### 빌더 패턴 구성요소

> Builder, ConcreteBuilder, Director, Product로 총 4개의 컴포넌트로 구성된다.
> 
- **Builder** : Product 객체의 일부 요소를 생성하기 위한 추상 인터페이스를 정의한다.
- **ConcreteBuilder** : Builder 클래스에 정의된 인터페이스를 구현한다.
- **Director** : ConcreteBuilder와 클라이언트 사이에서 중계자 역할을 수행한다.
- **Product** : 생성될 대상의 객체를 나타내는 클래스이다.

### 구현 예제

Product 객체를 만들어준다.

```java
public class Product {
    private String name;
    private int price;
    private String type;

    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public void setType(String type) {
        this.type = type;
    }

    @Override
    public String toString() {
        return "Product{" +
                "name='" + name + '\'' +
                ", price=" + price +
                ", type='" + type + '\'' +
                '}';
    }
}
```

최종적으로 만들어질 객체의 데이터 타입, 구조, 속성 등을 표현한다. 각각의 Builder는 이 Product 클래스의 객체를 생성하고, 이를 구성하는 작업을 수행한다.<br/>
→ *“Builder 패턴의 중심이다”*

<br/>

Builder 클래스를 만들어준다.

```java
abstract class Builder {
    Product product = new Product();

    public abstract Builder buildName();
    public abstract Builder buildPrice();
    public abstract Builder buildType();

    public Product getResult() {
        return product;
    }
}
```

Product 객체를 생성하고 각 구성요소의 build 메소드를 정의한다. 생성된 Product 객체를 반환하는 getResult() 메소드도 가지고 있다.<br/>
→ *“<span class="red">Builder 패턴의 구조만 제공</span>하고 추상 메서드의 실제 구현과 제품 클래스는 특정 문제에 대한 요구 사항에 따라 달라질 수 있다”*

<br/>

ConcreteBuilder 클래스를 만들어준다.

```java
public class ComputerBuilder extends Builder {
    @Override
    public Builder buildName() {
        product.setName("LG gram");
        return this;
    }

    @Override
    public Builder buildPrice() {
        product.setPrice(3000000);
        return this;
    }

    @Override
    public Builder buildType() {
        product.setType("computer");
        return this;
    }
}
```

Product의 다양한 요소를 조합하여 원하는 객체를 생성하기 위한 Builder 클래스의 추상 메소드를 구현한다. 생성한 객체는 Director 클래스에 의해 관리된다.<br/>
→ *“Product 객체에 대한 구체적인 정의를 가지고 있어 객체 생성 과정을 보다 유연하게 만든다”*

<br/>

Director 클래스를 만들어준다.

```java
public class Director {
    private final Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public Product construct() {
        return builder.buildName()
                .buildPrice()
                .buildType()
                .getResult();
    }
}
```

클라이언트는 ConcreteBuilder 클래스의 메소드를 직접 호출하는 것이 아니라, Director 클래스를 통해 ConcreteBuilder 클래스의 construct() 메소드를 사용하여 호출하게 된다.<br/>
 → *“ConcreteBuilder 클래스에서 생성한 객체의 구성 과정을 제어하고, <span class="red">원하는 객체를 생성하기 위한 구성 과정을 통일적으로 관리하는 역할</span>을 수행한다.”*

### 빌더 패턴 고려사항

> 빌더 패턴은 복잡한 객체 생성을 단순화하고 객체의 속성을 분리하여 생성할 수 있기 때문에 유연하다.<br/>
> 객체의 속성이 많거나 객체 생성 과정에서 유연성이 요구되는 경우, 객체 생성 과정이 복잡할 때 빌더 패턴을 고려해 볼 수 있다.

<br/>

**빌더 패턴 적용 시 장점**

1. 객체의 생성 과정이 복잡할 때 코드 가독성을 높여준다.
2. 생성 과정에서 필요한 변수를 명확하게 구분할 수 있어 생성 과정의 오류를 줄여준다.
3. 생성 과정에서 다양한 타입의 객체를 생성할 수 있어서 유연성이 높아진다.

<br/>

**빌더 패턴 적용 시 단점**

1. 생성 과정이 복잡한 객체를 생성할 때 구조가 복잡해질 수 있다.
2. 객체 생성 과정에서 필요한 변수가 많아질 경우 Builder 클래스의 코드가 복잡해질 수 있다.
3. 객체의 상태를 변경할 때 마다 새로운 객체를 생성하게 되어 성능이 저하될 수 있다.

<br/>