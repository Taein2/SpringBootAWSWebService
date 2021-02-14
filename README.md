# SpringBootAWSWebService
## [ 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 ]


## CHAPTER 5 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현<br>
### 스프링 시큐리티를 구현하는 이유<br>
스프링 애플리케이션에서의 보안을 위한 표준<br>
막강한 인증(Authentication)과 인가(Authorization 혹은 권한 부여) 기능을 가진 프레임 워크<br>
다양한 요구사항을 손쉽게 추가하고 변경할 수 있는 확장성을 가짐<br>

### 소셜 로그인을 구현하는 이유?<br>
직접 로그인 구현 시 배보다 배꼽이 커지는 경우가 많다<br>
OAuth로 로그인 구현을 소셜에 맡기고 서비스 개발에 집중<br>

### spring-security-oauth2-autoconfigure 라이브러리<br>
스프링 부트 1.5에서 쓰던 설정을 그대로 사용 가능<br>

### Spring Security Oauth2 Client 라이브러리 사용<br>
- 스프링 팀에서 1.5에서 사용되던 spring-security-oauth 프로젝트는 유지 상태로 결정했으며 더는 신규 기능이 추가되지 않고 버그 수정 정도의 기능만 추가될 예정, 신규 기능은 새 oauth2 라이브러리에서만 지원하겠다고 선언
- 스프링 부트용 라이브러리(starter) 출시
- 기존에 사용되던 방식은 확장 포인트가 적절하게 오픈되어 있지 않아 직접 상속하거나 오버라이딩 해야 하고 신규 라이브러리의 경우 확장 포인트를 고려해서 설계된 상태

spring-security-oauth2-autoconfigure 라이브러리 썼는지 , application.properties 혹은 application.yml 정보가 차이가 있는지 비교(1.5에서는 url 주소 모두 명시, 2.0방식은 client 인증 정보만 입력)<br>
1.5에서 2.0으로 넘어오면서 직접 입력한 값들은 enum 으로 대체<br>
CommonOAuth2Provider라는 enum이 새롭게 추가되어 구글,깃허브,페이스북,옥타의 기본 설정값을 제공(이외의 네이버,카카오 등은 직접 추가해주어야 함)<br>



### 클라이언트 ID와 클라이언트 보안 비밀을 application.properties에 등록<br>
spring.security.oauth2.client.registration.google.client-id=클라이언트ID<br>
spring.security.oauth2.client.registration.google.client-secret=클라이언트보안비밀<br>
spring.security.oauth2.client.registration.google.scope=profile,email<br>
- 많은 예제에서 scope를 별도로 등록하고 있지 않음
- 기본값이 openid, profile, email임
- 강제로 profile, email을 등록한 이유는 openid라는 scope가 있으면 Open Id Provider로 인식
- OpenId Provider인 서비스(구글)와 그렇지 않은 서비스(네이버/카카오 등)로 나눠서 각각 OAuth2Service를 만들어야 함
- 하나의 OAuth2Service로 사용하기 위해 일부로 openid scope를 빼고 등록


profile = xxx 라는 호출방식을 사용하면 해당 properties의 설정들을 가져올 수 있음<br>
이 책에서는 application.properties에서 application-oauth.properties를 포함하도록 구성<br>
spring.profiles.include=oauth<br>


### 클라이언트ID와 보안 비밀의 보안을 위해 .gitignore에 등록
application-oauth.properties

### .gitignore에 추가했는데도 커밋 목록에 나올 시
.gitignore가 제대로 작동되지 않아서 ignore처리된 파일이 자꾸 changes에 나올때가 있다.<br>
git의 캐시가 문제가 되는거라 아래 명령어로 캐시 내용을 전부 삭제후 다시 add All해서 커밋하면 된다.<br>
git rm -r --cached .<br>
git add .<br>
git commit -m "fixed untracked files"<br>

### 구글 로그인 연동하기
domain 아래 user 패키지 생성<br>
- 사용자 정보를 담당할 도메인 User class
- 사용자의 권한을 담당할 Enum 클래스 Role(스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야 함, EX)ROLE_GUEST, ROLE_USER)
- User의 CRUD를 담당할 interface UserRepository 

### 스프링 시큐리티 설정
build.gradle에 스프링 시큐리티 관련 의존성 추가
compile('org.springframework.boot:spring-boot-starter-oauth2-client') 소셜 기능 구현 시 필요한 의존성

### OAuth 라이브러리를 이용한 소셜 로그인 설정 코드 작성 (config.auth 패키지)
- SecurityConfig 클래스 생성
- CustomOAuth2UserService 클래스 생성 (구글 로그인 이후 가져온 사용자의 정보(email,name,picture 등)를 기반으로 가입 및 정보수정, 세션 저장 등의 기능 지원, 유저 정보 업데이트시 User 엔티티에 반영) 

### config.auth 에 dto 패키지를 만듬
- OAuthAttributes 클래스 생성 
- SessionUser 클래스 생성 (인증된 사용자 정보 name, email, picture만 선언)

### User 클래스를 사용하면 안되는 이유
User클래스 사용 시 직렬화 구현하지 않았다는 에러가 뜸<br>
이유는 User클래스가 엔티티이기 때문, 엔티티는 언제 다른 엔티티와 관계가 형성될지 모름<br>
따라서 성능 이슈, 부수 효과가 발생할 확률이 높기에 직렬화 기능을 가진 Dto를 하나 추가하여 운영 및 유지 보수에 도움을 줌<br>

### 로그인 테스트
index.mustache에 로그인 버튼과 로그인 성공 시 사용자 이름을 보여주는 html 코드 추가<br>
index.mustache에서 userName을 사용할 수 있게 IndexController에서 userName을 model에 저장하는 코드 추가

사용자 권한 GUEST 에서는 posts기능 사용 불가<br>
h2-console에서 사용자의 role을 USER로 변경하는 쿼리 작성<br>
update user set role = 'USER';<br>

### 어노테이션 기반으로 개선
같은 코드가 반복되는 경우는 수정이 필요한 경우 일일이 찾아가서 수정해야하므로 어노테이션을 이용하여 개선<br>
- config.auth패키지에 @LoginUser 어노테이션 생성
- LoginUserArgumentResolver 생성 (조건에 맞는 경우의 메소드가 있으면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터를 넘김)
- config 패키지에 WebConfig 클래스 작성 (LoginUserArgumentResolver가 스프링에서 인식될 수 있도록 WebMvcConfigurer에 추가)
- IndexController 클래스에서 반복되는 부분들을 @LoginUser로 개선

### 세션 저장소로 데이터베이스 사용하기
세션이 내장 톰캣의 메모리에 저장되기 때문이다.<br>
기본적으로 WAS(Web Application Server)의 메모리에 저장되고 호출된다. 하여, 내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에서는 항상 초기화됌<br>
2대 이상 서버에서 서비스하고 있다면 톰캣마다 세션 동기화 설정을 해야 함<br>
실제 현업에서의 세션 저장소 (3중 택 1)<br>
- 톰캣 세션을 사용
- MySQL과 같은 데이터베이스를 세션 저장소로 사용
- Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용

책에서는 두 번째 방식인 데이터베이스를 세션 저장소로 사용 (설정이 간단하고 사용자가 많은 서비스가 아니며 비용 절감을 위해서)<br>
build.gradle에 spring-session-jdbc 의존성 추가 <br>
compile('org.springframework.session:spring-session-jdbc')<br>

application.properties에 세션 저장소를 jdbc로 선택하도록 코드 추가<br>
spring.session.store-type=jdbc<br>

h2-console로 접속하면 SPRING_SESSION, SPRING_SESSION_ATTRIBUTES가 생성됨 (JPA로 인해 세션 테이블이 자동 생성)<br>



