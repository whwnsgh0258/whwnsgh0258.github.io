---
title: SpringSecurity(FormLogin - 인증 및 권한 부여)
description: SpringSecurity(FormLogin -인증 및 권한 부여)
author: JoJunHo
date: 2024-12-20 11:33:00 +0800
categories: [ java, SpringBoot, SpringSecurity, FormLogin ]
tags: [ java, Spring, SpringBoot,SpringSecurity, FormLogin ]
pin: false
---

> ### SpringSecurity
> [SpringSecurity - 1](https://whwnsgh0258.github.io/posts/Spring-Security/)  
> [SpringSecurity - 구조 및 처리 흐름](https://whwnsgh0258.github.io/posts/Spring-Security(%EA%B5%AC%EC%A1%B0%EC%99%80-%EC%B2%98%EB%A6%AC-%ED%9D%90%EB%A6%84)/)

## SpringSecurity 인증 및 권한 부여

### 인증과 권한 부여의 차이

- **인증(Authentication)**: 사용자가 누구인지 확인하는 과정입니다.
- **권한 부여(Authorization)**: 인증된 사용자가 특정 리소스 혹은 URL에 접근할 권한이 있는지 결정하는 과정 입니다.

### FormLogin 인증 및 인가

#### 1. 의존성

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.mysql:mysql-connector-j'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

#### 2. 프로젝트 기본 설정

```yaml
spring:
  application:
    name: SpringSecurity
  datasource:
    url: ${LOCAL_DB_URL} # 데이터베이스 주소: jdbc:mysql://localhost:3306/DataBase이름
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: ${LOCAL_DB_USER} # MySQL에서 설정한 Username(기본값: root)
    password: ${LOCAL_DB_PW} # MySQL에서 설정한 Password

    jpa:
      hibernate:
        ddl-auto: none # Hibernate DDL 생성방식.(none는 자동으로 생성 안하고 테이블 하나씩 만들어야함)
      show-sql: true # 실행되는 SQL 쿼리를 콘솔에 띄워주는 설정.
      properties:
        hibernate:
          dialect: org.hibernate.dialect.MySQL8Dialect # MySQL 8 버전에 맞는 Hibernate dialect 설정.
          format_sql: true # 쿼리를 콘솔에 출력할 때 한줄씩 들여쓰기해서 나오게하는 설정(이거 안하면 쿼리가 한줄에 다나옴)

```

#### 3. SecurityConfig FilterChain 설정

아래의 코드는 부트캠프 할 때 final 프로젝트에서 작성한 SecurityConfig의 FilterChain 메서드 입니다.

##### SecurityFilterChain 설정

- **securityMatcher()** (특정 URL 패턴에 대해 보안 필터를 적용)
  ```java
    .securityMatcher("/", "/views/**", "/views/users/login", "/views/users/signup",
            "/views/edit", "/user/**", "/api/**")
  ```
- **authorizeHttpRequests()** (요청에 대한 권한 부여 규칙을 설정)
  ```java
  .authorizeHttpRequests(authorize -> authorize
      // 정적 자원(css, 이미지, JS 등)과 일부 페이지에 대해서는 누구나 접근 허용
      .requestMatchers("/css/**", "/images/**", "/js/**", "/favicon.*",
      "/*/icon-*").permitAll()
      // 로그인, 회원가입, 메인 페이지 등 공용 페이지에 대해서는 누구나 접근 허용
      .requestMatchers("/", "/views/users/login", "/views/users/signup",
      "/api/users/signup", "/views/main", "/actuator/health")
      .permitAll()
      // 그 외의 요청은 인증이 필요함
      .anyRequest().authenticated()
  )
  ```
- **formLogin()** (폼 로그인 관련 설정)
  ```java
  .formLogin(form -> form
          .loginPage("/views/users/login") // 로그인 페이지 URL
          .loginProcessingUrl("/views/users/login") // 로그인 처리 URL
          .defaultSuccessUrl("/", true) // 로그인 성공 시 이동 URL
          .failureHandler(authenticationFailureHandler()) // 로그인 실패 핸들러
          .permitAll() // 모든 사용자 접근 가능
  )
  ```
- **rememberMe()** (Remember-Me 기능 설정으로 자동 로그인)
  ```java
    .rememberMe(rememberMe -> rememberMe
            // Remember-Me 토큰의 고유한 키를 설정
            .key("uniqueAndSecret")
            // Remember-Me 토큰의 유효 시간을 설정 (86400초 = 1일)
            .tokenValiditySeconds(86400)
            // Remember-Me 기능에 사용할 UserDetailsService 설정
            .userDetailsService(userDetailService))
  ```
- **logout()** (로그아웃 설정)
  ```java
  .logout(logout -> logout
              // 로그아웃 처리 URL 설정
              .logoutUrl("/api/users/logout")
              // 로그아웃 후 이동할 URL 설정
              .logoutSuccessUrl("/")
              // 로그아웃 시 삭제할 쿠키 설정
              .deleteCookies("JSESSIONID", "remember-me")
              // 로그아웃 후 세션을 무효화
              .invalidateHttpSession(true)
              // 인증 정보를 초기화
              .clearAuthentication(true)
              // 로그아웃 페이지는 누구나 접근할 수 있도록 설정
              .permitAll()
      )
  ```
- **authenticationProvider()** (인증을 처리하는 커스텀 AuthenticationProvider 설정)
  ```java
  .authenticationProvider(authenticationProvider);
  ```
- **AuthenticationFailureHandler** (로그인 실패를 처리하는 Handler 메서드)
  ```java
  @Bean
  public AuthenticationFailureHandler authenticationFailureHandler() {
    return new CustomAuthenticationFailureHandler();
  }
  ```

- **FilterChain 흐름**
  1. 클라이언트가 로그인 페이지로 이동 -> `.loginPage("/views/users/login")`에서 설정한 URL 제공
  2. 로그인 폼 제출 -> `.loginProcessingUrl("/views/users/login")`에 설정된 URL로 인증 요청 처리
  3. 인증 성공 -> `.defaultSuccessUrl("/", true)`로 리다이렉트
  4. 인증 실패 ->  `.failureHandler(authenticationFailureHandler())`로 실패 처리
  5. Remember-Me 활성화 → 설정된 키와 토큰 유효 기간으로 자동 로그인 지원.
  6. 로그아웃 → logoutUrl() 호출 시 세션과 쿠키 삭제 및 인증 정보 초기화.

#### 4. Entity
##### 4-1. Users
```java

@Getter
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Table(name = "users")
public class User extends BaseEntity {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(name = "username", unique = true)
  private String username;

  @Column(name = "password")
  private String password;

  @Column(name = "nickname")
  private String nickname;

  public void updateUser(String password, String nickname) {
    this.password = password;
    this.nickname = nickname;
  }
}
```

1. **BaseEntity**: createdAt, updatedAt을 기록하는 엔티티
2. **id**: 사용자 고유 식별 번호
3. **username**: 사용자 인증에 필요한 아이디
4. **password**: 사용자 비밀번호
5. **nickname**: 사용자 닉네임

#### 5. DTO
##### 5-1. CreateRequestUser
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class CreateUserRequest {

    // RFC 5322 이메일 정규식
    @Pattern(regexp = "^[a-zA-Z0-9_!#$%&’*+/=?`{|}~^.-]+@[a-zA-Z0-9.-]+\\.[a-zA-z]{2,}$", message = "아이디는 이메일 형태로 입력해주세요.")
    @NotBlank(message = "아이디는 빈칸을 입력할 수 없습니다.")
    private String username;

    // 비밀번호: 영문, 숫자, 특수문자를 포함하여 8~15자 작성
    @Size(min = 8, max = 16, message = "비밀번호는 8자 이상 16자 이하여야 합니다.")
    @Pattern(regexp = "^(?:(?=.*[a-zA-Z])(?=.*[0-9])|(?=.*[a-zA-Z])(?=.*[!@#$%^*+=-])|(?=.*[0-9])(?=.*[!@#$%^*+=-])).*$",
            message = "비밀번호는 영문, 숫자, 특수문자 중 2가지 종류를 포함해야 합니다.")
    @NotBlank(message = "비밀번호는 빈칸을 입력할 수 없습니다.")
    private String password;

    @NotBlank(message = "이름은 빈칸을 입력할 수 없습니다.")
    private String nickname;

    private Language language;
    private Position position;

    public User toEntity() {
        return User.builder()
                .username(username)
                .password(password)
                .nickname(nickname)
                .language(language)
                .position(position)
                .build();
    }
}
```
##### 5-2. UserResponse
```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UserResponse {

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private Long id;
    private String username;
    private String password;
    private String nickname;


    private Language language;
    private Level level;
    private Position position;

    public static UserResponse toDto(User user) {
        return UserResponse.builder()
                .id(user.getId())
                .username(user.getUsername())
                .password(user.getPassword())
                .nickname(user.getNickname())
                .language(user.getLanguage())
                .level(user.getLevel())
                .position(user.getPosition())
                .createdAt(user.getCreatedAt())
                .updatedAt(user.getUpdatedAt())
                .build();
    }
}
```
#### 6. Repository
##### 6-1. UserRepository
```java
public interface UserRepository extends JpaRepository<User, Long> {
  Optional<User> findByUsername(String username);
}
```
1. **Optional<User> findByUsername(String username)**: username으로 검색하고 없으면 예외처리
#### 7. Service
##### 7-1. UserService
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final InterviewRepository interviewRepository;

    // 회원 가입 메서드
    @Transactional
    public void createUser(CreateUserRequest createUserRequest) {
        userRepository.save(createUserRequest.toEntity());
    }

    // username 으로 찾는 메서드
    @Transactional(readOnly = true)
    public UserResponse findByUsername(String username) {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException(
                        "No user found with username:" + username));

        return UserResponse.toDto(user);
    }
}
```
#### 8. Controller
##### 8-1. UserController

```java
@Controller
@RequiredArgsConstructor
public class UserController {

  private final PasswordEncoder passwordEncoder;
  private final UserService userService;

  // 로그인 폼
  @GetMapping("/views/users/login")
  public String userLoginForm() {
    return "user/login";
  }

  // 회원가입 폼
  @GetMapping("/views/users/signup")
  public String userSignUpForm(Model model) {
    model.addAttribute("createUserRequest", new CreateUserRequest());

    return "user/signup";
  }
}
```

