+++
author = "lhj8390"
title = "[GoF 디자인 패턴] 해석자 (Interpreter) 패턴"
date = "2023-03-23"
description = "GoF 디자인 패턴 중 해석자 패턴에 대해 설명하고 적용해본다."
tags = ["GoF design pattern", "Interpreter", "architectural", "design pattern"]
categories = [
    "architecture"
]
subcategories = ["Design pattern"]
+++

## 해석자 (Interpreter) 패턴

---

> <span class="red">행동(behavioral) 패턴 중 하나</span>로, 문법에 대한 표현을 정의하고 이를 이용해 언어/문법을 해석하는 패턴이다.
> 

컴파일러나 인터프리터에서 사용되며, 일반적으로 문법 규칙을 정의하고 이에 대한 해석 클래스들을 만든다. 각 클래스들은 특정 규칙을 해석하며, 이들의 조합으로 전체 문장을 해석할 수 있다.

### 해석자 패턴 구성요소

- **AbstractExpression**(추상 표현식) : 해석할 문법 규칙에 대한 추상 클래스. 해석 메서드 정의
- **TerminalExpression**(단말 표현식) : 추상 표현식에서 정의한 해석 메서드를 구현한 클래스. 단순한 값 해석
- **NonterminalExpression**(비단말 표현식) : 추상 표현식에서 정의한 해석 메서드를 구현한 클래스. 더 복잡한 문법 규칙을 해석하는 데 사용
- **Context**(문맥) : 해석에 필요한 정보 제공
- **Client**(클라이언트) : 구문을 분석하는 클라이언트

### 구현 예제

**AbstractExpression** 역할을 하는 인터페이스를 선언한다.

```java
public interface Expression {
    boolean interpret(Context context);
}
```

해석을 위한 interpret() 메서드를 선언한다.

<br/>

해석에 필요한 정보를 제공하는 Context 클래스를 만든다.

```java
public class Context {
    private final String context;

    public Context(String context) {
        this.context = context;
    }

    public boolean lookup(String data) {
        return context.contains(data);
    }
}
```

context 변수를 통해 해석 규칙에서 사용하는 변수나 데이터를 저장하고, lookup() 메서드는 data 인자를 받아 data 문자열이 포함되어 있는지 여부를 반환한다.

<br/>

Expression 인터페이스를 구현한 TerminalExpression 클래스를 만든다.

```java
public class TerminalExpression implements Expression {

    private final String data;

    public TerminalExpression(String data) {
        this.data = data;
    }

    @Override
    public boolean interpret(Context context) {
        return context.lookup(data);
    }
}
```

해석 규칙의 마지막에 나오는 단순한 규칙(터미널 규칙)을 처리하는데 사용한다. <br/>
<span class="gray">(ex) A and B : “A” 또는 “B” 가 터미널 규칙</span> <br/>
→ *“해석 규칙에 대한 최종 판단을 담당한다.”*

<br/>

Expression 인터페이스를 구현한 NonterminalExpression 클래스를 만든다.

```java
public class AndExpression implements Expression {

    private final Expression expr1;
    private final Expression expr2;

    public AndExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }

    @Override
    public boolean interpret(Context context) {
        return expr1.interpret(context) && expr2.interpret(context);
    }
}

public class OrExpression implements Expression {

    private final Expression expr1;
    private final Expression expr2;

    public OrExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }

    @Override
    public boolean interpret(Context context) {
        return expr1.interpret(context) || expr2.interpret(context);
    }
}

```

Expression 객체들의 조합에 대한 규칙을 정의하며, 해당 규칙에 맞게 Expression 객체들을 연결한다. 하나 이상의 하위 규칙을 갖고 있으며, 규칙 트리의 가장 상위 규칙이 될 수 있다.

<br/>

사용자가 실제로 사용하는 Client 클래스이다.

```java
String data = "Computer Tablet";
Context context = new Context(data);
Expression terminal1 = new TerminalExpression("Computer");
Expression terminal2 = new TerminalExpression("Phone");
Expression and = new AndExpression(terminal1, terminal2);

Expression terminal3 = new TerminalExpression("Tablet");
Expression or = new OrExpression(terminal2, terminal3);

boolean andResult = and.interpret(context);
boolean orResult = or.interpret(context);

System.out.println(data + "\" is " + (andResult ? "valid" : "invalid"));
System.out.println(data + "\" is " + (orResult ? "valid" : "invalid"));
```

문자열에 대한 규칙을 정의하고 data 문자열이 규칙을 만족하는지 검사하는 예제이다.

### 해석자 패턴 고려사항

> 언어나 문법 등을 해석하거나 분석할 때 유용하게 사용할 수 있다. 하지만 문법이 자주 변경되거나 문자열의 길이가 긴 문법을 처리할 때는 적절하지 않다.
> 

<br/>

**해석자 패턴 사용 시 장점**

1. 새로운 문법 규칙을 추가할 때 기존 클래스를 수정하지 않아도 되기 때문에 쉽게 확장할 수 있다.
2. 각 해석자 클래스는 단일 책임 원칙을 따르기 때문에 재사용성이 높아진다.

<br/>

**해석자 패턴 사용 시 단점**

1. 입력 데이터의 크기에 따라 성능이 저하될 수 있다.
2. 유연성과 확장성을 제공하지만 그만큼 코드의 복잡성이 높아지기 때문에 유지보수가 어려울 수 있다.

<br/><br/><br/>