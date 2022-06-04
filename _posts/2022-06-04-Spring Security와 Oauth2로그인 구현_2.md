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
                .anyMatch(authorizedRedirectUri -> {
                    // Only validate host and port. Let the clients use different paths if they want to
                    URI authorizedURI = URI.create(authorizedRedirectUri);
                    return authorizedURI.getHost().equalsIgnoreCase(clientRedirectUri.getHost())
                            && authorizedURI.getPort() == clientRedirectUri.getPort();
                });
    }
~~~


