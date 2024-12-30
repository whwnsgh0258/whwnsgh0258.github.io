---
title: REST/REST API/RESTful API란?
description: RESTful API
author: 조준호
date: 2024-12-15 11:33:00 +0800
categories: [ java, Spring Boot, REST API ]
tags: [ java, cs ]
pin: false
---

## 1. API란?

API는 **Application Programming Interface**의 약어로 소프트웨어나 어플리케이션 간에 데이터를 교환하거나, 특정 기능을 사용할 수 있도록 제공되는
인터페이스 입니다.

## 2. REST API란?

### 2-1. REST란?

**Representational State Transfer**의 약어로, 자원의 이름으로 구분하여 해당 자원의 상태(정보)를 주고 받는 형태의 아키텍처 스타일입니다.

### 2-2. REST API의 구성요소

1. **자원(Resource) - URI(Uniform Resource Identifier)**
  - 모든 자원에는 고유한 ID가 존재하고, 이를 통해 자원을 구별 합니다.
  - 자원을 구별하는 ID는 `/api/user/{user_id}`와 같은 HTTP URI입니다.
  - 클라이언트는 위와 같은 URI를 이용해 자원을 지정하고 해당 자원의 상태 정보에 대한 조작을 Server에 요청합니다.
2. **행위(Verb) - HTTP Method**
  - HTTP 프로토콜 메서드를 사용합니다.
  - HTTP 메서드로는 POST(create), PUT, PATCH(update), GET(read), DELETE(delete)를 제공하며 이를 통해 CRUD 작업을 합니다.
3. **자원의 표현(Representation Of Resource)**
  - client와 server간의 데이터를 주고받는 형태를 의미하며, 주로 JSON 또는 XML을 사용합니다.

### 2-3. REST 설계 원칙

1. **Server - Client (서버-클라이언트 구조)**
   - 클라이언트와 서버는 각각 독립적으로 설게 됩니다.
     - 클라이언트는 UI 및 사용자 경험에 집중합니다.
     - 서버는 API를 제공하고 데이터 저장 및 비즈니스 로직 처리를 담당합니다.
   - 이 원칙을 따르면 클라이언트와 서버가 서로 다른 기술 스택으로 개발, 배포, 유지보수 될 수 있습니다.
2. **Stateless (무상태)**
   - 서버는 클라이언트의 상태를 저장하지 않습니다.
   - 서버측에 사용자 인증, context(세션, 로그인 정보)등을 저장하지 않기 때문에 각 요청마다 인증 토큰 또는 필요한 데이터를 모두 저장해야 합니다.
3. **Cacheable (캐시 처리 가능)**
   - 클라이언트나 서버측 리소스는 **캐시 가능**해야 하며, 서버는 캐시 가능 여부를 명시해야 합니다.
4. **Layered System (계층 구조)**
   - REST 아키택처는 서버와 클라이언트가 직접 통신하지 않고 **프록시, 보안, 캐시, 로드 벨런서**등의 중간 계층을 통해 통신합니다.
     - **프록시**: 요청을 필터링 하거나 클라이언트의 IP를 숨깁니다.
     - **보안**: 데이터를 암호화 하거나 인증 과정을 처리합니다.
     - **캐시**: 자주 요청되는 데이터를 저장하여 서버에 부하를 줄입니다.
     - **로드 벨런서**: 다수의 서버에 요청을 분산시켜 트레픽 처리량을 증가시킵니다.
5. **Uniform Interface (일관된 인터페이스)**
   - 동일한 URI는 항상 동일한 데이터를 반환해야합니다.
   - 자원을 기반으로 설계 되어야하며, 특정 데이터가 하나의 고유한 URL을 통해 접근 되어야 합니다.
   - HTTP 표준 메서드(`GET`, `POST`, `PATCH`, `PATCH`, `DELETE`)를 사용하여 행위를 표현 합니다.
   - 응답 데이터는 적절한 크기를 유지하며, 클라이언트가 필요로 하는 모든 정보를 포함해야 하며 별도의 문서 없이도 의미를 파악할 수 있어야 합니다.
6. **Code on Demand(코드 온 디멘드, 선택사항)**
   - 이는 필수적인 사항은 아니며 필요할 경우 서버측에서 직접 클라이언트 코드를 전달하여 실행할 수 있습니다.
     - 서버에서 자바스크립트 파일을 클라이언트에 전달아혀 동적으로 실행 시킬 수 있습니다.

## 3. RESTful API란?

**RESTful API**는 REST 설계원칙을 모두 준수하여 설계된 API로, 자원(Resource)을 URI로 식별하고, HTTP 메서드를 통해 자원의 상태를 CRUD 방식으로 처리합니다.

### 3-1. RESTful API의 특징

1. **자원 중심 (Resource-Oriented)**  
   RESTful API는 자원(Resource)을 URI로 식별하며, 각 자원에 대해 독립적인 처리를 합니다. 예를 들어, `/users`, `/posts/{post_id}`와 같은 URI로 자원을 표현합니다.
2. **표준 HTTP 메서드 사용**
   - `GET`: 자원 조회
   - `POST`: 자원 생성
   - `PUT`: 자원 전체 수정
   - `PATCH`: 자원 부분 수정
   - `DELETE`: 자원 삭제
3. **Stateless (무상태성)**  
   RESTful API는 요청마다 서버에 상태를 저장하지 않으며, 각 요청은 독립적입니다.
4. **표준화된 응답 형식**  
   RESTful API는 응답 데이터를 JSON 형식으로 표준화하여, 클라이언트가 쉽게 데이터를 처리할 수 있도록 합니다.
5. **캐싱 (Caching)**  
   자주 요청되는 자원은 캐시 가능하도록 설정하여 성능을 개선하고 서버 부하를 줄입니다.
