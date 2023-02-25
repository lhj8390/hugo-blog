+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 원형 (Prototype) 패턴"
date = "2023-02-25"
description = "GoF 디자인 패턴 중 원형, prototype 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Prototype", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++
## 원형 (Prototype)

---

> <span class="red">생성(Creational) 패턴 중 하나</span>로 원형이 되는 인스턴스를 사용하여 생성할 객체의 종류를 명시하고, 만든 견본을 복사해서 새로운 객체를 생성하는 패턴

생산 비용이 높은 인스턴스를 **복사**를 통해서 쉽게 생성할 수 있다.

{{< figure src="/images/gof-design-pattern-prototype/1.png" alt="image" >}}

### 원형 패턴 구성요소

- **Prototype** : 객체를 복제하기 위한 인터페이스 정의
- **ConcretePrototype** : Prototype 인터페이스를 구현한 클래스. clone() 메서드 구현
- **Client** : Prototype 패턴을 사용하는 클라이언트. 복제를 원하는 객체를 요청하고 새로운 객체 생성

### 구현 예제

Java의 Cloneable 인터페이스를 사용하여 <span class="red">**Shallow Copy**</span> 를 구현한 예제이다.

```java
public class Person implements Cloneable {
    private Long id;
    private String name;
    private Address address;

    public Person(Long id, String name, Address address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    @Override
    public Person clone() {
        try {
            return (Person) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Address {
    private String location;
    private String zipCode;

    public Address(String location, String zipCode) {
        this.location = location;
        this.zipCode = zipCode;
    }
}
```

Clonable 인터페이스를 구현하는 클래스를 만들어준다. Clonable 인터페이스를 이용하면 손쉽게 프로토타입을 구현할 수 있다.<br/>
→ “*객체 생성 비용을 절약할 수 있다*.”

<br/>

Prototype 패턴을 사용하는 Client 코드이다.

```java
Person person = new Person(1L, "사람 1", new Address("Montgomery", "36104"));
System.out.println("Person : " + person);
System.out.println("Address : " + person.getAddress());

Person clone = person.clone();
System.out.println("clone : " + clone);
System.out.println("Address : " + clone.getAddress());
```

**Person**(ConcretePrototype) 클래스에서 구현한 clone() 메소드를 사용하여 Person 객체를 복제한다. 하지만 복제된 객체와 원본 객체의 address 필드가 동일한 Address 객체를 참조하게 된다.

<br/>

Java의 Cloneable 인터페이스를 사용하여 <span class="red">**Deep Copy**</span> 를 구현한 예제이다.

```java
public class Person implements Cloneable {
    private Long id;
    private String name;
    private Address address;

    public Person(Long id, String name, Address address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    @Override
    public Person clone() {
        try {
            Person clone = (Person) super.clone();
            **clone.setAddress(clone.getAddress().clone());**
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Address implements Cloneable {
    private String location;
    private String zipCode;

    public Address(String location, String zipCode) {
        this.location = location;
        this.zipCode = zipCode;
    }

    @Override
    public Address clone() {
        try {
            return (Address) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

Person 클래스와 Address 클래스 각각에 대해 Clonable 인터페이스를 구현한다. Person의 clone() 메서드에 Person 객체를 복제한 후 **`clone.getAddress().clone()`** 를 호출하여 Address 객체도 복제를 수행한다.

<br/>

Prototype 패턴을 사용하는 Client 코드이다.

```java
Address address = new Address("Montgomery", "36104");
Person person = new Person(1L, "사람 1", address);

System.out.println("Person : " + person);
System.out.println("Address : " + person.getAddress());
Person clone = person.clone();

System.out.println("clone : " + clone);
System.out.println("Address : " + clone.getAddress());
```

**Person**(ConcretePrototype) 클래스에서 구현한 clone() 메소드를 사용하여 Person 객체를 복제한다. Person 객체를 복제하면서 Address 객체도 같이 복제되기 때문에 원본 객체와 복제된 객체의 Address 객체는 서로 다른 주소값을 가지게 된다.

### 원형 패턴 고려사항

> 객체 생성 비용이 크거나, 다양한 종류의 객체 생성이 필요한 경우 프로토타입 패턴을 이용하면 효율적으로 객체를 생성할 수 있다. <span class="gray"><br/>ex) *DB로부터 데이터를 가져오는 것*</span><br/> 하지만 객체의 상태가 자주 변경되거나 객체 구조가 복잡한 경우 구현이 어렵기 때문에 <span class="red">객체 구조가 단순하고 복제가 간단한 경우</span>에 프로토타입 패턴을 적용하는 것이 좋다.

<br/>

**원형 패턴 사용 시 장점**

1. 객체 생성 과정을 간소화할 수 있다.
2. 객체의 생성과 관리를 중앙에서 수행하여 객체의 중복 생성과 불필요한 객체 생성을 방지한다.
3. 객체를 동적으로 생성할 수 있다.

<br/>

**원형 패턴 사용 시 단점**

1. 객체 복제를 위한 별도의 인터페이스나 메서드를 정의해야 하므로, 코드가 복잡해질 수 있다.
2. 객체 상태를 변경하는 작업이 객체의 복제를 통해 전파되지 않을 수 있다.
3. 순환 참조가 있는 클래스는 복제하기 어렵다.

<br/>