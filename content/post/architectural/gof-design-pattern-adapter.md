+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 적응자(Adapter) 패턴"
date = "2023-05-01"
description = "GoF 디자인 패턴 중 적응자(Adapter) 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "adapter", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

> <span class="red">구조(structural) 패턴</span> 중 하나로, 호환되지 않는 인터페이스를 가진 두 개의 클래스를 함께 사용할 수 있는 디자인 패턴이다.

기존의 클래스와 새로운 인터페이스 간의 호환성을 제공하기 위해 중간에 어댑터(Adapter) 클래스를 두어 기존 클래스의 인터페이스를 새로운 인터페이스로 변환할 수 있다.

{{<figure src="/images/gof-design-pattern-adapter/1.png" class="large" caption="Object Adapter">}}

{{<figure src="/images/gof-design-pattern-adapter/2.png" class="large" caption="Class Adapter">}}

<br/>

### 적응자 패턴 구성요소

-   Target : Client가 사용할 새로운 인터페이스
-   Adaptee : 기존에 존재하는 <span class="ul">새로운 인터페이스를 구현하지 않는</span> 클래스
-   Adapter : Target 인터페이스를 구현하면서 Adaptee 인스턴스를 감싸 <span class="red">새로운 인터페이스에 맞게 변환해주는 클래스</span>


### 구현 예제

> 적응자 패턴은 Class Adapter, Object Adapter 두가지로 나뉜다. Class Adapter는 상속을 통해 구현되고, Object Adapter는 기존 클래스 객체(Adaptee) 를 다른 객체 내부에 포함시켜 새로운 인터페이스로 구현하는 방식이다.

<br/>

<span class="bg_b wide">**우선 Class Adapter를 구현한 예제이다.**</span>

기존에 존재하는 Adaptee 클래스를 만들어준다.

```java
public class GoogleAdaptee {
    public void translateByGoogle() {
        System.out.println("구글 번역기를 사용합니다.");
    }
}
```

이 GoogleAdaptee 클래스는 수정이 불가능한 상황이고 새로운 Target 인터페이스에서 GoogleAdaptee 기능을 사용하고 싶다.

<br/>

새로운 인터페이스인 Target 인터페이스를 만들어준다.

```java
public interface PapagoTranslation {
    void translateByPapago();
}
```

Adapter가 구현해야 할 Target 역할을 한다.

<br/>

Adapter 클래스에 Adaptee를 사용하는 새로운 인터페이스를 구현한다.

```java
public class PapagoAdapter extends GoogleAdaptee implements PapagoTranslation {

    @Override
    public void translateByPapago() {
        System.out.println("파파고 번역기에서 구글 번역 기능을 사용합니다.");
        this.translateByGoogle();
    }
}
```

Adaptee 와 Target을 연결해주는 Adapter의 역할을 한다. PapagoAdapter 클래스는 GoogleAdaptee 클래스를 상속받기 때문에 Adaptee 객체를 가지고 있고, PapagoTranslation 인터페이스를 구현하므로 Target 인터페이스의 역할을 수행할 수 있다.<br/>
→ “_호환되지 않는 두 개의 클래스를 함께 사용할 수 있게 되어 유연성이 증가한다._”

<br/>

<span class="bg_b wide">다음은 **Object Adapter를 구현한 예제이다.**</span>

Class Adapter 예제와 똑같은 Adaptee 클래스와 Targer 인터페이스를 만들어주었다.

```java
public class GoogleAdaptee {
    public void translateByGoogle() {
        System.out.println("구글 번역기를 사용합니다.");
    }
}
public interface PapagoTranslation {
    void translateByPapago();
}
```

GoogleAdaptee 클래스는 기존에 존재하는 클래스이고, PapagoTranslation 인터페이스는 Client가 새로 구현하려는 인터페이스이다.

<br/>

Adapter 클래스를 만들어준다.

```java
public class PapagoAdapter implements PapagoTranslation {
    private final GoogleAdaptee googleAdaptee;

    public PapagoAdapter(GoogleAdaptee googleAdaptee) {
        this.googleAdaptee = googleAdaptee;
    }
    @Override
    public void translateByPapago() {
        System.out.println("파파고 번역기에서 구글 번역 기능을 사용합니다.");
        googleAdaptee.translateByGoogle();
    }
}
```

Class Adapter과 다르게 Adaptee 객체를 인수로 받아서 Target 인터페이스를 구현한다.

Class Adapter와 Object Adapter에 대해 요약하자면 다음과 같다.
- <font color="#d99694"> Class Adapter</font>
	- 장점 : Adaptee를 오버라이드 할 수 있다. Adaptee 객체를 만들지 않아도 된다.
	- 단점 : 다중 상속이 지원되는 언어에서만 사용할 수 있다. 특정 Adaptee 클래스만 적용할 수 있다.
- <font color="#d99694">Object Adapter</font>
	- 장점 : 객체 합성을 사용하기 때문에 유연하다.
	- 단점 : Adaptee 객체를 만들어야 한다.


<br/>

### 적응자 패턴 고려사항

> 적응자 패턴은 호환되지 않는 인터페이스를 두 개의 클래스와 함께 사용할 경우 유용하게 사용할 수 있다. <span class="red">기존 코드를 재사용하거나 수정을 최소화할 경우</span> 적응자 패턴을 사용하는 것은 좋은 방법이지만, <span class="red">클래스 간의 호환성 문제가 없거나 클래스 간의 차이가 너무 크고 여러 개의 클래스를 함께 사용해야 할 때</span>는 코드가 복잡해 질 수 있다.

<br/>

**적응자 패턴 사용 시 장점**

1.  기존 코드의 재사용성을 높일 수 있다.
2.  새로운 인터페이스를 추가할 때, 기존 코드 수정을 최소화할 수 있다.

<br/>

**적응자 패턴 사용 시 단점**

1.  Object Adapter 패턴을 사용할 경우 추가적인 객체 생성으로 인해 오버헤드가 발생할 수 있다.
2.  Class Adapter 패턴을 사용할 경우 이미 다른 클래스를 상속하고 있는 경우 적응자 패턴을 만들 수 없다는 한계가 있다.

<br/>
<br/>
<br/>