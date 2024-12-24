---
title: SpringSecurity
description: SpringSecurity
author: 조준호
date: 2024-12-18 11:33:00 +0800
categories: [ java, Spring, SpringSecurity ]
tags: [ java, cs, Spring, SpringBoot,SpringSecurity ]
pin: false
---

## 1. SpringSecurity란?

> **Spring Security**는 **Spring Framework**의 하위 보안 프레임워크로 스프링 기반의 웹 어플리케이션에서 인증(**Authentication**)
> 및 인가(**Authorization**)를 담당합니다.
> {: .prompt-info }

## 2. SpringSecurity 주요 기능

### 1. 인증(Authentication)

- 리소스에 접근하는 사용자가 누구인지 확인하는 과정 입니다.
- **인증 방식**:
  - **폼 로그인(Form Login)**: 사용자 로그인 정보를 HTML form을 통해 입력받아 인증을 처리하는 방식.
  - **HTTP Basic 인증**: HTTP 프로토콜의 `Authorization`헤더에 사용자 이름과 비밀번호를 Base64로 인코딩하여 전달하는 방식
  - **OAuth2**: 외부 인증 서버(Kakao, Google, FaceBook 등)를 통해 사용자 인증을 처리하는 방식입니다.
  - **JWT(JSON Web Token)**: Json 형식으로 인코딩 된 토큰을 사용하여 인증과 권한 정보를 전달합니다.
  - **LDAP 인증**: 주로 기업의 백오피스 영역에서 사용하는 인증 방식으로 알고있음...

### 2. 인가 - 권한 부여(Authorization)

- 인증된 사용자가 특정 리소스나 기능에 접근할 수 있는 권한이 있는지 확인하는 로직
- 역할(Role) 또는 권한(Authority) 기반으로 접근을 제어합니다.
  - URL 패턴, 메서드, 데이터에 대한 세부적인 권한

### 3. CSRF 방어

- **CSRF(Cross-Site Request Forgery)** 공격을 방지하기 위한 기본적인 방어 메커니즘을 제공 합니다.

> CSRF 공격은 사용자의 의도와 상관없이 악의적인 요청을 보내는 공격입니다.
> {: .prompt-info }

### 4. 세션 관리

- 세션 타임아웃 설정과 동시 세션을 제한 합니다.
- **세션 고정 공격 방지(Session Fixation Protection)** 기능을 제공합니다.

> 1. **세션 타임아웃**: 사용자가 일정 시간동안 사용하지 않으면 세션을 자동을 만료 합니다.
> 2. **동사 새션 제한**:  동일한 계정으로 여러 장치에서 동시에 로그인하지 못하도록 제한합니다.
> 3. **세션 고정 공격 방지**: 로그인 시 세션 ID를 새로 발급하여 공격자가 고정한 세션을 무력화합니다.
     {: .prompt-info }

### 5. 비밀번호 암호화

- 안전한 비밀번호 관리를 위해 `BCryptPasswordEncoder`와 같은 암호화 도구를 지원합니다.
- 평문 비밀번호 저장을 방지하고, 안전하게 해시된 비밀번호를 사용합니다.

### 6. 보안 헤더 설정

- HTTP 응답 헤더를 설정하여 **클릭재킹(Clickjacking)**, **MINE 타입의 스니핑**등의 공격을 방어합니다.
  - `X-Frame-Options`
  - `X-Content-Type-Options`
  - `X-XSS-Protection`

> SpringSecurity를 사용하면 기본적으로 설정이 됩니다.  
> ![SpringSecurityHeader](/assets/img/postImg/12:18/SpringSecurityHeader.png)
> {: .prompt-tip }

## 3. Spring Security 버전

저는 우선 이후의 Security 관련 포스팅을 Spring Security 6.x 버전으로 작성할 것이기 때문에 6.x 버전에 대한 내용과 이전 버전인 5.x 버전과의 차이점을
알아보겠습니다.

### 3-1. Spring Security 6.x 버전

**Spring Security 6.x**는 Spring Framework 6와 Spring Boot 3과 함께 작동하며, JakartaEE 9 이상의 API로 전환된 보안 기능을
제공하며, Java 17 이상의 버전을 요구합니다.

### 3-2. Spring Security 5.x VS 6.x 버전

1. **Java 및 Spring Framework 버전 요구사항**
   - **5.x**: Java 8 이상 및 Spring Framework 5.x와 호환됩니다.
   - **6.x**: Java 17 이상 및 Spring Framework 6.x와 호환 됩니다.
2. **Jakarta EE 전환**
   - Spring Security는 javax 패키지에서 jakarta 패키지로 변경 되었습니다.
3. **SecurityFilterChain으로 전환**
   - **5.x**: WebSecurityConfigurerAdapter를 상속받아 보안 설정을 구현 했습니다.
   - **6.x**: WebSecurityConfigurerAdapter가 제거되고, SecurityFilterChain 빈을 사용하여 보안 설정을 구현 합니다.
4. **Authorization 설정 변경**
   - **5.x**: authorizeRequest()를 사용합니다.
   - **6.x**: authorizeHttpRequest()로 대체되고, 더 세분화된 조건과 메서드 메칭 기능을 제공 합니다.
