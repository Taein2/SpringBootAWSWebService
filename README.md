# SpringBootAWSWebService
## [ 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 ]


### CHAPTER 5 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현
스프링 시큐리티를 구현하는 이유
스프링 애플리케이션에서의 보안을 위한 표준
막강한 인증(Authentication)과 인가(Authorization 혹은 권한 부여) 기능을 가진 프레임 워크
다양한 요구사항을 손쉽게 추가하고 변경할 수 있는 확장성을 가짐

소셜 로그인을 구현하는 이유?
직접 로그인 구현 시 배보다 배꼽이 커지는 경우가 많다
OAuth로 로그인 구현을 소셜에 맡기고 서비스 개발에 집중

spring-security-oauth2-autoconfigure 라이브러리
스프링 부트 1.5에서 쓰던 설정을 그대로 사용 가능

Spring Security Oauth2 Client 라이브러리 사용
- 스프링 팀에서 1.5에서 사용되던 spring-security-oauth 프로젝트는 유지 상태로 결정했으며 더는 신규 기능이 추가되지 않고 버그 수정 정도의 기능만 추가될 예정, 신규 기능은 새 oauth2 라이브러리에서만 지원하겠다고 선언
- 스프링 부트용 라이브러리(starter) 출시
- 기존에 사용되던 방식은 확장 포인트가 적절하게 오픈되어 있지 않아 직접 상속하거나 오버라이딩 해야 하고 신규 라이브러리의 경우 확장 포인트를 고려해서 설계된 상태

spring-security-oauth2-autoconfigure 라이브러리 썼는지 , application.properties 혹은 application.yml 정보가 차이가 있는지 비교(1.5에서는 url 주소 모두 명시, 2.0방식은 client 인증 정보만 입력)
1.5에서 2.0으로 넘어오면서 직접 입력한 값들은 enum 으로 대체
CommonOAuth2Provider라는 enum이 새롭게 추가되어 구글,깃허브,페이스북,옥타의 기본 설정값을 제공(이외의 네이버,카카오 등은 직접 추가해주어야 함)



클라이언트 ID와 클라이언트 보안 비밀을 application.properties에 등록
spring.security.oauth2.client.registration.google.client-id=클라이언트ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트보안비밀
spring.security.oauth2.client.registration.google.scope=profile,email
- 많은 예제에서 scope를 별도로 등록하고 있지 않음
- 기본값이 openid, profile, email임
- 강제로 profile, email을 등록한 이유는 openid라는 scope가 있으면 Open Id Provider로 인식
- OpenId Provider인 서비스(구글)와 그렇지 않은 서비스(네이버/카카오 등)로 나눠서 각각 OAuth2Service를 만들어야 함
- 하나의 OAuth2Service로 사용하기 위해 일부로 openid scope를 빼고 등록
profile = xxx 라는 호출방식을 사용하면 해당 properties의 설정들을 가져올 수 있음
이 책에서는 application.properties에서 application-oauth.properties를 포함하도록 구성
spring.profiles.include=oauth


클라이언트ID와 보안 비밀의 보안을 위해 .gitignore에 등록
application-oauth.properties
