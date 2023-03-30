+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 데이터로 작업하기"
author = "lhj8390"
description = "스프링에서 JDBC, JPA를 이용하는 방법에 대해 설명한다."
date = "2022-06-30"
tags = ["spring", "web", "java", "spring boot"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++

## 개요

---

> 스프링인액션 2장을 읽고 스프링에서 JDBC, JPA를 이용하는 방법에 대해 작성하였다.
> 
> - JDBC template 사용하기
> - SimpleJdbcInsert를 이용해 데이터 가공하기
> - Custom Converter 사용법
> 
> 에 대해 기술하였다.
> 

## JDBC로 작업하기

---

> 스프링의 JDBC 지원은 JDBCTemplate 클래스에 기반을 둔다.
> 
> 
> JDBC Template은 Spring JDBC 접근 방법 중 하나로, 
> 
> 내부적으로 Plain JDBC API를 사용하지만 Plain JDBC의 문제점들을 제거하고 원래의 JDBC에서 지원되지 않는 편리한 기능을 제공하는 Spring에서 제공하는 class이다.
> 
<br/>

### **Plain JDBC API의 문제점**

1. 쿼리를 실행하기 전과 후에 연결부터 resultSet을 닫고 클린업하는 등등 많은 코드를 작성해야한다.
2. 데이터베이스 로직에서 예외 처리 코드를 수행해야 한다.
3. 트랜잭션을 처리해야 한다.
4. 이러한 모든 코드를 반복하는 것으로, 시간이 낭비된다.
<br/>
<br/>

**JDBC Template을 사용하지 않은 예시**

```java
@Override
public Ingredient findById(String id) {
    Connection connection = null;
    PreparedStatement statement = null;
    ResultSet resultSet = null;

    try {
        connection = dataSource.getConnection();
        statement = connection.prepareStatement(
                "select id, name, type from Ingredient where id = ?");
        statement.setString(1, id);
        resultSet = statement.executeQuery();

        Ingredient ingredient = null;
        if (resultSet.next()) {
            ingredient = new Ingredient(resultSet.getString("id"), resultSet.getString("name"), 
                                resultSet.Type.valueOf(resultSet.getString("type")));
        }
        return ingredient;

    } catch (SQLException e) {

    } finally {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {}
        }
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {}
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {}
        }
    }
}
```

이와 같이 데이터베이스 연결 생성, 명령문 생성, 연결과 명령문 및 resultSet을 닫고 클린업하는 코드들로 쿼리 코드가 둘러싸여 있다.<br/>
또 예외처리 시 catch 블록에서 해결될 수 없으므로 현재 메서드를 호출한 상위 코드로 예외 처리를 넘겨야 하는 단점이 존재한다.
<br/>
<br/>

**JDBC Template을 이용한 예시**

```java
private JdbcTemplate jdbc;

@Override
public Ingredient findById(String id) {
    return jdbc.queryForObject(
            "select id, name, type from Ingredient where id=?",
            this::mapRowToIngredient, id);
}

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
    return new Ingredient(rs.getString("id"), rs.getString("name"), 
                            Ingredient.Type.valueOf(rs.getString("type")));
}
```

명령문(statement)나 데이터베이스 연결 객체를 생성, 메서드의 실행이 끝난 후 객체들을 클린업하는 코드가 없는 것을 알 수 있다.<br/>
또한 예외처리 (catch 블록)를 수행하는 코드도 없어 코드가 훨씬 간결해졌다.
<br/>

## JDBC Template 사용하기

---

### **빌드명세에 추가**

```java
dependencies {
    implementation "org.springframework.boot:spring-boot-starter-jdbc"
    runtimeOnly "com.h2database:h2:2.1.212"
}
```

스프링 부트의 JDBC 스타터 의존성을 build.gradle 파일에 추가한다.

또한 H2 내장 데이터베이스를 사용하기 위해 h2database에 대한 의존성도 추가한다. ([https://mvnrepository.com/artifact/com.h2database/h2](https://mvnrepository.com/artifact/com.h2database/h2))
<br/>
<br/>

### **JDBC repository 정의하기**

```java
@Repository
public class JdbcIngredientRepository implements IngredientRepository {

    private JdbcTemplate jdbc;

    @Autowired
    public JdbcIngredientRepository(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }

    @Override
    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type from Ingredient", this::mapRowToIngredient);
    }

    @Override
    public Ingredient findById(String id) {
        return jdbc.queryForObject(
                "select id, name, type from Ingredient where id=?",
                this::mapRowToIngredient, id);
    }

    @Override
    public Ingredient save(Ingredient ingredient) {
        jdbc.update(
            "insert into Ingredient(id, name, type) values(?, ?, ?)",
            ingredient.getId(),
            ingredient.getName(),
            ingredient.getType().toString()
        );
        return ingredient;
    }

    private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
        return new Ingredient(rs.getString("id"), rs.getString("name"), 
                                Ingredient.Type.valueOf(rs.getString("type")));
    }
}
```

- **findAll()** : 객체가 저장된 컬렉션을 반환 - 쿼리로 생성된 ResultSet의 행 개수만큼 호출하고 각각의 객체로 생성하여 List에 저장한다.
    JdbcTemplate의 query() 메서드는 **두개의 인자**를 받는데, 첫번째 인자는 (1)쿼리를 수행하는 SQL, 두번째 인자는 스프링의 (2)RowMapper 인터페이스를 구현한 mapRowToIngredient 메서드이다.

- **findById()** : 하나의 객체만 반환
    JdbcTemplate의 queryForObject() 메서드는 **세개의 인자**를 받는데, 첫번째와 두번째 인자는 query() 메서드와 같고, 세번째 인자는 (3)검색할 행의 id를 전달한다.

- **update()** : 데이터를 추가하거나 변경하는 쿼리에 사용할 수 있다.
    JdbcTemplate의 update() 메서드는 (1)쿼리를 수행하는 SQL, (2)쿼리 매개변수에 지정할 값을 인자로 전달한다.
<br/>
<br/>

**RowMapper 인터페이스를 이용해 구현**

```java
@Override
public Ingredient findById(String id) {
    return jdbc.queryForObject(
        "select id, name, type from Ingredient where id=?",
        new RowMapper<Ingredient>() {
            public Ingredient mapRow(ResultSet rs, int rowNum) throws SQLException {
                return new Ingredient(rs.getString("id"), rs.getString("name"), 
                                        Ingredient.Type.valueOf(rs.getString("type")));
            };
        }, id);
}
```

RowMapper를 구현한 익명 클래스를 이용하여 findById() 메서드를 생성할 수 있다. <br/>
findById() 메서드가 호출될 때마다 익명 클래스 인스턴스가 생성되어 인자로 전달된 후 mapRow() 메서드가 실행되는 방식이다.
<br/>
<br/>

### 엔티티를 삽입한 후 자동 생성된 키를 받을 경우

> 다른 테이블과 외래키를 통해 참조되는 경우 객체에 대한 primary key 값이 필요한데, 상단의 update() 메서드를 사용할 경우 해당 객체의 id 값을 받아올 수 없다.
> 
> 
> 이때, 사용하는 것이 **KeyHolder**이다.
> 

```java
private long saveTacoInfo(Taco taco) {
    taco.setCreatedAt(new Date());
    PreparedStatementCreatorFactory factory = new PreparedStatementCreatorFactory(
            "insert into Taco (name, createdAt) values (?, ?)",
            Types.VARCHAR, Types.TIMESTAMP
    );
    factory.setReturnGeneratedKeys(true);

    PreparedStatementCreator psc =
        factory.newPreparedStatementCreator(
            Arrays.asList(
                taco.getName(),
                new Timestamp(taco.getCreatedAt().getTime())));

    KeyHolder keyHolder = new GeneratedKeyHolder();
    jdbc.update(psc, keyHolder);
    return keyHolder.getKey().longValue();
}
```

여기에서 update() 메서드는 (1) PreparedStatementCreator 객체와 (2) KeyHolder 객체를 인자로 받는다.

PreparedStatementCreator 객체를 만들기 위해서는 

1. 우선 실행할 SQL 명령어와 각 쿼리 매개변수의 타입을 인자로 전달하는 **PreparedStatementCreatorFactory**를 만들고,
2. **newPreparedStatementCreator**()를 호출하여 쿼리 매개변수 값을 인자로 전달한다.

<aside>
setReturnGeneratedKeys(true) 으로 설정하여 자동 생성 키를 반환한다고 명시해주었다.<br/>

</aside>

update() 메서드의 실행이 끝나면 keyHolder.getKey() 를 통해 ID 값을 반환할 수 있다.

**예시 2.**

```java
jdbc.update(connection -> {
        PreparedStatement ps = connection
                .prepareStatement("insert into Taco (name, createdAt) values (?, ?)");
        ps.setString(1, taco.getName());
        ps.setTimestamp(2, new Timestamp(taco.getCreatedAt().getTime()));
        return ps;
    }, keyHolder);
```

이와 같이 사용할 수도 있다.

## SimpleJdbcInsert를 사용하기

---

> 복잡한 PreparedStatementCreator 대신 SimpleJdbcInsert를 이용하면 손쉽게 id 값을 받아올 수 있다.<br/>
> SimpleJdbcInsert는 데이터를 더 쉽게 테이블에 추가하기 위해 JdbcTemplate를 래핑한 객체이다.
> 
<br/>

### SimpleJdbcInsert 인스턴스 초기화

```java
@Repository
public class JdbcOrderRepository implements OrderRepository {

    private SimpleJdbcInsert orderInserter;
    private ObjectMapper objectMapper;

    @Autowired
    public JdbcOrderRepository(JdbcTemplate jdbc) {
        this.orderInserter = new SimpleJdbcInsert(jdbc).withTableName("Taco_Order")
                                .usingGeneratedKeyColumns("id");

        this.objectMapper = new ObjectMapper();
    }
}
```

JdbcTemplate를 이용하여 SimpleJdbcInsert 인스턴스를 생성하는 코드이다.<br/>
SimpleJdbcInsert 객체에는 테이블 이름이 Taco_Order이고, id는 데이터베이스가 생성해주는 값으로 사용한다고 명시해주었다.

```java
private long saveOrderDetails(Order order) {
    @SuppressWarnings("unchecked")
    Map<String, Object> values = objectMapper.convertValue(order, Map.class);
    values.put("placedAt", order.getPlacedAt());
    long orderId = orderInserter.executeAndReturnKey(values).longValue();

    return orderId;
}
```

SimpleJdbcInsert는 데이터를 추가하는 execute() 메서드와 executeAndReturnKey() 메서드가 있다. -  *두 메서드는 Map<String, Object>의 타입의 인자를 받는다.*

Order 객체를 Map으로 변환하기 위해 objectMapper의 convertValue() 메소드를 사용하였다.<br/>
executeAndReturnKey() 메서드를 이용하면 데이터가 테이블에 저장된 후 데이터베이스에서 생성된 ID가 Number 객체로 반환된다.

## Custom Converter 사용

---

> ID 값을 사용하여 특정 객체로 변환이 필요할 경우가 있는데, 이때 Converter 클래스를 사용한다.<br/>
> Converter를 사용하면 가독성을 높이는 효과도 있다.
> 

```java
@Component
public class IngredientByIdConverter implements Converter<String, Ingredient> {
    private final IngredientRepository ingredientRepo;

    @Autowired
    public IngredientByIdConverter(IngredientRepository ingredientRepo) {
        this.ingredientRepo = ingredientRepo;
    }

    @Override
    public Ingredient convert(String id) {
        return ingredientRepo.findById(id);
    }
}
```

**Converter<S,T>** 인터페이스를 이용하면 S타입을 T타입으로 변환할 수 있다.<br/>
상태 정보가 없기 때문에 얼마든지 빈으로 등록해서 사용해도 상관없다.