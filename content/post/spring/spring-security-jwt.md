+++
aliases = ["posts", "articles", "blog", "showcase", "docs"]
title = "Spring Security + JWT 인증"
author = "lhj8390"
description = "스프링 시큐리티에서 JWT 인증을 수행하는 방법에 대해 정리했다."
date = "2022-10-29"
tags = ["spring", "web", "java", "spring boot", "spring security"]
categories = ["Framework"]
subcategories = ["Spring"]
+++

> Spring Security는 커스터마이징이 가능한 인증 및 액세스 제어 프레임워크이다. 스프링 기반 애플리케이션을 보호하기 위한 표준으로, 쉽게 확장이 가능하다는 장점이 있다.
> 

Spring boot gradle 프로젝트에서 Spring Security 5.7.x 를 활용해서 JWT로 인증을 구현하는 방법에 대해 작성해보았다. 

## 기본 용어 설명

---

Spring Security에서 사용하는 용어는 다음과 같다.

- **Authentication :** 제공된 자격 정보를 기반으로 사용자의 ID를 확인하는 프로세스이다.
- **Authorization :** 사용자가 인증된 후 해당 사용자가 액션을 수행할 수 있는 권한을 가지고 있는지 확인한다.
- **Principle :** 현재 인증된 사용자이다.
- **Granted authority :** 인증된 사용자의 권한을 의미한다.
- **Role :** 인증된 사용자의 권한 그룹이다.

## 스프링 보안 아키텍처

---

Spring Security 인증은 다음 그림과 같은 플로우로 이루어져 있다.

{{<figure src="/images/spring-security-jwt/1.png" class="large" caption="https://backendstory.com/spring-security-authentication-architecture-explained-in-depth/">}}


1. **Spring Security Filter Chain**
    
    Spring Security 프레임워크를 추가하면 필터 체인이 자동으로 등록되고 필터 체인이 가지고 있는 여러가지 보안 필터들을 하나씩 거치게 된다. 보통 아이디, 패스워드를 통한 인증을 수행할 경우 “UsernamePasswordAuthenticationFilter” 이름의 필터로 인증을 수행하게 된다.
    
2. **Authentication Manager**
    
    Authentication Manager의 구현체인 ProviderManager는 Authentication Provider 목록 중에서 전달받은 인증 정보를 지원하는지 체크하고, 인증을 수행할 수 있는 Authentication Provider에게 위임하여 인증 과정을 수행하라고 요청한다.
    
3. **Authentication Provider**
    
    기본적으로 DaoAuthenticationProvider 객체가 인증을 수행한다. DaoAuthenticationProvider는 UserDetailsService 객체에게 사용자 아이디를 넘겨주고 UserDetails 객체를 전달받는다. 전달받은 객체에서 PasswordEncoder를 사용하여 암호가 유효한지 확인한다.
    
4. **UserDetailsService**
    
    DB에서 사용자 정보를 읽고 UserDetails를 반환하는 인터페이스이다.
    

**Authentication Provider**를 통해 인증을 성공하면 “UsernamePasswordAuthenticationToken” 타입의 Authentication 객체가 반환된다. ( <span class="gray">*Authentication 객체의 Principal은 UserDetails이다.* </span>) Authentication 객체는 보안 필터에 의해 SecurityContextHolder에 설정되면서 인증이 완료된다.

## JWT를 사용하여 Spring Security 구현

---

### 1. Gradle 프로젝트 설정

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'io.jsonwebtoken:jjwt:0.9.1'
}
```
Spring Security와 함께 JWT dependencies도 함께 추가해준다.

### 2.Security 설정

Spring Security 5.7.x 버전부터는 WebSecurityConfigurerAdapter의 확장을 지원하지 않는다. 

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {
  
  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      // CSRF 설정 비활성화
      .csrf().disable()

      // /api/auth/** 요청을 제외한 나머지 요청은 admin만 접근 가능
      .authorizeRequests()
        .antMatchers("/api/auth/**").permitAll()
        .anyRequest().hasRole("ADMIN")
      .and()

        // 토큰 기반 인증이기 때문에 세션 설정을 Stateless 로 설정
        .sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
      .and()

        // 폼 로그인 비활성화
        .formLogin().disable()

      // JWT 토큰 필터 추가
      .addFilterBefore(jwtAuthenticationFilter(), 
         UsernamePasswordAuthenticationFilter.class)

      // 에러 핸들링
      .exceptionHandling().authenticationEntryPoint(
         (request, response, authException) ->
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized")
         )
      .accessDeniedHandler((request, response, accessDeniedException) ->
         response.sendError(HttpServletResponse.SC_FORBIDDEN, "Forbidden")
      );
    return http.build();
  }
}
```

기본적인 WebSecurity 설정 파일이다. 우리는 REST API를 통해 JWT 토큰 인증을 수행할 예정이기 때문에 다음과 같은 설정을 한다.

- authorizeRequests : 엔드포인트에 대한 권한 설정
- sessionCreationPolicy : 세션 관리를 상태 비저장 상태로 설정
- jwtAuthenticationFilter : JWT 토큰 필터 추가
- authenticationEntryPoint : 인증과정 실패를 처리해주는 로직을 구현한다. 401 Unauthorized 응답을 반환한다.
- accessDeniedHandler : 권한이 없는 사용자가 접근할 경우에 대한 로직을 구현한다. 403 Forbidden 응답을 반환한다.

### 3. JWT 생성 및 인증 클래스 구현

JWT 토큰을 관리해주는 클래스를 생성해준다.

```java
@Slf4j
@Component
public class JwtTokenProvider {
  @Value(value = "${app.jwtSecret}")
  private String jwtSecret;

  @Value(value = "${app.jwtExpiration}")
  private int jwtExpiration;

  // Authentication 객체를 받아와 JWT 토큰을 생성한다.
  public String generateToken(Authentication authentication) {
    String email = (String) authentication.getPrincipal();

    Date expiryDate = new Date(new Date().getTime() + jwtExpiration);

    return Jwts.builder()
        .setSubject(email)
        .setIssuedAt(new Date())
        .setExpiration(expiryDate)
        .signWith(SignatureAlgorithm.HS512, jwtSecret)
        .compact();
  }

  // JWT 토큰을 통해 email 값을 받는다.
  public String getEmailFromJWT(String token) {
    Claims claims = Jwts.parser()
          .setSigningKey(jwtSecret)
          .parseClaimsJws(token)
          .getBody();

    return claims.getSubject();
  }

  // JWT 토큰이 유효한지 검증한다.
  public boolean validateToken(String authToken) {
    try {
      Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(authToken);
      return true;
    } catch (SecurityException ex) {
      log.error("Invalid JWT signature");
    } catch (MalformedJwtException ex) {
      log.error("Invalid JWT token");
    } catch (ExpiredJwtException ex) {
      log.error("Expired JWT token");
    } catch (UnsupportedJwtException ex) {
      log.error("Unsupported JWT token");
    } catch (IllegalArgumentException ex) {
      log.error("JWT claims string is empty");
    }
    return false;
  }
}
```

다음과 같은 메서드를 구현해준다.
- generateToken : Authentication 객체를 받아와 JWT 토큰을 생성한다.
- getEmailFromJWT : JWT 토큰을 통해 email 값을 받는다.
- validateToken : JWT 토큰이 유효한지 검증한다.

### 4. CustomUserDetailsService 생성

UserDetailsService 인터페이스를 구현한 CustomUserDetailsService 클래스를 생성한다.

```java
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {
  private final UserRepository userRepository;

  @Override
  @Transactional
  public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {

    User user = userRepository.findByEmail(email)
      .orElseThrow(() -> new UsernameNotFoundException("해당 이름의 사용자가 없습니다."));

    List<GrantedAuthority> authorities = user.getRoles().stream().map(role ->
        new SimpleGrantedAuthority(role.getName().name())
    ).collect(Collectors.toList());

    return new org.springframework.security.core.userdetails.User
        (user.getUsername(), user.getPassword(), authorities);
  }
}
```

CustomUserDetailsService에서는 사용자의 데이터를 DB에서 받아오도록 한다. loadUserByUsername 메소드를 Override하여 <span class="red">**DaoAuthenticationProvider는 여기서 반환하는 UserDetails와 Authentication의 패스워드가 유효한지 검증할 수 있다.** </span>

### 5. 로그인 API 구현

실제로 로그인을 시도하는 Service 코드이다.

```java
private final AuthenticationManager authenticationManager;

public AuthResponseDTO login(LoginRequestDTO dto) {

  Authentication authentication = new UsernamePasswordAuthenticationToken(dto.getEmail(), dto.getPassword());
  authenticationManager.authenticate(authentication);

  return AuthResponseDTO.builder()
      .token(tokenProvider.generateToken(authentication))
      .build();
}
```

전달받은 DTO 객체를 이용하여 UsernamePasswordAuthenticationToken 타입의 Authentication 객체를 생성해준다. 인증에 성공할 경우 JWT token 값을 응답 데이터에 넣어준다.

로그인 API 기능이 동작하기 위해서는 AuthenticationManager 에 대한 액세스 권한이 필요하다.

```java
@Bean
public AuthenticationManager authenticationManager
    (AuthenticationConfiguration authConfiguration) throws Exception {

  return authConfiguration.getAuthenticationManager();
}
```

기본적으로는 AuthenticationManager에 액세스할 수 없으며 **구성 클래스에서 빈으로 명시적으로 노출해야 한다.**

AuthenticationManager가 인증을 수행할 때 PasswordEncoder를 이용해 전달받은 패스워드가 일치한지 여부를 체크하는데, 이 PasswordEncoder도 구성 클래스에서 빈으로 노출해야 한다.

```java
@Bean
PasswordEncoder passwordEncoder() {
    String idForEncode = "bcrypt";

    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put(idForEncode, new BCryptPasswordEncoder());

    return new DelegatingPasswordEncoder(idForEncode, encoders);
}
```

Spring Security 5 에서는 **DelegatingPasswordEncoder**가 기본 암호화 방식으로 사용된다. DelegatingPasswordEncoder을 사용할 경우 Password 앞에 {id}로 PasswordEncoder 유형이 정의된다. BCrypt 암호화 방식을 사용하기 위해 위와 같이 작성한다.

### 6. 토큰 검증

로그인 API가 정상적으로 수행되었다면 JWT token 값을 반환받게 되고 이를 Authorization 헤더에 넣을 수 있다.

```java
@AllArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

  private JwtTokenProvider tokenProvider;
  private CustomUserDetailsService userDetailsService;

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) 
      throws ServletException, IOException {

    String token = getTokenFromRequest(request);

    // 인증 토큰이 존재할 경우 토큰을 검증한다.
    if (StringUtils.hasText(token) && tokenProvider.validateToken(token)) {
      String email = tokenProvider.getEmailFromJWT(token);
      UserDetails userDetails = userDetailsService.loadUserByUsername(email);

      Authentication authentication = new UsernamePasswordAuthenticationToken
           (userDetails, null, userDetails.getAuthorities());
      // 토큰이 유효할 경우 SecurityContextHolder에 인증 값을 넣는다.
      SecurityContextHolder.getContext().setAuthentication(authentication);
    }

    filterChain.doFilter(request, response);
  }
  
  // Authorization 헤더에 인증 값을 받아온다. 
  private String getTokenFromRequest(HttpServletRequest request) {
    String bearerToken = request.getHeader("Authorization");
    if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
      return bearerToken.substring(7);
    }
    return null;
  }
}
```

Http header에 토큰이 있을 경우 유효한 토큰인지 검증하는 과정을 수행한다. OncePerRequestFilters는 http 요청 당 한번만 실행된다. 

이 필터를 Spring Security 구성 클래스에 추가해준다. 

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
  // .. 나머지 생략
  .addFilterBefore(jwtAuthenticationFilter(), 
      UsernamePasswordAuthenticationFilter.class)
}
```

addFilterBefore() 를 통해 UsernamePasswordAuthenticationFilter보다 먼저 실행되게 하여 Authorization 헤더에 유효한 토큰이 있을 경우 바로 인증을 성공시키게 한다.