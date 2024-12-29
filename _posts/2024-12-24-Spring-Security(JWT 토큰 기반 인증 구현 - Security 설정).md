---
title: JWT 토큰 기반 인증 구현 - 1
description: 초기 설정, 엔티티 생성, 시큐리티 설정 - config, filter
author: JoJunHo
date: 2024-12-24 11:33:00 +0800
categories: [ java, SpringBoot, SpringSecurity, JWT ]
tags: [ java, SpringBoot, SpringSecurity, JWT ]
pin: false
---

> ### SpringSecurity
> [SpringSecurity - 개념](https://whwnsgh0258.github.io/posts/Spring-Security/)  
> [SpringSecurity - 구조 및 처리 흐름](https://whwnsgh0258.github.io/posts/Spring-Security(%EA%B5%AC%EC%A1%B0%EC%99%80-%EC%B2%98%EB%A6%AC-%ED%9D%90%EB%A6%84)/)  
> [SpringSecurity - FormLogin](https://whwnsgh0258.github.io/posts/Spring-Security(FormLogin-%EC%9D%B8%EC%A6%9D-%EB%B0%8F-%EA%B6%8C%ED%95%9C-%EB%B6%80%EC%97%AC)/)  
> [SpringSecurity - JWT](https://whwnsgh0258.github.io/posts/Spring-Security(JWT-%ED%86%A0%ED%81%B0-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EA%B5%AC%EC%84%B1%EC%9A%94%EC%86%8C)/)

안녕하세요. 백엔드 개발자가 되기위해 공부하고있는 조준호입니다.  
이전 포스팅 에서는 JWT 토큰의 개념과 구성요소 그리고 엑세스 토큰과 리프레시 토큰에 대해 알아보았습니다.  
이번 포스팅 에서는 이전에 알아본 JWT 토큰 기반 인증을 구현 해보겠습니다.

## JWT 토큰 기반 인증 구현

### 1. 프로젝트 환경

#### 📚Spring Boot, SpringSecurity

- **Spring Boot**: 3.4.0 버전
  ![SpringBoot 버전](/assets/img/postImg/12:24/SpringBootVersion.png)
- **Spring Security**: 6.4.1 버전
  ![시큐리티 버전](/assets/img/postImg/12:24/SecurityVersion.png)

### 2. 종속성 및 프로젝트 기본 설정

#### 종속성 추가

```groovy
dependencies {
  // Spring Boot + Security + Test + dev
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'org.springframework.boot:spring-boot-starter-security'
  implementation 'org.springframework.boot:spring-boot-starter-validation'
  implementation 'org.springframework.boot:spring-boot-starter-web'
  compileOnly 'org.projectlombok:lombok'
  developmentOnly 'org.springframework.boot:spring-boot-devtools'
  runtimeOnly 'com.mysql:mysql-connector-j'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'org.springframework.security:spring-security-test'
  testRuntimeOnly 'org.junit.platform:junit-platform-launcher'

  //JWT
  implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
  runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
  runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
}
```

#### 프로젝트 기본 설정

```yaml
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/${SCHEMA_NAME}
    username: yourDatabaseId
    password: yourDatabasePassword
  jpa:
    hibernate:
      ddl-auto: none # create, create-drop, update, validate, none 중 택1
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect
    show-sql: true
  jwt:
    secret: ${SECRET_KEY} # openssl rand -hex 64 이 명령어를 터미널에 입력해서 생성 가능!
    expiration-time: ${EXPIRATION_TIME} # 1시간: 1 * 60 * 60 * 1000 = 3600000
    refresh-expiration-time: ${REFRESH_EXPIATION_TIME} # 24시간: 86400000
```

> **ddl-auto**  
> Hibernate가 DataBase 스키마를 어떻게 관리할지 결정 합니다.   
> **`create`**: 어플리케이션 시작 시 기존 스키마를 삭제하고 새로 생성 합니다.(데이터 날라감)  
> **`create-drop`**: `create`와 같지만, 어플리케이션 종료 시 스키마를 삭제 합니다.  
> **`update`**: 기존 스키마를 유지 시키면서 변경된 부분만 업데이트 합니다.  
> **`validation`**: 엔티티와 테이블이 정확히 매핑 되었는지만 확인 합니다.  
> **`none`**: 스키마 자동 생성 기능을 사용하지 않습니다.
> {: .prompt-info}

### 3. 데이터 베이스 생성

저는 **`ddl-auto`** 를 none로 설정 하기 때문에 일단 스키마를 만들고 로직을 생성 합니다.
근데 none으로 설정 하는게 아니면 굳이 이렇게 할 필요는 없음

#### IntelliJ에서 DB 생성

1. IntelliJ 우측에 DB를 클릭했을 때 나오는 창에서 `+`클릭
   ![DB 생성](/assets/img/postImg/12:24/DBSource.png)
2. MySQL 데이터 소스를 생성하면 다음과 같은 창이 나오는데 **username, password** 입력하면 됨
   ![img.png](/assets/img/postImg/12:24/DBCreate.png)

#### 쿼리 작성

```mysql
CREATE DATABASE jwt_auth; # 스키마 생성
USE jwt_auth; # 스키마 사용
CREATE TABLE jwt_auth.user
(                                                 # 테이블 생성
  id           BIGINT PRIMARY KEY AUTO_INCREMENT, # 사용자 고유 번호
  username     varchar(255),                      # 인증에 사용할 사용자 아이디(이메일 형식을 사용 할거임)
  password     varchar(255),                      # 인증에 사용할 비밀번호(소문자(영) or 대문자(영) + 숫자 + 특수기호 포함)
  phone_number varchar(255),                      # 010-xxxx-xxxx 형식으로 작성
  address      varchar(255),                      # 이것도 정규식이 있는거로 알고있는데 잘 모르기도 하고 당장 중요한건 아니니까 패스
  role         varchar(255),                      # 사용자 권한
  created_at   datetime,                          # 생성 일시
  updated_at   datetime                           # 수정 일시
);
```

> **쿼리에 별도의 설정없이 cloumn만 생성한 이유**
> 1. 어플리케이션 레벨에서 데이터를 검증 후 DB로 전달하는 로직을 구현해서 DB의 종속성을 줄이기 위함 입니다.
> 2. DTO 로직에서 @NotBlank, @Size 등의 유효성 검증을 통해 데이터의 무결성을 보장 합니다.
> 3. 중복 여부가 중요한 **`username`** column은 비즈니스 로직에서 DB와의 사전 검증을 통해 중복을 방지하고, 중복 데이터가 감지되면 예외를 발생 시킵니다.
     {: .prompt-tip}

### 4. 엔티티 생성

#### BaseEntity

각 도메인마다 중복되는 엔티티를 BaseEntity에 작성

```java

@Getter
@MappedSuperclass
public abstract class BaseEntity {

  @Column(name = "created_at")
  private LocalDateTime createdAt;

  @Column(name = "updated_at")
  private LocalDateTime updatedAt;

  @PrePersist
  protected void onCreate() {
    this.createdAt = LocalDateTime.now();
    this.updatedAt = LocalDateTime.now();
  }

  @PreUpdate
  protected void onUpdate() {
    this.updatedAt = LocalDateTime.now();
  }
}
```

#### User 엔티티

```java

@Entity(name = "user")
@Table
@NoArgsConstructor
public class User {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(name = "username")
  private String username;

  @Column(name = "password")
  private String password;

  @Column(name = "phone_number")
  private String phoneNumber;

  @Column(name = "address")
  private String address;

  @Column(name = "role")
  @Enumerated(value = EnumType.STRING)
  private Role role;

  @Builder
  public User(String username, String password, String phoneNumber, String address,
    String nickname, Role role) {
    this.username = username;
    this.password = password;
    this.phoneNumber = phoneNumber;
    this.address = address;
    this.role = role;
  }
}
```

### 5. Security 관련 설정

#### SecurityConfig

```java

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

  private final JwtTokenProvider jwtTokenProvider;

  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }

  @Bean
  public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration)
    throws Exception {
    return configuration.getAuthenticationManager();
  }

  @Bean
  public JwtAuthenticationFilter jwtAuthenticationFilter(
    AuthenticationManager authenticationManager) {
    return new JwtAuthenticationFilter(jwtTokenProvider, authenticationManager);
  }

  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http,
    CustomLogoutHandler customLogoutHandler, AuthenticationManager authenticationManager)
    throws Exception {
    http
      .csrf(AbstractHttpConfigurer::disable)
      .formLogin(AbstractHttpConfigurer::disable)
      .httpBasic(AbstractHttpConfigurer::disable);
    http
      .authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/user", "/api/user/login")
        .permitAll()
        .requestMatchers("/api/user/{username}").authenticated()
        .requestMatchers("/api/board/**", "/api/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated());
    http
      .logout(logout -> logout
        .logoutUrl("/api/user/logout")
        .addLogoutHandler(customLogoutHandler)
        .logoutSuccessHandler(
          ((request, response, authentication) -> SecurityContextHolder.clearContext())));
    http
      .sessionManagement(session -> session
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS));
    http
      .addFilterBefore(jwtAuthenticationFilter(authenticationManager),
        UsernamePasswordAuthenticationFilter.class);
    return http.build();
  }
}
```

1. **passwordEncoder 메서드**: 비밀번호 암호화 및 검증 메서드
2. **authenticationManager 메서드**: 인증 작업 처리
3. **jwtAuthenticationFilter 메서드**: JWT 토큰을 검증 및 인증
4. **securityFilterChain 메서드**: 보안 설정. (URL 접근 권한, 로그아웃 처리, 세션 관리, JWT 필터 추가 등)

#### JwtAuthenticationFilter

```java

@Slf4j
public class JwtAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

  private final JwtTokenProvider jwtTokenProvider;

  public JwtAuthenticationFilter(JwtTokenProvider jwtTokenProvider,
    AuthenticationManager authenticationManager) {
    super(new AntPathRequestMatcher("/api/**"));
    this.jwtTokenProvider = jwtTokenProvider;
    setAuthenticationManager(authenticationManager);
  }

  @Override
  public Authentication attemptAuthentication(HttpServletRequest request,
    HttpServletResponse response) throws AuthenticationException, IOException, ServletException {

    throw new UnsupportedOperationException(
      "JwtAuthenticationFilter는 attemptAuthentication을 사용하지 않습니다.");
  }

  @Override
  protected boolean requiresAuthentication(HttpServletRequest request,
    HttpServletResponse response) {
    String requestURI = request.getRequestURI();
    // 로그인 및 회원가입 경로 제외
    return !(requestURI.equals("/api/user") || requestURI.equals("/api/user/login"));
  }

  @Override
  public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    throws IOException, ServletException {
    String token = jwtTokenProvider.resolveAccessToken(
      (HttpServletRequest) request); // 요청 헤더에서 토큰 가져오기
    if (token != null && jwtTokenProvider.validateToken(token)) { // 토큰 유효성 검사
      SecurityContextHolder.getContext()
        .setAuthentication(jwtTokenProvider.getAuthentication(token));
    }
    chain.doFilter(request, response); // 필터 체인 계속 진행
  }
}
```

1. **AbstractAuthenticationProcessingFilter**: Spring Security에서 제공하는 인증 필터 추상 클래스 입니다. 이 필터를 상속받아
   인증 로직을 커스터 마이징 할 수 있습니다.
2. **attemptAuthentication 메서드**: 일반적인 로그인 시 인증 과정을 처리하는 메서드로, JWT 인증은 로그인 요청을 별도로 하지 않기 때문에 해당 메서드를
   사용하지 않습니다. 만약 이 메서드가 호출 되면 **`UnsupportedOperationException`** 가 예외를 던집니다.
3. **requiresAuthentication 메서드**: 특정 요청에 필터를 적용할지 여부를 결정하는 메서드로, **`'/api/user'`(회원가입)** 및 
   **`'/api/user/login'`(로그인)** 요청은 JWT 인증이 필요하지 않기 때문에 필터를 생략합니다.
4. **doFilter 메서드**: 필터의 핵심 로직으로, 요청에서 JWT를 추출하고 검증한 후, 인증 정보를 설정합니다.

