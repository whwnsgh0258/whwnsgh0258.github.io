---
title: SpringSecurity(구조와 처리 흐름)
description: SpringSecurity(구조와 처리 흐름)
author: JoJunHo
date: 2024-12-19 11:33:00 +0800
categories: [ java, SpringBoot, SpringSecurity ]
tags: [ java, cs, Spring, SpringBoot,SpringSecurity ]
pin: false
---
[SpringSecurity - 1](https://whwnsgh0258.github.io/posts/Spring-Security/)

## 1. SpringSecurity 구조와 처리 흐름

### SpringSecurity 구조

> 아래의 다이어그램은 SpringSecurity 인증 및 권한 부여 과정에서 사용되는 주요 컴포넌트와 관계를 나타냅니다.

![SpringSecurity 컴포넌트 구조](/assets/img/postImg/12:19/Springsecurity구조%20머메이드.png)

#### 1. SpringFilterChain

- HTTP 요청을 처리하기 위해 **FilterChain**을 구성 합니다.
- HttpSecurity로 설정된 보안 설정을 기반으로 작동합니다.
- **주요 메서드**
  - **getFilter**: 필터 목록 반환
  - **matches(HttpServletRequest)**: 요청이 필터체인에 매칭 되는지 확인
  - 구현 예시  
    ```java
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http,
        CustomLogoutHandler customLogoutHandler, AuthenticationManager authenticationManager) throws Exception {
      http
          .csrf(AbstractHttpConfigurer::disable)
          .formLogin(AbstractHttpConfigurer::disable)
          .httpBasic(AbstractHttpConfigurer::disable);
    http
          .authorizeHttpRequests(auth -> auth
          .requestMatchers("/api/user", "/api/user/login")
          .permitAll()
          .requestMatchers("/api/user/**")
          .hasAnyRole("USER", "ADMIN")
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
          .addFilterBefore(jwtAuthenticationFilter(authenticationManager), UsernamePasswordAuthenticationFilter.class);
    return http.build();
    }
    ```

#### 2. HttpSecurity

- **보안 설정 API**를 제공 합니다.
- SecurityFilterChain을 생성하기 위한 설정을 정의 합니다.
- **주요 메서드**
  - **csrf()**: CSRF 보호 설정, 특정 URL 예외처리, CSRF 토큰 관리
  - **authorizeHttpRequest()**: URL 기반의 접근 권한 설정, 역할(Role) 기반의 권한 설, 인증 및 인가 규칙 정의
  - **formLogin()**:
  - **logout()**: 로그아웃 처리 설정, 세션 및 쿠키 삭제, 로그아웃 후 처리
  - **sessionManagement()**: 세션 정책 설정
  - **headers()**: 보안 관련 HTTP 헤더 설정, XSS 보호 및 클릭잭킹 방지
  - **oauth2Login()**: OAuth2 인증 설정, 소셜 로그인 처리(카카오, 네이버, 구글 등)
  - **addFilterBefore()/addFilterAfter()**: 커스텀 필터 추가, 필터 체인 순서 조정
  - **rememberMe()**: Remember-Me 인증 설정, 토큰 유효기간 설정, 영구 토큰 저장소 설정
  - **exceptionHandling()**: 보안 예외처리 설정, 접근 거부 처리, 인증 실패 처리

#### 3. AbstractAuthenticationProcessFilter

- 인증을 처리하는 **추상 클래스 필터 입니다**.
- 인증 요청을 AuthenticationManager로 전달하고, 결과에 따라 성공과 실패 처리를 수행 합니다.
- **주요 메서드**
  - **attemptAuthentication()**: 인증 시도
  - **successfulAuthentication()**: 인증 성공
  - **unsuccessfulAuthentication()**: 인증 실패

#### 4. AuthenticationManager

- 인증 처리에 진입점이 되는 인터페이스, 인증을 처리하는 중앙 관리자 역할을 합니다.
- 인증을 요청 받아 **AuthenticationProvider**로 위입합니다.
- **주요 메서드**
  - **authentication(Authentication)**: 인증 요청 처리

#### 5. ProviderManager

- AuthenticationManager 인터페이스를 구현한 구현체로, 다수의 AuthenticationProvier를 체인으로 관리 합니다.
- 요청에 적합한 AuthenticationProvider를 찾아서 인증을 위임합니다.
- **주요 필드**
  - **Providers: AuthenticationProvider** 목록

#### 6. AuthenticationProvider

- **인증 로직을 실제로 수행**하는 인터페이스 입니다.
- 다양한 인증 방식을 지원하며 여러 구현체가 존재합니다.
- **주요 메서드**
  - **authentication(Authentication)**: 인증 처리
  - **supports(Class)**: 지원하는 인증 유형(예시: JWT, OAuth2)

#### 7. JwtAuthenticationProvider

- **JWT 기반 인증**을 처리하는 AuthenticationProvider 구현체 입니다.
- JWT 토큰을 검증하고 사용자 정보를 추출 합니다.

#### 8. OAuth2AuthenticationProvider

- **OAuth2 기반 인증**을 처리하는 AuthenticationProvider의 구현체.
- OAuth2 프로토콜을 통해 인증을 수행합니다.

#### 9. UserDetailsService

- 사용자 정보를 로드하는 서비스 인터페이스 입니다.
- 인증 과정에서 사용자의 상세 정보를 제공 합니다.
- **주요 메서드**
  - **loadUserByUsername(String)**: 사용자의 이름을 기반으로 UserDetails 로드
- 구현 예시
  ```java
  @Service
  @RequiredArgsConstructor
  public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found with username: " + username));
        List<GrantedAuthority> authorities = List.of(new SimpleGrantedAuthority(user.getRole().getValue()));

        UserResponse userResponse = UserResponse.toDto(user);
        return new CustomUserDetails(userResponse, authorities);
    }
  }
  ```

#### 10. SecurityContext

- 인증 정보를 저장하는 객체 입니다.
- 어플리케이션 전역에서 사용자 인증 정보를 관리 합니다.
- **주요 메서드**
  - **getAuthentication()**: 현재 인증된 사용자의 정보를 가져옵니다.
  - **setAuthentication(Authentication)**: 인증 정보를 설정 합니다.

#### 11. SecurityContextHolder

- SecurityContext를 관리하는 스프링 시큐리티의 **중앙 관리 클래스** 입니다.
- 기본적으로 ThreadLocal을 사용하며 각 스레드 마다 별도의 SecurityContext를 유지합니다.
- **주요 메서드**
  - **getContext()**: 현재 스레드의 SecurityContext를 가져옵니다.
  - **setContext(SeccurityContext)**: 현재 스레드의 SecurityContext를 설정 합니다.

#### 12. Authentication

- 인증 정보를 나타내는 인터페이스 입니다.
- 사용자 정보와 권한을 나타냅니다.
- **주요 필드**
  - **getPrincipal**: 사용자 정보 반환
  - **getCredentials**: 자격 증명 반환(비밀번호)
  - **getAuthorities()**: 사용자 권한 정보

#### 13. UserDetails

- 사용자의 정보를 표현하는 인터페이스 입니다.
- **주요 메서드**
  - **getUsername()**: 사용자 이름 반환
  - **getPassword()**: 사용자 비밀번호 반환
  - **getAuthorities()**: 사용자 권한 정보 반환
- 구현 예시
  ```java
  @Getter
  @RequiredArgsConstructor
  public class CustomUserDetails implements UserDetails {
  
    private final UserResponse userResponse;
    private final List<GrantedAuthority> authorities;
  
  
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
      return List.of(new SimpleGrantedAuthority(userResponse.getRole().name()));
    }

    @Override
    public String getPassword() {
      return userResponse.getPassword();
   }
  
    public Long getUserId(){
      return userResponse.getId();
    }

    @Override
    public String getUsername() {
      return userResponse.getUsername();
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

### SringSecurity 인증 처리 흐름
![SpringSecurity 처리 흐름 머메이드.png](/assets/img/postImg/12:19/SpringSecurity%20처리%20흐름%20머메이드.png)

1. **Client**: 사용자 요청이 들어옵니다.
2. **SecurityFilterChain**: 사용자의 요청을 SecurityFilterChain을 통해 들어오게 됩니다.
3. **AuthenticationFilter**: 요청을 AuthenticationFilter가 가로채고, 인증을 처리 합니다. 인증되지 않은 사용자가 요청을 보내면 인증 처리를 위해 AuthenticationManager로 인증 객체를 전달 합니다.
4. **AuthenticationManager**: 인증을 담당하는 AuthenticationManager는 여러 AuthenticationProvider 중 적절한 곳에 인증을 위임합니다.
5. **AuthenticationProvider**: AuthenticationProvider는 실제 인증을 처리 합니다. 주로 UserDetaillsService를 호출해 사용자의 정보를 로드 합니다.
6. **UserDetailsService**: UserDetailsService는 사용자 이름을 기반으로 사용자 정보를 로드하고, UserDetails 객체를 반환합니다.
7. **SecurityContext**: 인증이 완료되면 Authentication 객체는 SecurityContext에 저장되어, 이후의 요청에서 인증된 사용자의 정보를 확인할 수 있게 됩니다.
8.	**HttpServletRequest**: 인증 완료된 사용자 정보가 담긴 SecurityContext는 요청을 처리한 후, HTTP Response로 사용자에게 반환됩니다.
