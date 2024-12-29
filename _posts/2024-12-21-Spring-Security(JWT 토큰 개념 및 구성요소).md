---
title: JWT 토큰 개념 및 구성요소
description: JWT 토큰 개념 및 구성요소
author: JoJunHo
date: 2024-12-21 11:33:00 +0800
categories: [ java, SpringBoot, SpringSecurity, JWT ]
tags: [ java, SpringBoot, SpringSecurity, JWT ]
pin: false
---

> ### SpringSecurity
> [SpringSecurity - 1](https://whwnsgh0258.github.io/posts/Spring-Security/)  
> [SpringSecurity - 구조 및 처리 흐름](https://whwnsgh0258.github.io/posts/Spring-Security(%EA%B5%AC%EC%A1%B0%EC%99%80-%EC%B2%98%EB%A6%AC-%ED%9D%90%EB%A6%84)/)  
> [SpringSecurity - FormLogin](https://whwnsgh0258.github.io/posts/Spring-Security(FormLogin-%EC%9D%B8%EC%A6%9D-%EB%B0%8F-%EA%B6%8C%ED%95%9C-%EB%B6%80%EC%97%AC)/)

안녕하세요. 백엔드 개발자가 되기위해 공부하고있는 조준호입니다.  
오늘은 내용은 **JWT(JSON Web Token)** 의 개념과 구성요소에 대해 알아보려 합니다.

## 1. JWT 토큰 이란?

**JWT(JSON Web Token)** 은 JSON 포맷을 사용하여 정보를 안전하게 전송하는 토큰 기반 인증 시스템 입니다.

주로 웹 어플리케이션에서의 인증을 위해 사용되며, JWT는 사용자가 **로그인**을 하면 서버에서 인증 정보를 담은 토큰을 생성하고, 클라이언트는 서버에서 생성된 토큰을
저장하여 이후의 요청에 이를 포함시켜 서버와 통신 합니다.

이를 통해 서버는 매 요청마다 세션을 조회하는 대신 토큰을 통해 사용자를 인증해 서버의 부하를 줄입니다.

> JWT는 JSON 포맷을 사용해 데이터를 구조화 하고, BaseUrl64 인코딩을 통해 데이터를 문자열 형태로 전송할 수 있습니다.
> 또한, **서명(Signature)** 을 사용하여 데이터의 무결성을 보장하며, 토큰의 변조를 방지 할 수 있습니다.  
{: .prompt-tip}

## 2. JWT 토큰은 왜 쓰이고, 어디에 써야할까?

### 2-1. JWT 사용이유

JWT 토큰을 사용하면 Stateless(상태 비유지)한 인증을 가능하게 하여 **확장성**과 **분산 시스템에서의 활용**에 용이 합니다.

1. **서버 부담 감소**: JWT는 인증 정보를 서버가 아닌 클라이언트에 저장하므로 서버는 상태를 관리 할 필요가 없고 클라이언트에서 요청시 토큰을 포함하여 요청을 보내기
   때문에 서버측 자원을 줄일 수 있습니다.
2. **확장성**: 서버가 클라이언트의 상태를 저장하지 않기 때문에 여러대의 서버가 동일한 JWT를 사용하여 요청을 처리할 수 있습니다.
3. **독립적인 요청 처리**: JWT는 인증이 필요한 각 요청마다 토큰을 포함하여 서버가 전달 받기 때문에, 각 요청은 독립적으로 처리 됩니다
4. **크로스 도메인 인증 용이**: 마이크로서비스 아키텍처에서 서로 다른 서비스들이 JWT 토큰을 공유하며 인증할 수 있습니다.

### 2-2. 토큰 VS 세션 인증 플로우 비교

1. **토큰 기반 인증 플로우(SecretKey 사용)**: 토큰이 보안, 사용자 데이터를 관리  
   ![토큰 기반 인증 플로우](/assets/img/postImg/12:21/tokenAuthentication.png)
2. **세션 기반 인증 플로우**: 서버에서 보안, 사용자 데이터 관리  
   ![토큰 기반 인증 플로우](/assets/img/postImg/12:21/sessionAuthentication.png)

|          | 세션 기반 인증                           | 토큰 기반 인증                        |
  |----------|------------------------------------|---------------------------------|
| 상태 관리    | 	서버에서 SessionID 관리 **(Stateful)**	 | 서버는 상태를 관리하지 않음 **(Stateless)** | 
| 인증 정보 저장 | 세션 ID는 서버에 저장                      | JWT는 클라이언트에 저장                  |
| 요청 간 독립성 | 세션 ID 확인을 위해 데이터베이스 필요             | JWT 자체로 독립적 인증 가능               |
| 확장성      | 서버 간 세션 동기화 필요                     | 서버 간 상태 동기화 불필요                 |
| 보안       | 세션 탈취에 취약                          | 서명 및 암호화로 변조 방지 가능              |

## 3. JWT 구성 요소

**JWT**는 **`Header`**, **`Payload`**, **`Sigunature`**로 이루어져 있으며, 이 세부분은 **`.(dot)`** 으로구분합니다.
이렇게 생성된 **JWT**는 최종적으로 **Base64Url**로 인코딩 되어 클라이언트와 서버간에 전달 됩니다.

> **Base64Url 이란?**  
> Base64Url은 URL과 파일 이름에 안전하게 포함될 수 있도록 수정된 Base64의 변형이라고 할 수 있습니다. JWT와 같은 인증 토큰은 Base64Url로 인코딩되어
> 사용되며, 이는 URL 파라미터나 헤더에서 문제없이 사용할 수 있도록 설계된 방식입니다.
 {: .prompt-tip}

### 3-1. Header - 헤더

- JWT의 **헤더(Header)** 는 토큰의 타입과 서명에 사용할 알고리즘 정보를 포함합니다.
- **구조**: JSON 형태
- **필수 필드**:
  - **arg**: 서명에 사용하는 해시 알고리즘(예시: HS256, RS256)
  - **type**: 토큰의 타입(항상 JWT)
- 예시
  ```json
  {
    "arg": "HS256",
    "type": "JWT"
  }
  ```

### 3-2. Payload - 사용자 정의

- JWT의 **Payload**는 토큰에 담을 실제 데이터를 포함하며, **Claim**으로 표현 됩니다.
- Claim은 인증 및 권한 부여와 관련된 정보나 사용자 정의 데이터를 담을 수 있습니다.
- **구조**: JSON 형태
- **주요 Claim**:
  - **등록된 클레임(Registered Claims)**: 표준에 의해 정의된 Claim
    - **iss(Issuer)**: 토큰 발급자
    - **sub(Subject)**: 토큰의 주제(주로 사용자 ID)
    - **aud(Audience)**: 토큰의 대상(어떤 서비스나 어플리케이션에서만 유효한지 나타냄)
    - **iat(Issued At)**: 토큰 발급 시간
    - **exp(Expiration Time)**: 토큰 만료 시간
  - **미등록 클레임(Private Claims)**: 사용자 정의 정보
    - 예) email, nickname 등등

  - **예시**
    ```json
    {
     "sub": "test@email.com",
     "iss": "table-link.com",
     "auth": "USER",
     "iat": 1734852534,
     "exp": 1734854334
    }
    ```
     

### 3-3. Signature - 알고리즘 + 비밀 키

- JWT의 **서명(Signature)** 은 토큰의 무결성을 검증하기 위해 사용되며, 헤더와 페이로드를 결합한 후 지정된 알고리즘과 비밀 키로 서명합니다.
- **역할**:
  - 서명이 유효한지 확인합니다.(서명이 유효하지 않으면 토큰은 변조 되었다고 간주 합니다.)
- **생성 방식**
  ```text
  Signature = HMACSHA256(
    Base64UrlEncode(Header) + "." + Base64UrlEncode(Payload),
    secretKey
  )
  ```
  
## 4. AccessToken VS RefreshToken

### 4-1. AccessToken

사용자에 대한 정보를 담고 있어서 서비스에 **접근(Access)** 할 수 있는 토큰을 의미 합니다. 

- **특징**:
  - 짧은 유효기간
  - 서버에 보호된 리소스에 접근 할 때 사용

### 4-1. RefreshToken

AccessToken이 만료 되었을 때 서버에서 이를 확인하고 새로운 AccessToken을 발급 해주기 위한 토큰 입니다.

- **특징**
  - 긴 유효기간
  - 서버에서만 사용되는 토큰
  
