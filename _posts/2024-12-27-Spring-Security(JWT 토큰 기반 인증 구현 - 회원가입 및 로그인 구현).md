---
title: JWT 토큰 기반 인증 구현 - 회원가입 및 로그인 구현
description: 회원가입 및 로그인 구현
author: JoJunHo
date: 2024-12-29 11:33:00 +0800
categories: [ java, SpringBoot, SpringSecurity, JWT ]
tags: [ java, SpringBoot, SpringSecurity, JWT ]
lastmod: 2024-12-30
sitemap:
  changefreq: daily
  priority: 1.0
---

> ### SpringSecurity
> [SpringSecurity - 개념](https://whwnsgh0258.github.io/posts/Spring-Security/)  
> [SpringSecurity - 구조 및 처리 흐름](https://whwnsgh0258.github.io/posts/Spring-Security(%EA%B5%AC%EC%A1%B0%EC%99%80-%EC%B2%98%EB%A6%AC-%ED%9D%90%EB%A6%84)/)  
> [SpringSecurity - FormLogin](https://whwnsgh0258.github.io/posts/Spring-Security(FormLogin-%EC%9D%B8%EC%A6%9D-%EB%B0%8F-%EA%B6%8C%ED%95%9C-%EB%B6%80%EC%97%AC)/)  
> [SpringSecurity - JWT](https://whwnsgh0258.github.io/posts/Spring-Security(JWT-%ED%86%A0%ED%81%B0-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EA%B5%AC%EC%84%B1%EC%9A%94%EC%86%8C)/)
> [SpringSecurity - JWT 토큰 기반 인증](https://whwnsgh0258.github.io/posts/Spring-Security(JWT-%ED%86%A0%ED%81%B0-%EA%B8%B0%EB%B0%98-%EC%9D%B8%EC%A6%9D-%EA%B5%AC%ED%98%84)/)

안녕하세요. 백엔드 개발자가 되기위해 공부하고있는 조준호입니다.  
이전 포스팅 에서는 JWT 토큰 인증 및 인가 구현을 위한 Security 설정을 작성했습니다.  
이번 포스팅 에서는 이전 포스팅에 이어서 Security 설정에 맞게 회원가입 및 로그인 로직을 구현해 보겠습니다.


## 회원가입

### UserController
```java
/**
 * 회원가입
 */
@PostMapping
public ResponseEntity<ApiResponse<UserResponse>> signupUser(
  @Valid @RequestBody SignUpUserRequest signUpUserRequest) {
  // 닉네임 중복 확인
  if (userService.isNicknameDuplicate(signUpUserRequest.getNickname())) {
    ApiResponse<UserResponse> errorResponse = ApiResponse.<UserResponse>builder()
      .result(null)
      .resultCode(ErrorCode.NICKNAME_ALREADY_EXISTS.getStatus())
      .resultMsg(ErrorCode.NICKNAME_ALREADY_EXISTS.getMessage())
      .build();
    return ResponseEntity.status(ErrorCode.EMAIL_ALREADY_EXISTS.getStatus()).body(errorResponse);
  // 이메일 중복 확인
  } else if (userService.isUsernameDuplicate(signUpUserRequest.getUsername())) {
    ApiResponse<UserResponse> errorResponse = ApiResponse.<UserResponse>builder()
      .result(null)
      .resultCode(ErrorCode.EMAIL_ALREADY_EXISTS.getStatus())
      .resultMsg(ErrorCode.EMAIL_ALREADY_EXISTS.getMessage())
      .build();
    return ResponseEntity.status(ErrorCode.EMAIL_ALREADY_EXISTS.getStatus()).body(errorResponse);
  } else {
    signUpUserRequest.setRole(Role.USER);
    User createdUser = userService.createUser(signUpUserRequest);
    UserResponse userResponse = UserResponse.toDto(createdUser);
    ApiResponse<UserResponse> successResponse = ApiResponse.<UserResponse>builder()
      .result(userResponse)
      .resultCode(HttpStatus.CREATED.value())
      .resultMsg("회원가입 성공")
      .build();
    return ResponseEntity.status(HttpStatus.CREATED).body(successResponse);
  }
}
```

### UserService

```java
// 회원가입 메서드
  @Transactional
  public User createUser(SignUpUserRequest signUpUserRequest) {
    User user = signUpUserRequest.toEntity();
    user.encodePassword(passwordEncoder.encode(user.getPassword()));
    return userRepository.save(user);
  }

  // 아이디 중복 검사
  @Transactional(readOnly = true)
  public Boolean isUsernameDuplicate(String username) {
    Optional<User> user = userRepository.findByUsername(username);
    return user.isPresent();
  }

  // 닉네임 중복 검사
  @Transactional(readOnly = true)
  public Boolean isNicknameDuplicate(String nickname) {
    Optional<User> user = userRepository.findByNickname(nickname);
    return user.isPresent();
  }
```

### UserRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {

  Optional<User> findByUsername(String username);

  Optional<User> findByNickname(String nickname);
}
```

### 결과
![#](/assets/img/postImg/12:29/회원가입.png)

## 로그인

### UserController

```java
/**
 * 로그인
 */
@PostMapping("/login")
public ResponseEntity<ApiResponse<LoginResponse>> loginUser(
  @Valid @RequestBody SignInUserRequest signInUserRequest) {
  LoginResponse loginResponse = userService.login(signInUserRequest.getUsername(),
      signInUserRequest.getPassword());
  // 로그인 성공 시 응답 반환
  ApiResponse<LoginResponse> response = ApiResponse.<LoginResponse>builder()
      .result(loginResponse)
      .resultCode(HttpStatus.OK.value())
      .resultMsg("로그인 성공")
      .build();

  return ResponseEntity.ok(response);
}
```

### UserService

- 로그인 로직
  ```java
  // 회원 로그인
  @Transactional
  public LoginResponse login(String username, String password) {   
    User user = getUser(username);
    validatePassword(password, user);
    UsernamePasswordAuthenticationToken authenticationToken =
        new UsernamePasswordAuthenticationToken(username, password);
    Authentication authentication = authenticationManager.authenticate(authenticationToken);
    JwtTokenResponse jwtToken = jwtTokenProvider.generateTokens(authentication);
  
    saveJwtRefreshToken(jwtToken, user);
    return LoginResponse.toDto(jwtToken);
  }
  ```
- 비밀번호 확인
  ```java
  // 비밀번호 확인
  private void validatePassword(String signInUserRequest, User user) {
    if (!passwordEncoder.matches(signInUserRequest, user.getPassword())) {
      throw new BadCredentialsException("Invalid password");
    }
  }
  ```
- RefreshToken 서버에 저장
  ```java
  private void saveJwtRefreshToken(JwtTokenResponse jwtToken, User user) {
    JwtTokenDto jwtTokenDto = new JwtTokenDto();
    jwtTokenDto.setToken(jwtToken.getRefreshToken());
    jwtTokenDto.setUserId(user);
    jwtTokenDto.setIssuedAt(LocalDateTime.now());
    jwtTokenDto.setExpiresAtFromNow();
    jwtTokenDto.setRevoked(false);

    jwtTokenRepository.save(jwtTokenDto.toEntity());
  }
  ```

### 결과
![로그인.png](/assets/img/postImg/12:29/로그인.png)
