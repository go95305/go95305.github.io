---
layout: single
title:  "Spring Security와 Oauth2 로그인(Google)"
category:
- Spring
- Spring Security
- Oauth2
---

<br/>

### 1. Architecture
<br/>

#### 1-1 패키지 구조
~~~text
-- action-in-blog.iml
|-- mvnw
|-- mvnw.cmd
|-- pom.xml
`-- src
    |-- main
    |   |-- java
    |   |   `-- com
    |   |       `-- board
    |   |           |-- config
    |   |           |   `-- auth
    |   |           |      |-- AppProperties.java
    |   |           |      |-- CookieUtils.java
    |   |           |      |-- CustomOAuth2UserService.java
    |   |           |      |-- CustomUserDetailsService.java
    |   |           |      |-- GoogleOAuth2UserInfo.java
    |   |           |      |-- HttpCookieOAuth2AuthorizationRequestRepository.java
    |   |           |      |-- OAuth2AuthenticationFailureHandler.java
    |   |           |      |-- OAuth2AuthenticationSuccessHandler.java
    |   |           |      |-- OAuth2UserInfo.java
    |   |           |      |-- OAuth2UserInfoFactory.java
    |   |           |      |-- RandomStringGenerator.java
    |   |           |      |-- SecurityConfig.java
    |   |           |      |-- TokenAuthenticationFilter.java
    |   |           |      |-- TokenProvider.java
    |   |           |      |-- TokenValue.java
    |   |           |      |-- UserPrincipal.java
    |   |           |-- domain
    |   |           |   `-- entity
    |   |           |      |-- AuthProvider.java
    |   |           |      |-- RoleType.java
    |   |           |      |-- User.java
    |   |           |   `-- repository
    |   |           |      |-- UserRepository.java    
    |   |           `-- exception
    |   |               `-- entity
    |   |                  |-- BadRequestException.java
    |   |                  |-- OAuth2AuthenticationProcessingException.java
    |   |                  |-- ResourceNotFoundException.java
    |   `-- resources
    |       `-- application.yml
    |       `-- application-auth.yml
~~~

<br/>

#### 1-2 흐름도
<p align="center">
    <img src="/images/oauth2_flow.png" width="100%" class="image__border">
</p>

<br/>

### 2. Spring Security 설정
<br/>






