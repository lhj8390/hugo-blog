+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 장식자(Decorator) 패턴"
date = "2023-03-09"
description = "GoF 디자인 패턴 중 장식자 패턴, Decorator 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Decorator", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

## 장식자(Decorator) 패턴

---

> <span class="red">구조(structural) 패턴 중 하나</span>로, 기존 객체에 새로운 기능을 동적으로 추가할 수 있도록 하는 패턴이다.

기능을 추가하기 위해 서브클래스를 생성하는 것보다 융통성 있는 방법을 제공한다.

### 장식자 패턴 구성요소

- **Component** : 기본 객체의 추상화된 인터페이스
- **Concrete Component** : Component의 구현체로 실제로 기능을 수행하는 객체
- **Decorator** : Component의 인터페이스를 구현하며 Component를 감싸는 역할 수행
- **Concrete Decorator** : Decorator 클래스의 서브클래스로, 추가적인 기능을 제공하는 구체적인 클래스

### 구현 예제

Component 인터페이스를 선언한다.

```java
public interface Pizza {
    void addIngredients();
}
```

Pizza는 추상화된 개념으로, 이를 구현하는 클래스는 실제 피자를 나타낸다.

<br/>

Component 인터페이스를 구현하는 Concrete Component 객체를 만들어준다.

```java
public class BasePizza implements Pizza {
    @Override
    public void addIngredients() {
        System.out.println("기본 재료로 피자를 만듭니다.");
    }
}
```

Pizza 인터페이스에서 선언된 `addIngredients()` 를 오버라이드하여 기본적인 기능을 제공한다.

<br/>

Component 인터페이스를 구현하고, Component 객체를 감싸는 Decorator 추상 클래스를 정의한다.

```java
public abstract class ToppingDecorator implements Pizza {
    Pizza basePizza;

    public ToppingDecorator(Pizza basePizza) {
        this.basePizza = basePizza;
    }

    @Override
    public void addIngredients() {
        basePizza.addIngredients();
    }
}
```

추상 클래스로 구현되어 있기 때문에 직접 객체를 생성할 수 없으나 이 클래스를 상속받은 구체적인 클래스를 정의하여 객체를 생성할 수 있다.<br/>
→ *“객체의 기능을 동적으로 추가하거나 제거할 수 있어 코드의 유연성과 확장성이 증가한다.”*

<br/>

추가적인 기능을 제공하는 Concrete Decorator 클래스를 만든다.

```java
public class Meat extends ToppingDecorator {
    public Meat(Pizza basePizza) {
        super(basePizza);
    }

    @Override
    public void addIngredients() {
        super.addIngredients();
        System.out.println("고기를 추가합니다.");
    }
}
```

Meat 객체는 기존 객체에 덮어씌워지는 형태로 사용되며, 기존 객체의 기능에 추가적인 기능을 넣을 수 있다.<br/>
→ *“객체의 기능을 쉽게 확장하고 유지보수할 수 있으며, 코드의 재사용성과 확장성을 높일 수 있다.”*

<br/>

decorator 패턴을 사용하는 예시이다.

```java
Pizza basePizza = new BasePizza();
basePizza.addIngredients();

Pizza cheesePizza = new Cheese(basePizza);
cheesePizza.addIngredients();

Pizza meatPizza = new Meat(new Cheese(basePizza));
meatPizza.addIngredients();
```

우선 기본 피자 객체인 basePizza 객체를 생성한다. 그 다음 basePizza 객체를 감싸는 형태로 Cheese, Meat 이름의 Concrete Decorator 클래스를 생성하여 추가적인 기능을 수행하도록 했다. cheesePizza, meatPizza 객체는 각자 다른 재료를 추가하여 다양한 종류의 피자를 생성할 수 있다.<br/>
→ *“OCP(Open-Closed Principle) 원칙을 준수하면서 기능을 추가하거나 수정할 수 있는 유연성을 제공한다.”*

### 장식자 패턴 고려사항

> 기존 객체에 동적으로 새로운 기능을 추가할 때 유용하게 사용할 수 있지만 <span class="red">객체의 생성과 사용이 분리되어 있는 경우, 객체 구조가 복잡한 경우 장식자 패턴을 사용하기 어렵다.</span>
> 

<br/>

**장식자 패턴 사용시 장점**
1. 적으로 새로운 기능을 추가할 수 있기 때문에 확장성이 높아진다.
2. 객체의 책임을 분리함으로써 단일 책임 원칙을 준수할 수 있다. (OCP 원칙 준수)<br/>
    기존 객체는 자신이 수행해야 하는 책임에만 집중
    

<br/>

**장식자 패턴 사용시 단점**
1. 객체를 래핑하여 추가적인 기능을 부여하는 방식이기 때문에 구조가 복잡해진다.
2. 추가적인 기능을 구현하기 위해 객체를 계속해서 생성하기 때문에 객체 수가 증가한다.
3. 상속을 사용하여 구현되므로 상속 구조에 제한이 생길 수 있다.

<br/>