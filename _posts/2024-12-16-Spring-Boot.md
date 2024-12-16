---
title: Spring VS Spring Boot
description: Spring Boot
author: 조준호
date: 2024-12-15 11:33:00 +0800
categories: [ java, Spring Boot ]
tags: [ java, cs ]
pin: false
---

## 1. Spring Framework란?

**Spring Framework**는 Java 기반의 어플리케이션 개발을 위한 오픈소스 Back-end 프레임워크입니다.

Spring이 사용되기 이전에는 **EJB(Enterprise Java Bean)** 방식을 사용해서 웹 어플리케이션을 개발했습니다.
EJB 방식은 특정 회사의 컨테이너가 없이는 기술 구현이 어렵고 프로그래밍 모델이 복잡해 지며, 자동화된 테스트가 거의 불가능한 치명적인 단점이 있섰습니다.

Spring은 이러한 EJB의 단점을 해결하며, 순수 자바 객체(POJO)만을 사용하여 복잡성을 제거한 단순하고 가벼운 코드로 어플리케이션 개발을 위한 프레임워크 입니다.

### 1-1. Spring Framework의 특징

Spring Framework의 주요 특징은 다음과 같습니다.

1. **IoC(제어의 역전)**  
   객체의 생성 및 생명주기를 개발자가 아닌 컨테이너가 관리하여 객체 간의 의존성을 효과적으로 해결합니다.
2. **DI(의존성 주입)**  
   스프링은 의존성 주입을 통해 객체간의 관계를 정의 하며, 이를 통해 어플리케이션의 결합도를 낮추고 유연성과 테스트 용이성을 향상 시킵니다.
3. **AOP(관점 지향 프로그래밍) 지원**  
   어플리케이션의 핵심 기능과 부가적인 기능을 분리하여 모듈화 할 수 있습니다.
4. **웹 어플리케이션 지원**  
   유연하고 확장성 있는 웹 어플리케이션을 개발할 수 있는 **MVC(Model-View-Controller)** 아키텍처 패턴을 지원합니다.

### 1-2. Spring Boot란?

Spring Boot는 기존 Spring Framework를 보다 편하게 사용할 수 있게 만든 프레임워크 입니다. Spring Boot에서는 개발자가 설정 파일을
작성할 필요 없이 프로젝트의 설정과 라이브러리 의존성을 자동으로 처리해주는 **Auto Configuration**기능이 있습니다.

### 1-3. Spring Boot의 특징 및 Spring Framework와의 차이점

1. **Auto Configuration**  
   어플리케이션에 필요한 설정을 자동으로 구성해 줍니다.
   - **Spring Framework**: 개발자가 수많은 XML이나 Java Config 파일을 작성해야 했습니다.
   - **Spring Boot**: `@EnableAutoConfiguration`을 사용해 필요한 설정을 자동으로 구성해줍니다. 이를 통해 설정 작업을 대폭 줄이고, 기본적인 설정은 Spring Boot가 알아서 처리합니다.
2. **내장 웹 서버 제공**  
   Spring Boot는 Tomcat, Jetty, Undertow 등의 웹 서버를 내장하고 있기 때문에 별도의 서버 설치 없이 프로그램을 실행 시킬 수 있습니다.
   - **Spring Framework**: 별도의 Tomcat, Jetty, 또는 다른 WAS(Web Application Server)를 설치하고 수동으로 애플리케이션을 배포해야 했습니다.
   - **Spring Boot**: Tomcat, Jetty, Undertow 같은 웹 서버를 내장하고 있어 별도의 WAS 설치 없이 애플리케이션을 독립적으로 실행할 수 있습니다.
3. **Starter Dependencies**  
   관련 라이브러리들을 묶어서 제공하여 의존성 관리를 간소화합니다. 예를 들어 spring-boot-starter-web은 웹 애플리케이션 개발에 필요한 모든 라이브러리를
   포함합니다.
   - **Spring Framework**: 필요한 라이브러리를 개별적으로 추가하고 버전을 직접 관리해야 했습니다.
   - **Spring Boot**: 스타터 의존성을 제공하여 개발자가 필요한 의존성을 한 번에 추가할 수 있습니다.
4. **독립 실행형 애플리케이션 지원**  
   Spring Boot는 애플리케이션을 실행 가능한 JAR 파일로 패키징할 수 있는 기능을 제공합니다.
   - **Spring Framework**: 애플리케이션을 WAR 파일로 빌드한 후 외부 WAS에 배포해야 했습니다.
   - **Spring Boot**: 실행 가능한 JAR 파일을 생성할 수 있어 별도의 WAS 없이 애플리케이션을 바로 실행할 수 있습니다.  

