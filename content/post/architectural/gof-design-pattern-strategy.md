+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 전략 (Strategy) 패턴"
date = "2023-05-09"
description = "GoF 디자인 패턴 중 전략, Strategy 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Strategy", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

## 전략 (Strategy) 패턴

---

> <span class="red">행동(behavioral) 패턴 중 하나</span>로, 동일한 문제를 해결하기 위한 여러 알고리즘이 존재할 때, 이를 캡슐화하고 상호교환이 가능하도록 만들어주는 패턴이다.
> 

알고리즘을 사용하는 로직과 알고리즘을 제공하는 로직을 분리시켜줌으로써, 알고리즘의 변경이나 추가가 필요한 경우 유연하게 변경할 수 있다.

### 전략 패턴 구성요소

Context 객체가 Strategy 객체를 가지고 있어 Strategy 객체의 변경이나 추가가 용이하다.

- Context : 알고리즘을 사용하는 객체. 적용할 알고리즘을 선택하고 호출
- Strategy : 알고리즘을 캡슐화한 인터페이스
- Concrete Strategy : Strategy 인터페이스를 실제로 구현한 클래스. Context 객체가 필요한 Strategy 을 선택하여 호출.

### 구현 예제

결제 수단에 따라 다양한 Strategy 를 적용하는 예제를 만들어보았다. 
우선 Strategy 인터페이스를 정의한다.

```java
public interface Strategy {
    int getPoint(int price);
}
```

구체적인 Strategy 클래스(결제 수단)에 따라 적립되는 포인트를 반환하는 메서드인 getPoint() 메서드를 정의한다. 이 Strategy 인터페이스를 구현한 클래스들은 각각 다른 방식으로 알고리즘을 구현할 수 있게 된다.<br/>
→ *“다양한 Strategy 클래스를 하나의 타입으로 다룰 수 있도록 한다”*

Strategy 인터페이스를 구현한 Concrete Strategy 클래스를 만든다.

```java
public class CardStrategy implements Strategy {

    @Override
    public int getPoint(int price) {
        return (int) (price * 0.003);
    }
}

public class CashStrategy implements Strategy {

    @Override
    public int getPoint(int price) {
        return (int) (price * 0.01);
    }
}
```

Strategy 인터페이스를 구현하는 다양한 Concrete Strategy 클래스를 만들어 고유한 알고리즘을 수행하게 한다. CardStrategy 클래스는 카드 결제를 할 경우, CashStrategy 클래스는 현금 결제를 할 경우에 대한 전략을 수행하게 된다.<br/>
→ *“클라이언트에서는 인터페이스를 호출하는 것만으로도 해당 인터페이스를 구현한 다양한 전략 클래스를 이용할 수 있게 된다.”*

Strategy 클래스를 사용하는 Context 클래스이다.

```java
public class Payment {
    private final int price;
    private final Strategy strategy;

    public Payment(Strategy strategy, int price) {
        this.strategy = strategy;
        this.price = price;
    }

    public void getPoint() {
        int point = strategy.getPoint(price);
        System.out.println(point + " 포인트가 적립되었습니다!");
    }

}
```

생성자에서는 Strategy 객체와 price 정보를 받아와서 저장하고 getPoint() 메서드를 통해 해당 Strategy 객체와 price 정보에서 계산한 point 값을 반환받을 수 있다.

<br/>

전략 패턴을 사용하는 클라이언트 코드이다.

```java
Payment card = new Payment(new CardStrategy(), 10000);
card.getPoint();

Payment cash = new Payment(new CashStrategy(), 10000);
cash.getPoint();

Payment phone = new Payment(new PhoneStrategy(), 10000);
phone.getPoint();
```

전략 패턴을 사용하여 다양한 결제 방식에 따라 다른 포인트 적립 알고리즘을 적용하도록 구현하였다. <br/>
→ *“결제 방식이 추가되거나 변경될 때, Payment 클래스를 수정하지 않고도 새로운 전략 객체를 만들어 사용할 수 있기 때문에 유연성과 확장성이 높아진다.”*

### 전략 패턴 고려사항

> 알고리즘을 동적으로 선택해야 할 때, 비슷한 알고리즘을 가진 다른 클래스들이 존재할 경우에는 전략 패턴을 유용하게 사용할 수 있다. 하지만 알고리즘의 개수가 적거나 자주 변경되지 않는 경우에는 직접 구현하는 것이 더 적합하다.
> 

<br/>

**전략 패턴 사용 시 장점**

1. 알고리즘을 변경하거나 추가하기 쉽다. - *확장성/유연성 증가*
2. 캡슐화가 잘 되어 있어 가독성이 좋아지고 유지보수하기 쉬워진다.
3. 각각의 Strategy 클래스에 대해 개별적으로 테스트가 가능하다. - *효과적으로 테스트를 수행할 수 있다.*

<br/>

**전략 패턴 사용 시 단점**

1. 구현할 Strategy 클래스가 많아져 코드가 복잡해질 수 있다.
2. Strategy 객체를 생성하고 사용하기 위한 오버헤드가 있을 수 있다.

<br/><br/>
<br/><br/>
