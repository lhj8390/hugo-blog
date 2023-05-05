+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 반복자 (Iterator) 패턴"
date = "2023-05-05"
description = "GoF 디자인 패턴 중 반복자 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Iterator", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

> <span class="red">행동(behavioral) 패턴 중 하나</span>로, 객체의 내부 구조와 상관없이 컬렉션의 요소들을 순차적으로 접근하고 처리할 수 있는 방법을 제공하는 패턴이다.

컬렉션 내에 구현이 어떤지 몰라도 해당 객체에 접근해 반복 작업을 처리할 수 있다.

<br/>

### 반복자 패턴 구성요소

-   **Iterator** : 집합체의 요소들을 순서대로 검색하는 인터페이스. 다음 요소를 가져오는 메서드와 다음 요소가 있는지 확인하는 메서드 존재
-   **Concrete Iterator** : iterator 인터페이스를 실제로 구현하는 클래스
-   **Aggregate** : 원소들의 집합을 나타내는 인터페이스. Iterator를 생성하는 메서드 포함.
-   **Concrete Aggreagate** : Aggreagate 인터페이스를 구현하는 클래스

<br/>

### 구현 예제

iterator 인터페이스를 생성한다.

```java
public interface Iterator<T> {
    boolean hasNext();
    T next();
}
```

다음 요소가 있는지 확인하는 hasNext() 메서드와 다음 요소를 반환하는 next() 메서드를 선언한다. 이를 통해 구체적인 컬렉션과 관계 없이 일관된 방식으로 순회를 할 수 있다.

<br/>

iterator 인터페이스를 구현한 Concreate Iterator 클래스를 만들어준다.

```java
public class ProductIterator implements Iterator<Product> {

    private final Product[] productList;
    private int position;

    public ProductIterator(Product[] productList) {
        this.productList = productList;
        this.position = 0;
    }

    @Override
    public boolean hasNext() {
        return (position < productList.length);
    }

    @Override
    public Product next() {
        return (this.hasNext())? productList[position++] : null;
    }
}
```

생성자를 통해 Product[] 배열을 받아오며, Product[] 배열을 순회하면서 각각의 요소를 반환하는 역할을 수행한다. hasNext() 메서드는 현재 위치(position)와 productList 배열의 길이를 비교하여 다음 요소가 있는지를 판단하고 next() 메서드는 다음 요소가 있을 경우 productList 배열에서 현재 위치(position)의 요소를 반환한다.

<br/>

Aggregate 인터페이스를 선언한다.

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

컬렉션 클래스에서 요소들을 반복해서 가져올 수 있도록 Iterator 객체를 반환하는 iterator() 메서드를 정의한다.<br/>
→ _“컬렉션 내부의 요소에 접근할 수 있는 방법을 제공한다.”_

<br/>

마지막으로 Aggregate 인터페이스를 구현하는 Concrete Aggreagate 클래스를 만든다.

```java
public class ProductCollection implements Iterable<Product> {

    private final Product[] productList;

    public ProductCollection(Product[] productList) {
        this.productList = productList;
    }

    @Override
    public Iterator<Product> iterator() {
        return new ProductIterator(productList);
    }
}
```

Product[] 배열을 가지고 있는 클래스로 iterator() 메서드를 통해 Product[] 배열을 전달하고 요소를 순회하면서 각각의 요소에 접근할 수 있게 된다.<br/>
→ _“컬렉션에서 요소에 접근하는 방법을 추상화함으로써, 컬렉션의 내부 구조를 알 필요 없이 요소에 접근할 수 있다.”_

<br/>

### 반복자 패턴 고려사항

> 컬렉션 내부의 요소를 순회할 때 요소의 타입과 무관하게 일관된 방법으로 접근하고 싶을 경우 주로 사용한다. 하지만 컬렉션 내부의 요소를 순회할 때 수정, 삭제 등의 작업을 수행하는 경우 반복자 패턴은 적합하지 않다. 또한 컬렉션 내부 구조가 너무 단순하거나 크기가 작은 경우에는 오히려 복잡도를 높이는 결과를 가져오기 때문에 사용하지 않는 편이 좋다.

<br/>

**반복자 패턴 사용 시 장점**

1.  컬렉션 내부 구조를 외부로 노출하지 않고 일관된 방법으로 컬렉션 요소에 접근할 수 있다.
2.  객체의 구현과 요소를 순회하는 방법을 분리함으로써, 객체 간의 결합도를 낮춘다.
3.  새로운 타입의 컬렉션을 추가할 때 일관된 인터페이스를 유지할 수 있어 유지보수에 용이하다.

<br/>

**반복자 패턴 사용 시 단점**

1.  Iterator 객체를 생성하고 유지하는 데 필요한 자원이 추가된다.
2.  Iterator 객체를 사용하는 데 일정한 오버헤드가 발생한다.
3.  코드의 복잡도가 증가할 수 있다.

<br/>
<br/><br/>
