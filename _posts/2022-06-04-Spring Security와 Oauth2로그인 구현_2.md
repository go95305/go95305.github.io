---
layout: single
title:  "Spring Security와 Oauth2 로그인(Google)-2"
category:
- Spring
- Spring Security
- Oauth2
---

<br/>

*해당 글은 Spring Securitydhk Oauth2 로그인(Google)-1에 이어서 작성한 글입니다.*

<br/>

전체코드는 <https://github.com/BestDevelopersIsHere/Board-Backend.git>에서 참고 가능합니다. 해당 글에는 전체 코드를 첨부하지 않겠습니다.

<br/>

### CustomOAuth2UserService
- Resource Server에서 받아온 구글 유저 정보가 DB에 존재하지않으면 User Entity만들어서 테이블에 적재한다.
- DB에 존재하면 UserPrincipal Dto형태로 return해준다.

~~~java
private OAuth2User processOAuth2User(OAuth2UserRequest oAuth2UserRequest, OAuth2User oAuth2User) {
        OAuth2UserInfo oAuth2UserInfo = OAuth2UserInfoFactory.getOAuth2UserInfo(oAuth2UserRequest.getClientRegistration().getRegistrationId(), oAuth2User.getAttributes());
        if (StringUtils.isEmpty(oAuth2UserInfo.getId())) {
            throw new OAuth2AuthenticationProcessingException("Id not found from OAuth2 provider");
        }

        Optional<User> userOptional = userRepository.findByProviderId(oAuth2UserInfo.getId()); // providerId로 조회
        User user;
        if (userOptional.isPresent()) {
            user = userOptional.get();
            if (!user.getProvider().equals(AuthProvider.valueOf(oAuth2UserRequest.getClientRegistration().getRegistrationId()))) {
                throw new OAuth2AuthenticationProcessingException("Looks like you're signed up with " +
                        user.getProvider() + " account. Please use your " + user.getProvider() +
                        " account to login.");
            }
            user = updateExistingUser(user, oAuth2UserInfo);
        } else {
            user = registerNewUser(oAuth2UserRequest, oAuth2UserInfo);
        }

        System.out.println("processOAuth2User : " + user);
        return UserPrincipal.create(user, oAuth2User.getAttributes());
    }
~~~

<br/>

### OAuth2AuthenticationSuccessHandler

- 앞서 SecurityConfig.java 에서 <br/> 
`.baseUri("/oauth2/authorize")
.authorizationRequestRepository(cookieAuthorizationRequestRepository())` 을 통해 Cookie값을 설정했었다. 거기엔 Client단 redirect_uri가 포함되어있는데 그 내용을 참조하여 jwt토큰을 보내줄 URL을 설정한다.

~~~java
protected String determineTargetUrl(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        Optional<String> redirectUri = CookieUtils.getCookie(request, REDIRECT_URI_PARAM_COOKIE_NAME)
                .map(Cookie::getValue);
        
        if (redirectUri.isPresent() && !isAuthorizedRedirectUri(redirectUri.get())) {
            throw new BadRequestException("Sorry! We've got an Unauthorized Redirect URI and can't proceed with the authentication");
        }
        String targetUrl = redirectUri.orElse("http://localhost:3000/redirect");
        System.out.println("targetUrl: " + targetUrl);
        String accessToken = tokenProvider.createToken(authentication, 0);
        System.out.println("access: " + accessToken);
        String refreshToken = tokenProvider.createToken(authentication, 1);

        logger.info("expired : " + new Date().getTime() + new Date(appProperties.getAuth().getTokenExpirationMsec()).getTime());
        return UriComponentsBuilder.fromUriString(targetUrl)
                .queryParam(ACCESS_TOKEN_NAME, accessToken)
                .queryParam(REFRESH_TOKEN_NAME, refreshToken)
                .queryParam(EXPIRED_TIME_NAME,
                        new Date().getTime() + new Date(appProperties.getAuth().getTokenExpirationMsec()).getTime())
                .build().toUriString();
    }
~~~

<br/>

위의 `isAuthorizedRedirectUri`는 application-oauth_yml에 작성된 redirect_uri를 참조하여 가져온다.

~~~yaml
oauth2:
    authorizedRedirectUris:
      - http://localhost:3000/redirect
~~~

<br/>

~~~java
private boolean isAuthorizedRedirectUri(String uri) {
        URI clientRedirectUri = URI.create(uri);

        return appProperties.getOauth2().getAuthorizedRedirectUris()
  .stream()
  .anyMatch(authorizedRedirectUri->{
  // Only validate host and port. Let the clients use different paths if they want to
  URI authorizedURI=URI.create(authorizedRedirectUri);
  return authorizedURI.getHost().equalsIgnoreCase(clientRedirectUri.getHost())
  &&authorizedURI.getPort()==clientRedirectUri.getPort();
  });
  }
~~~

<br/>

이제 직접 로그인과 JWT토큰이 만들어지는 과정을 화면을 통해 확인해보겠다.

<p align="center">
    <img src="/images/login.png" width="100%" class="image__border">
</p>

로그인을 눌러주면 구글 로그인 화면으로 이동된다.

<p align="center">
    <img src="/images/google_login.png" width="100%" class="image__border">
</p>

구글 계정을 선택한다.

<p align="center">
    <img src="/images/developer_mode.png" width="100%" class="image__border">
</p>

로그인 후, 개발자모드를 열어서 확인해보면 성공적으로 JWT 토큰이 만들어져서 Client쪽으로 보내진것을 확인할 수 있다.

<br/>

### 로그아웃?

결론부터 말하면 form login을 통한 커스텀 로그인이 아니고 구글, 카카오 등과 같은 oauth2 로그인을 구현한 경우 로그아웃이라는 개념이 따로없다. 물론 구글의 로그아웃은 있지만 현재 구글 로그인을 사용한
프로젝트의 로그아웃이 없다는 뜻이다. 관련 글을 Stacfk Overflow를 통해 확인할 수 있었다.

> *When you logout of your app, you're logging out of your app:
Here's where developers new to OAuth sometimes get a little confused... Google and Stack Overflow, Assembla, Vinesh's-very-cool-slick-webapp, are all different entities, and Google knows nothing about your account on Vinesh's cool webapp, and vice versa, aside from what's exposed via the API you're using to access profile information. When your user logs out, he or she isn't logging out of Google, he/she is logging out of your app, or Stack Overflow, or Assembla, or whatever web application used Google OAuth to authenticate the user. In fact, I can log out of all of my Google accounts and still be logged into Stack Overflow. Once your app knows who the user is, that person can log out of Google. Google is no longer needed. With that said, what you're asking to do is log the user out of a service that really doesn't belong to you. Think about it like this: As a user, how annoyed do you think I would be if I logged into 5 different services with my Google account, then the first time I logged out of one of them, I have to login to my Gmail account again because that app developer decided that, when I log out of his application, I should also be logged out of Google? That's going to get old really fast. In short, you really don't want to do this...*

우리가 Youtube를 이용하다가 로그아웃을 하면 Youtube만 로그아웃할 수 있는게 아니라 구글 자체를 로그아웃하는 것이 적절한 예시인거 같다.

### REFER TO

- <https://stackoverflow.com/questions/12909332/how-to-logout-of-an-application-where-i-used-oauth2-to-login-with-google>