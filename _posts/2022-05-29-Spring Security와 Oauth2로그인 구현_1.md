---
layout: single
title:  "Spring Security와 Oauth2 로그인(Google)-1"
category:
- Spring
- Spring Security
- Oauth2
---

<br/>

#### 패키지 구조
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

#### 흐름도
<p align="center">
    <img src="/images/oauth2_flow.png" width="100%" class="image__border">
</p>
<center>https://jyami.tistory.com/121</center><br>

<br/>

<br/>

### build.gradle

~~~java
implementation 'org.springframework.boot:spring-boot-starter-security'
annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
implementation group: 'io.jsonwebtoken', name: 'jjwt', version: '0.9.1'
~~~
`annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"` 설정을 추가해주지 않을 경우 `application-oauth.yml`에서 설정을 가져와서 redirectUri검증 작업의 수행이불가능하다.
<p align="center">
    <img src="/images/gradle_oauth2_client.png" width="100%" class="image__border">
</p>

<br/>

### application-oauth.yml
`authorizedRedirctUris`는 아래에서 확인하겠지만, Client측에서 설정해준 redirect_uri와 같은지를 검증하는데 사용.<br/>
`authorizedRedirctUris`는 서버에서 JWT토큰을 받을 URL인데 잘못된 Client(해커)가 Client단에서의 요청을 자신이 원하는 redirect_uri로 설정하여 개인정보 탈취의 위험성이 있기때문에 서버단에서 다시한번 검증 수행이 필요하다.
~~~yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: {client-id}
            client-secret: {client-secret}
            redirectUri: "{baseUrl}/oauth2/callback/{registrationId}"
            scope: profile,email

app:
  auth:
    tokenSecret: {token Secret}
    tokenExpirationMsec: 3600000
    refreshTokenExpirationMesc: 864000000
  oauth2:
    authorizedRedirectUris:
      - http://localhost:3000/redirect
~~~



### 2. Spring Security 설정
<br/>

#### 2-1 SecurityConfig
- `.authorizeRequest().antMatchers(~~):`antMatchers에 포함되는 요청들은 모두 인증과정이 필요.
- `.authorizationEndpoint()`: Client단에서 보낸 GET요청을 토대로 인증을 진행할 수 있는 화면으로 redirect한다. (`https://accounts.google.com/o/oauth2/auth`이 인증요청을 받아들이는 URL이므로 이 URL을 Client단에 보여준다)
- `.redirectionEndpoint()`<br/>
   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.baseUri("/oauth2/callback/*")`: 인증 서버에서 받아온 Authorization Code를 가져와 보내줄 URL설정(이정보들을 그대로 Client에게 보내주지않고 Server단에서 JWT토큰을 생성하고 그 토큰을 보내준다.)
- `.userInfoEndpoint()`<br/>
   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`.userService(customOAuth2UserService)`: Google에서 사용자 정보를 가져와서 DB에 유저 정보를 생성하거나 업데이트한다. 그리고 authorization code를 통해 Access token을 발급받는다.
- `.successHandler(oAuth2AuthenticationSuccessHandler)`: Access Token을 성공적으로 받아왔으면 실행
- `.failureHandler(oAuth2AuthenticationFailureHandler)`:  Authorization code 받가오기 실패했으면 실행

~~~java
package com.board.config.auth;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;

@Configuration
@RequiredArgsConstructor
@EnableGlobalMethodSecurity(
        securedEnabled = true,
        jsr250Enabled = true,
        prePostEnabled = true
)
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CustomOAuth2UserService customOAuth2UserService;
    private final OAuth2AuthenticationSuccessHandler oAuth2AuthenticationSuccessHandler;
    private final OAuth2AuthenticationFailureHandler oAuth2AuthenticationFailureHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .cors()
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .csrf().disable()
                .formLogin().disable()
                .httpBasic().disable()

                .authorizeRequests()
                .antMatchers("/oauth2/authorize/**", "/oauth2/callback/**", "/user/refresh", "/user/logout", "/user/home")
                .permitAll()
                .anyRequest().authenticated()
                .and()
                .logout()
                .logoutSuccessUrl("/")
                .and()
                .oauth2Login()
                .authorizationEndpoint()
                .baseUri("/oauth2/authorize")// Client가 queryString으로 보내는 GET요청. oauth default uri , ex) {baseUrl}/oauth2/authorize/{provider}?redirect_uri=~
                .and()
                .redirectionEndpoint()
                .baseUri("/oauth2/callback/*")
                .and()
                .userInfoEndpoint()
                .userService(customOAuth2UserService)
                .and()
                .successHandler(oAuth2AuthenticationSuccessHandler) // 성공적으로 authorization code를 발급받을경우
                .failureHandler(oAuth2AuthenticationFailureHandler);
    }

}

~~~

<br/>

이번 글 에서는 Spring Security에 대한 설정은 여기까지만 보고 다음 글부터는 각 Service, Handler, Repository등 세부적인 설정을 확인하겠다. 

<br/>

### REFER TO
- <https://deeplify.dev/back-end/spring/oauth2-social-login#%EC%A0%84%EC%B2%B4-%EC%8B%9C%ED%80%80%EC%8A%A4-%EB%8B%A4%EC%9D%B4%EC%96%B4%EA%B7%B8%EB%9E%A8>
- <https://jyami.tistory.com/121>


