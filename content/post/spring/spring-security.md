+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "[스프링인액션] 스프링 시큐리티"
author = "lhj8390"
description = "스프링에서 스프링 시큐리티를 이용하는 방법에 대해 설명한다."
date = "2022-07-14"
tags = ["spring", "web", "java", "spring boot", "spring security"]
categories = ["Web"]
subcategories = ["Spring"]
series = ["스프링인액션"]
thumbnail= "images/spring-basic.png"
+++
## 개요

---

> 스프링 시큐리티를 이용하여 스프링 애플리케이션에 보안 기능을 구현한다.
> - 스프링 시큐리티 구성하기
> - 여러 방법을 사용하여 사용자 스토어 구성해보기
> - password encoder 적용하기
> - 스프링 시큐리티 보안 구성하기
> - 사용자 정보 취득하기
> 
> 에 대해 기술해보았다.
> 

## 스프링 시큐리티 구성하기

---

### 빌드 명세

```groovy
implementation 'org.springframework.boot:spring-boot-starter-security'
testImplementation 'org.springframework.security:spring-security-test'
```

스프링 부트 보안 스타터 의존성을 추가하였다.

### 스프링 시큐리티 구성 클래스

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests()
        .antMatchers("/design", "/orders")
        .access("hasRole('ROLE_USER')")
        .antMatchers("/", "/**").access("permitAll")
        .and()
        .httpBasic();
    }

    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("user1")
            .password("{noop}password1")
            .authorities("ROLE_USER")
            .and()
            .withUser("user2")
            .password("{noop}password2")
            .authorities("ROLE_USER");
    }
}
```

사용자의 HTTP 요청 경로에 대해 접근 제한과 같은 보안 처리를 원하는 대로 처리할 수 있게 도와준다.<br/>
SecurityConfig 클래스는 보안 구성 클래스인 WebSecurityConfigurerAdaper의 서브 클래스이다.

- **configure(HttpSecurity)** : HTTP 보안을 구성하는 메서드
- **configure(AuthenticationManagerBuilder)** : 사용자 인증 정보를 구성하는 메서드

## 사용자 스토어 구성

---

> 보안을 구성하기에 사용자를 처리할 수 있도록 사용자 정보를 유지/관리하는 사용자 스토어를 구성할 수 있다.<br/>
> 스프링 시큐리티에서는 4가지의 사용자 스토어 구성 방법을 제공한다.
> - 인메모리 사용자 스토어
> - JDBC 기반 사용자 스토어
> - LDAP 기반 사용자 스토어
> - 커스텀 사용자 명세 서비스
> 
> 사용자 스토어는 SecurityConfig 클래스에서 **configure(AuthenticationManagerBuilder)** 메서드를 오버라이딩하여 구성할 수 있다.
> 

### 인메모리 사용자 스토어

user1, user2라는 사용자를 인메모리 사용자 스토어에 구성하는 예제이다.

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        .withUser("user1")
        .password("{noop}password1")
        .authorities("ROLE_USER")
        .and()
        .withUser("user2")
        .password("{noop}password2")
        .authorities("ROLE_USER");
}
```

inMemoryAuthentication() 메서드를 사용하여 보안 구성 자체에 사용자 정보를 직접 지정할 수 있다.

- withUser() : 사용자의 구성 시작
- passwrd() : 비밀번호 설정
- authorities() : 권한 설정 - <span class="gray it">roles(”user”) 로도 사용할 수 있다.</span>
- and() : 연속해서 withUser() 호출 가능

### JDBC 기반의 사용자 스토어

관계형 데이터베이스에 유지되는 사용자 정보를 스토어에 구성하는 예제이다.

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .jdbcAuthentication()
        .dataSource(dataSource)
        .usersByUsernameQuery("select username, password, enabled from users where username=?")
        .authoritiesByUsernameQuery("select username, authority from authorities where username=?");
}
```

jdbcAuthentication() 메서드 호출과 함께 데이터베이스에 액세스할 수 있도록 DataSource를 설정하였다.<br/>
스프링 시큐리티는 사전 지정된 데이터베이스 테이블이 존재하는데, 우리가 사용하고 싶은 데이터베이스를 커스터마이징할 수도 있다.

<span class="red">→ usersByUsernameQuery() 와 authoritiesByUsernameQuery() 메서드를 사용하여 사용자 정보와 권한 쿼리를 대체하였다.</span>
<br/><br/>

<aside>
<span class="red">스프링 시큐리티의 SQL 쿼리를 대체할 때의 주의사항!</span><br/>
1.  스프링 시큐리티의 기본 데이터베이스 테이블 이름이 달라도 되지만 테이블이 갖는 데이터 타입과 길이는 일치해야 한다.
2.  where 절에서 사용되는 매개변수는 하나이고, 그 매개변수는 username이어야 한다.
3. 사용자 정보 인증 쿼리에서는 username, password, enabled 열의 값을 반환하여야 한다.
4. 사용자 권한 쿼리에서는 username과 authority을 포함하는 0 또는 다수의 행을 반환할 수 있다.
5. 그룹 권한 쿼리에서는 group id, group_name, authority을 포함하는 0 또는 다수의 행을 반환할 수 있다.<br/>
</aside>

### LDAP 기반의 사용자 스토어

LDAP 인증의 간단한 구성방법에 대한 예제이다.

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .ldapAuthentication()
        .userSearchBase("ou=people")
        .userSearchFilter("(uid={0})")
        .groupSearchBase("ou=groups")
        .groupSearchFilter("member={0}")
        .contextSource()
        .root("dc=tacocloud,dc=com")
        .ldif("classpath:users.ldif")
        .and()
        .passwordCompare()
        .passwordEncoder(new BCryptPasswordEncoder())
        .passwordAttribute("userPasscode");
}
```

**userSearchFilter()** 와 **groupSearchFilter()** 메서드 : LDAP 기본 쿼리의 필터를 제공하기 위해 사용한다. 사용자와 그룹을 검색할 수 있다.<br/>
**userSearchBase()** 와 **groupSearchBase()** 메서드 : 사용자와 그룹을 찾기 위한 기준점 쿼리를 지정할 수 있다. - <span class="gray it">루트로부터 검색하지 않고 사용자는 people 구성 단위(OU), 그룹은 groups 구성 단위부터 검색이 시작된다.</span>

### 사용자 인증 커스터마이징

사용자 데이터를 JPA를 통해 처리하기 위해서는 사용자 도메인 객체와 퍼시스턴스를 직접 정의해야 한다.<br/>
사용자 데이터를 구현하는 User 객체와 user 데이터를 넣는 repository를 만든 다음,  스프링 시큐리티 구성 내부에 UserDetailsService 구현체를 넘겨줘서 구현한다.

<br/>

**사용자 도메인 객체 정의**

```java
@Entity
@Data
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@RequiredArgsConstructor
public class User implements UserDetails {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private final String username;
    private final String password;
    private final String fullname;
    private final String street;
    private final String city;
    private final String state;
    private final String zip;
    private final String phoneNumber;

    @Override
    public Collection<? extends
            GrantedAuthority> getAuthorities() {
        return Arrays.asList(new
                SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

스프링 시큐리티에서 제공하는 UserDetails 인터페이스를 구현하였다. 이를 통해 User 클래스는 기본 사용자 정보를 프레임워크에게 제공할 수 있다.

- getAuthorities() : 사용자에게 부여된 권한을 저장한 컬렉션을 반환하는 메서드이다.
- isAccountNonExpired(), isAccountNonLocked(), isCredentialsNonExpired(), isEnabled() : 사용자 계정의 활성화, 비활성화 여부를 나타내는 메서드이다.
<br/><br/>

**사용자 명세 서비스 생성하기**

```java
@Service
public class UserRepositoryUserDetailsService implements UserDetailsService {
    private UserRepository userRepo;

    @Autowired
    public UserRepositoryUserDetailsService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepo.findByUsername(username);
        if (user != null) {
            return user;
        }
        throw new UsernameNotFoundException("User '" + username + "' not found");
    }
}
```

스프링 시큐리티에서 제공하는 UserDetailsService 인터페이스를 구현한 클래스이다.

- loadUserByUsername() : findByUsername() 에서 null을 반환하면 UsernameNotFoundException을 발생시키며 null이 아닌 경우 UserDetail의 구현체인 User 값이 반환된다. - *UserDetails 인터페이스에 User Entity 삽입*
<br/><br/>

**스프링 시큐리티 구성하기**

```java
@Autowired
private UserDetailsService userDetailsService;

@Bean
public PasswordEncoder encoder() {
    return new BCryptPasswordEncoder();
}

@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception{
    auth.userDetailsService(userDetailsService).passwordEncoder(encoder());
}
```

- userDetailsService() : SecurityConfig로 자동 주입된 UserDetailService 인스턴스를 인자로 전달하여 호출한다.
- passwordEncoder() : 데이터베이스에 비밀번호가 암호화되어 저장될 수 있도록 인코더를 구성한다.

## 암호화된 비밀번호 사용하기

---

> 스프링 시큐리티 5 버전부터는 의무적으로 PasswordEncoder를 이용해서 비밀번호를 암호화해야 한다.
> 
> 
> 비밀번호를 데이터베이스에 저장할 때와 사용자가 입력한 비밀번호는 모두 같은 암호화 알고리즘을 사용해 인증을 수행한다.
> 

### PasswordEncoder 인터페이스 정의

```java
public interface PasswordEncoder {

	/**
	 * rawPassword를 인코딩합니다. 일반적으로 좋은 인코딩 알고리즘은 
	 * 무작위로 생성된 8바이트 이상의 솔트와 결합된 SHA-1 이상의 해시를 적용합니다.
	 */
	String encode(CharSequence rawPassword);

	/**
	 * 저장소에서 가져온 인코딩된 암호가 인코딩된 후 제출된 원시 암호와 일치하는지 확인합니다. 
	 * 암호가 일치하면 true를 반환하고 일치하지 않으면 false를 반환합니다. 
   	 * 저장된 암호 자체는 디코딩되지 않습니다.
	 * @param rawPassword - 인코딩 및 비교할 원시 암호
	 * @param encodedPassword - 스토리지에서 인코딩된 암호를 추출하여 비교
	 * @return true - 인코딩 후 원시 암호가 스토리지의 인코딩된 암호와 일치할 경우
	 */
	boolean matches(CharSequence rawPassword, String encodedPassword);

}
```

- encode : 패스워드를 암호화할 때 사용한다.
- matches : 사용자에게 입력받은 패스워드를 비교하고자 할 때 사용한다.

### JDBC 기반 암호화

비밀번호를 암호화할 때는 아래와 같이 paswrdEncoder() 메소드를 호출하여 비밀번호 인코더를 지정한다.

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .jdbcAuthentication()
        .dataSource(dataSource)
        .usersByUsernameQuery("select username, password, enabled from users where username=?")
        .authoritiesByUsernameQuery("select username, authority from authorities where username=?")
        .passwordEncoder(new NoEncodingPasswordEncoder());
}
```

passwordEncoder() 메서드는 스프링 시큐리티의 PasswordEncoder 인터페이스를 구현한 모든 객체를 인자로 받을 수 있다.

- BCryptPasswordEncoder : bcrypt를 해싱 암호화한다.
- NoOpPaswordEncoder : 암호화하지 않는다.
- Pbkdf2PasswordEncoder : PBKDF2를 암호화한다.
- SCryptPasswordEncoder : scrypt를 해싱 암호화한다.
- StandardPasswordEncoder : SHA-256을 해싱 암호화한다.

### LDAP 비밀번호 비교 구성

LDAP의 기본 인증 전략은 사용자가 직접 LDAP 서버에서 인증받도록 하지만, 하단처럼 비밀번호를 비교하는 방법도 존재한다. 

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .ldapAuthentication()
        .userSearchBase("ou=people")
        .userSearchFilter("(uid={0})")
        .groupSearchBase("ou=groups")
        .groupSearchFilter("member={0}")
        .contextSource()
        .root("dc=tacocloud,dc=com")
        .ldif("classpath:users.ldif")
        .and()
        .passwordCompare()
        .passwordEncoder(new BCryptPasswordEncoder())
        .passwordAttribute("userPasscode");
}
```

입력된 비밀번호를 LDAP 디렉터리에 전송하고, 이 비밀번호를 사용자의 비밀번호 속성 값과 비교하도록 LDAP 서버에 요청한다.

- **passwordCompare**() : 로그인 폼에 입력된 비밀번호가 사용자의 LDAP 서버에 있는 userPassword 값과 비교하도록 요청하는 메서드이다.
- **passwordAttribute**() : 비밀번호가 userPassword 속성이 아닌 다른 속성의 이름을 사용하고 있다면 해당 메서드를 통해 속성 값을 명시해준다. - <span class="gray it">여기서는 속성 값이 userPasscode</span>
- **passwordEncoder**() : 비밀번호가 LDAP 서버에 전달될 때 해당 인코더를 통해 암호화된다는 것을 의미한다.

## 스프링 시큐리티 보안 구성

---

> configure(HttpSecurity) 메서드를 통해 다음과 같이 웹 수준에서 보안을 처리하는 방법을 구성할 수 있다.
> 
> - HTTP 요청 처리를 허용하기 전에 충족되어야 할 보안 조건을 구성한다.
> - 커스텀 로그인 페이지를 구성한다.
> - 사용자가 로그아웃을 할 수 있도록 한다.
> - CSRF 공격으로부터 보호하도록 구성한다.

### 특정 페이지에 한해 인증 수행하기

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/design", "/orders")
        .access("hasRole('ROLE_USER')")
        .antMatchers("/", "/**").access("permitAll");
}
```

authorizeRequests() 메서드를 통해 /design 과 /order 페이지의 요청은 ROLE_USER의 권한을 갖는 사용자만 접근 가능하도록 허용하고 이외의 요청은 모든 사용자에게 허용하였다.
<br/><br/>

**요청 경로가 보안 처리되는 방법을 정의하는 메소드**

| 메서드 | 설명 |
| --- | --- |
| access(String) | 인자로 전달된 SpEL 표현식이 true 이면 접근을 허용한다. |
| anonymous() | 익명의 사용자에게 접근을 허용한다. |
| authenticated() | 익명이 아닌 사용자로 인증된 경우 접근을 허용한다. |
| denyAll() | 무조건 접근을 거부한다. |
| fullyAuthenticated() | 익명이 아니거나 remember-me가 아닌 사용자로 인증되면 접근을 허용한다. |
| hasAnyAuthority(String…) | 지정된 권한 중 어떤 것이라도 사용자가 가지고 있으면 접근을 허용한다. |
| hasAnyRole(String…) | 지정된 역할 중 어느 하나라도 사용자가 가지고 있으면 접근을 허용한다. |
| hasAuthority(String) | 지정된 권한을 사용자가 가지고 있으면 접근을 허용한다. |
| hasIpAddress(String) | 지정된 IP 주소로부터 요청이 오면 접근을 허용한다. |
| hasRole(String) | 지정된 역할을 사용자가 가지고 있으면 접근을 허용한다. |
| not() | 다른 접근 메서드들의 효력을 무효화한다. |
| permitAll() | 무조건 접근을 허용한다. |
| rememberMe() | 이전 로그인 정보를 쿠키나 데이터베이스로 저장한 후 일정 기간내에 재 접근하여 자동 로그인 된 사용자의 접근을 허용한다. |


<br/>

### 로그인 페이지 경로 지정하기

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/design", "/orders")
        .access("hasRole('ROLE_USER')")
        .antMatchers("/", "/**").access("permitAll")
        .and()
        .formLogin()
        .loginPage("/login")
        .loginProcessingUrl("/authenticate")
        .defaultSuccessUrl("/design")
        .usernameParameter("user")
        .passwordParameter("pwd");
}
```

기본 로그인 페이지를 교체하려면 스프링 시큐리티 구성을 통해 커스텀 로그인 페이지 경로를 알려주어야 한다.<br/>
formLogin() 메서드를 통해 로그인 폼을 구성하겠다고 명시하고, loginPage() 메서드를 통해 로그인 페이지 경로를 지정할 수 있다.<br/>
스프링 시큐리티가 인증이 필요하다고 판단될 경우 loginPage() 메서드에 적용한 경로로 연결해준다.

- **loginProcessingUrl**() : 로그인을 처리하는 경로를 지정할 때 사용한다.
- **defaultSuccessUrl**() : 로그인이 성공적으로 처리되었을 경우 리다이렉트할 페이지 경로를 지정할 때 사용한다. <span class="gray it">- 로그인 페이지에 다이렉트로 접근했을 경우</span> 
 ※ 스프링 시큐리티는 기본적으로 사용자가 머물던 페이지로 이동하는데, 강제적으로 페이지 경로를 지정할 경우에는 **defaultSuccessUrl(”페이지 경로”, true)** 로 지정해준다.
- **usrenameParameter**() : 로그인 폼에서 기본 사용자 이름 필드 이름은 “username”이다. 이 필드 이름을 변경할 경우 사용한다.
- **passwordParameter**() : 로그인 폼에서 기본 비밀번호 필드 이름은 “password”이다. 이 필드 이름을 변경할 경우 사용한다.
<br/>

### 로그아웃 페이지 경로 지정하기

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        //... 이하 생략
        .and()
        .logout()
        .logoutUrl("/logout")
        .logoutSuccessUrl("/")
        .invalidateHttpSession(true)
        .deleteCookies("JSESSIONID")
}
```

로그아웃 설정은 logout() 메서드를 호출하고 logoutUrl() 메서드를 통해 logout을 요청하기 위한 url을 설정할 수 있다. <span class="gray it">- login의 loginProcessingUrl() 메서드와 비슷</span>

- **logoutSuccessUrl**() : 로그아웃 성공 시 리다이렉트할 페이지 경로를 지정할 때 사용한다.
- **invalidateHttpSession**() : 로그아웃이 될 때 세션을 무효화할 것인지 지정할 때 사용한다. 기본 값은 true이다.
- **deleteCookies**() : 로그아웃 될 때 삭제할 쿠키 목록을 지정할 때 사용한다.

### CSRF 설정하기

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        //... 이하 생략
        .and()
        .csrf();
}
```

```html
<form method="POST" th:action="@{/login}" id="loginForm">
```

스프링 시큐리티에는 내장된 CSRF 방어 기능이 있는데, 스프링에 내장된 Thymeleaf 등을 사용한다면 숨김 필드 설정 필요 없이 csrf 필드가 자동으로 생성된다.<br/>
폼 전송을 수행할 때 자동으로 생성된 csrf 필드를 포함시킬 경우 `th:action` 속성을 지정하면 된다.
<br/>
<br/>

**csrf 비활성화하기**

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        //... 이하 생략
        .and()
        .csrf()
        .disable()
}
```

단, REST API로 실행되는 애플리케이션의 경우에는 csrf 를 disable 처리해야 한다.

## 사용자 정보 취득

---

> 인증된 사용자가 누구인지 사용자 정보를 취득하기 위해서는 4가지 방법이 존재한다.
> 
> - Principal 객체를 컨트롤러 메서드에 주입한다.
> - Authentication 객체를 컨트롤러 메서드에 주입한다.
> - SecurityContextHolder를 사용하여 보안 컨텍스트를 얻는다.
> - @AuthenticationPrincipal 어노테이션을 메서드에 지정한다.

### Principal 객체 이용

```java
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Principal principal) {
        return principal.getName();
    }
}
```

Principal 객체를 컨트롤러의 인자로 받아 사용자 정보를 받아오는 코드이다.

### Authentication 객체 이용

```java
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Authentication authentication) {

        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        System.out.println("User has authorities: " + userDetails.getAuthorities());

        return authentication.getName();
    }
}
```

Authentication 객체를 컨트롤러의 인자로 받아 사용자 정보를 받아올 수 있다. <br/>
Authentication 객체의 getPrincipal() 메서드는 Object 타입을 반환하는데 UserDetails 타입으로 변환하여 원하는 데이터를 받아올 수도 있다.

### SecurityContextHolder 를 사용

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
String currentPrincipalName = authentication.getName();
```

보안 컨텍스트로부터 Authentication 객체를 얻은 후 Principal 객체를 받아와 사용자 정보를 받아올 수 있다. <br/>
어플리케이션의 어디든 사용할 수는 있지만 코드를 검증하는 데에 있어서 안 좋은 요소이다.

### @AuthenticationPrincipal 어노테이션 사용

```java
@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(@AuthenticationPrincipal User user) {
        return user.getUsername();
    }
}
```

@AuthenticationPrincipal은 타입 변환이 필요 없고 Authentication과 동일하게 보안 특정 코드만 갖는다.<br/>
메서드에 어노테이션만 지정하면 되어서 편리하게 UserDetails 구현체를 받아올 수 있다.