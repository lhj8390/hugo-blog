+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 플라이급 (Flyweight) 패턴"
date = "2023-03-20"
description = "GoF 디자인 패턴 중 플라이급 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "flyweight", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

## 플라이급 (Flyweight) 패턴

---

> <span class="red">구조(structural) 패턴 중 하나</span>로, 객체를 공유하여 메모리 사용량을 줄일 수 있는 패턴이다.
> 

객체 생성과 소멸이 빈번하게 발생할 경우, 메모리 사용량이 많아지고 성능 저하를 유발할 수 있는데 동일한 데이터를 가지는 객체들을 하나의 객체로 만들어 객체 생성 비용과 메모리 사용량을 줄일 수 있다.

### 플라이급 패턴 구성요소

- **Flyweight** : 공유 가능한 상태를 가지는 인터페이스
- **Concrete Flyweight** : Flyweight 인터페이스를 구현하며 공유 가능한 상태 저장
- **Unshared Concrete Flyweight** : 객체의 상태가 공유되지 않는 객체 생성. 객체 생성 비용 절약이 목적
- **Flyweight Factory** : Flyweight 객체 생성 및 관리 - 동일한 객체가 여러 번 생성되는 것 방지

### 구성 예제

Flyweight 패턴을 적용하기 위해 인터페이스를 만들어준다.

```java
public interface Product {
    void buy();
}
```

Product 객체를 공유하여 생성할 때, buy() 메소드를 호출하는 코드에서는 어떤 구체적인 클래스에서 buy() 메소드를 구현했는지 몰라도 되므로 코드의 유연성과 재사용성이 향상된다.

<br/>

Flyweight 인터페이스를 구현한 SharedProduct 클래스를 만든다.

```java
public class SharedProduct implements Product {
    private final ProductType type;

    public SharedProduct(ProductType type) {
        this.type = type;
    }

    @Override
    public void buy() {
        System.out.println(type + " 을 구매합니다.");
    }
}
```

이 클래스는 플라이급 패턴에서 공유되는 객체를 표현하는데 사용된다. 같은 ProductType을 가진 객체가 이미 생성되었다면 새로 생성하지 않고 이미 생성된 객체를 공유하여 사용하는 것이 목적이다.<br/>
→ *“메모리 사용을 최적화할 수 있다.”*

<br/>

Flyweight 패턴의 반대 개념인 Unshared Concrete Flyweight를 구현한다.

```java
public class UnsharedProduct implements Product {
    private final ProductType type;
    private final int price;

    public UnsharedProduct(ProductType type, int price) {
        this.type = type;
        this.price = price;
    }

    @Override
    public void buy() {
        System.out.println(price + " 가격으로 " + type + " 을 구매합니다.");
    }
}
```

고유한 속성을 가진 객체를 반복해서 생성해야 할 때 사용한다. SharedProduct 클래스와는 다르게 객체마다 고유한 ProductType과 price를 가질 수 있다.<br/>
→ *“객체를 유연하게 생성할 수 있다.”*

<br/>

객체를 생성하고 캐싱하는 Flyweight Factory 클래스를 만든다.

```java
public class ProductFactory {
    private static final HashMap<ProductType, Product> products = new HashMap<>();

    public static Product getProduct(ProductType type) {
        if (!products.containsKey(type)) {
            products.put(type, new SharedProduct(type));
        }
        return products.get(type);
    }

    public static Product getProduct(ProductType type, int price) {
        return new UnsharedProduct(type, price);
    }

}
```

SharedProduct과 UnsharedProduct 클래스를 통합적으로 관리할 수 있는 getProduct() 메서드를 오버로딩(<span class="gray">overloading</span>) 하였다. 첫 번째 메서드는 HashMap을 사용하여 이미 생성된 Product 객체(<span class="gray">*SharedProduct*</span>)를 공유하고 두 번째 메서드는 새로운 UnsharedProduct 객체를 생성한다.

<br/>

플라이급 패턴을 사용하는 Client 코드 예시이다.

```java
Product computer1 = ProductFactory.getProduct(ProductType.COMPUTER);
computer1.buy();
Product computer2 = ProductFactory.getProduct(ProductType.COMPUTER);
computer2.buy();

System.out.println("computer1 == computer2 : " + (computer1 == computer2));

// unshared product 생성 (고유한 속성)
Product phone1 = ProductFactory.getProduct(ProductType.PHONE, 100000);
phone1.buy();
Product phone2 = ProductFactory.getProduct(ProductType.PHONE, 300000);
phone2.buy();

System.out.println("phone1 == phone2 : " + (phone1 == phone2));
```

computer 객체는 이미 생성된 적이 있으면 기존 computer 객체를 공유한다. 따라서 `computer1 == computer2` 명령어를 실행할 경우 동일한 객체이므로 true 값이 반환된다. phone 객체는 각각 고유한 속성을 가진다. 따라서 `phone1 == phone2` 명령어를 실행할 경우 서로 다른 객체이므로 false 값이 반환된다.

### 플라이급 패턴 고려사항

> 객체 생성 비용이 높거나 객체를 불변하게 유지하려는 경우에 플라이급 패턴을 효과적으로 적용할 수 있다. 하지만 객체 상태가 변경될 가능성이 있거나 객체 공유로 인해 병목 현상이 발생하는 경우에는 지양해야 한다.
> 

<br/>


**플라이급 패턴 사용 시 장점**

1. 메모리 사용량을 줄일 수 있다.
2. 객체 생성 비용을 절약할 수 있다.
3. 객체의 불변성을 유지할 수 있다.

<br/>

**플라이급 패턴 사용 시 장점**

1. 공유 객체에 대한 동시 접근이 많아질 경우 성능 저하를 발생시킬 수 있다.
2. 공유 객체 수정 시 모든 참조 객체에 영향을 줄 수 있다.

<br/>
<br/>
