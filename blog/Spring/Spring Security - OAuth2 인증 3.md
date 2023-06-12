태그: #Spring 
연결문서: [Spring Security - OAuth2 인증 4](Spring%20Security%20-%20OAuth2%20인증%204.md)

# Spring Security에서의 OAuth2 인증

## OAuth2 샘플 애플리케이션 구현
- 해당 샘플 애플리케이션은 서버 측에서 HTML을 렌더링 해주는 SSR(Server Side Rendering) 방식의 애플리케이션을 구현함

### 의존성 추가
- OAuth2 인증을 사용하기 위해 제일 먼저 해야 할 작업은 OAuth2에 대한 의존성을 추가하는 것
<br>

```
...

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'    // (1)
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	implementation 'org.springframework.boot:spring-boot-starter-security'     // (2)
	implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'  // (3)
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	implementation 'org.mapstruct:mapstruct:1.5.2.Final'
	annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.2.Final'
	implementation 'org.springframework.boot:spring-boot-starter-mail'
	implementation 'com.google.code.gson:gson'
}

...
```

<br>

- (1)에서는 HTML 화면을 구성하기 위한 템플릿인 타임리프(Thymeleaf)를 추가함
- OAuth2샘플 애플리케이션은 Spring Security 기반의 애플리케이션이므로 (2)와 같이 spring-boot-starter-security를 추가함
- OAuth2샘플 애플리케이션은 구글의 OAuth2 시스템을 이용하는 OAuth2 클라이언트이므로 클라이언트로써의 역할을 하기 위해 spring-boot-starter-oauth2-client를 추가함

<br>

### 보호된 웹 페이지
- SSR 방식의 웹 애플리케이션은 HTML로 렌더링 되는 페이지가 존재함
- OAuth2 샘플 애플리케이션은 OAuth2 인증을 통해 보호되는 아주 간단한 HTML 페이지를 포함하고 있음
<br>

![](image_36.png)

<br>

- 위 사진은 OAuth2로 로그인 인증에 성공하지 않으면 웹 브라우저에서 확인할 수 없는 HTML 화면
- 이 화면이 OAuth 2를 통해 잘 보호되느냐 그렇지 않으냐가 중요함

<br>

```html
// sample-oauth2.html(src/main/resources/templates)
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Welcome to Sample OAuth 2.0</title>
</head>
<body>
    <div style="text-align: center"><h2>Welcome to Sample OAuth 2.0!!</h2></div>
</body>
</html>
```

<br>

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloHomeController {
    @GetMapping("/sample-oauth2")
    public String home() {
        return "sample-oauth2";
    }
}
```

- sample-oauth2 화면에 대한 뷰를 리턴하는 HelloHomeController의 코드으로, SSR(Server Side Rendering) 방식의 핸들러(Controller) 메서드의 리턴 타입이 String이면 뷰 이름(sample-oauth2)을 리턴하며, 최종적으로 sample-oauth2.html을 웹브라우저로 전송함

<br>

### OAuth 2 인증을 위한 SecurityConfiguration 설정

#### 1. Spring Boot의 자동 구성을 이용한 OAuth2 인증 설정
- Spring Security에서 OAuth2 인증을 가장 간단하게 사용할 수 있는 방법은 바로 Spring Boot의 자동 구성을 이용한 방법
- OAuth 2에 대한 자세한 설정 방법을 몰라도 Spring Boot의 자동 구성을 통해 대부분의 설정이 자동으로 구성됨
<br>

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .formLogin().disable()
            .httpBasic().disable()
            .authorizeHttpRequests(authorize -> authorize    // (1)
                    .anyRequest().authenticated()
            )
            .oauth2Login(withDefaults());    // (2)
        return http.build();
    }
}
```

<br>

- (1)과 같이 인증된 request에 대해서만 접근을 허용하도록 authorize.anyRequest().authenticated()를 추가함
- (2)와 같이 .oauth2Login(withDefaults()) 를 추가해서 OAuth 2 로그인 인증을 활성화함
- 상태에서 애플리케이션을 실행하면 아래와 같은 에러가 발생함
<br>

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of method setFilterChains in org.springframework.security.config.annotation.web.configuration.WebSecurityConfiguration required a bean of type 'org.springframework.security.oauth2.client.registration.ClientRegistrationRepository' that could not be found.

Action:

Consider defining a bean of type 'org.springframework.security.oauth2.client.registration.ClientRegistrationRepository' in your configuration.
```

<br>

- 에러 로그를 유추해 보면 ClientRegistrationRepository라는 Bean이 없으니 SecurityConfiguration에 추가하라는 의미
- 이용해야 할 OAuth 2 시스템에 대한 클라이언트 ID와 클라이언트 보안 비밀번호(Secret)를 설정하지 않았기 때문에 발생하는 에러

<br>

### OAuth2 클라이언트 등록 정보 추가

```java
// application.yml
spring:
  h2:
    console:
      enabled: true
      path: /h2
  datasource:
    url: jdbc:h2:mem:test
  jpa:
    hibernate:
      ddl-auto: create  
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  sql:
    init:
      data-locations: classpath*:db/h2/data.sql
  security:
    oauth2:
      client:
        registration:
          google:
            clientId: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx           # (1)
            clientSecret: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx  # (2)

...
```

<br>

- (1)은 OAuth 2 클라이언트 생성 시, 안전하게 보관하고 있는 클라이언트 ID
- (2)는 클라이언트 보안 비밀번호 즉, 클라이언트의 Secret
- 이제 애플리케이션을 실행하면 정상적으로 실행이 됨
- 만약 OAuth 2의 설정을 정상적으로 구성했는데도 불구하고, 애플리케이션을 재실행해도 특별한 에러 메시지 없이 구글 로그인 화면이 뜨지 않으면, Google 계정 관리 -> 보안 -> 다른 사이트 로그인 수단에서 Google 계정을 통한 로그인을 선택한 후, 애플리케이션의 액세스 권한 삭제 버튼을 클릭해 애플리케이션의 권한을 제거해 주어야 함
- 민감한 정보의 경우 application.yml 파일에 그대로 노출하는 것은 보안상 바람직하지 않음
- 만약 실무에서 OAuth 2 클라이언트 ID와 Secret 같은 민감한 정보를 설정한다면 OS의 시스템 환경 변수에 설정하거나 또는 application.yml 파일에 구성하는 프로퍼티 정보를 애플리케이션 외부의 안전한 경로에 위치시키는 등의 방식으로 사용해야함

<br>

#### 2. Configuration을 통한 OAuth2 인증 설정
- Spring Boot에서는 자동 구성을 통한 OAuth2 인증 설정뿐만 아니라 Configuration을 통해 Bean을 등록함으로써 OAuth2의 인증을 설정할 수 있음
<br>

```java
// SecurityConfiguration
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.oauth2.client.CommonOAuth2Provider;
import org.springframework.security.oauth2.client.registration.ClientRegistration;
import org.springframework.security.oauth2.client.registration.ClientRegistrationRepository;
import org.springframework.security.oauth2.client.registration.InMemoryClientRegistrationRepository;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {
    @Value("${spring.security.oauth2.client.registration.google.clientId}")  // (1)
    private String clientId;

    @Value("${spring.security.oauth2.client.registration.google.clientSecret}") // (2)
    private String clientSecret;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .formLogin().disable()
            .httpBasic().disable()
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().authenticated()
            )
            .oauth2Login(withDefaults());
        return http.build();
    }

    // (3)
    @Bean
    public ClientRegistrationRepository clientRegistrationRepository() {
        var clientRegistration = clientRegistration();    // (3-1)

        return new InMemoryClientRegistrationRepository(clientRegistration);   // (3-2)
    }

    // (4)
    private ClientRegistration clientRegistration() {
        // (4-1)
        return CommonOAuth2Provider
                .GOOGLE
                .getBuilder("google")
                .clientId(clientId)
                .clientSecret(clientSecret)
                .build();
    }
}
```

- (1)과 (2)에서는 application.yml 파일에 설정되어 있는 구글의 Client ID와 Secret을 로드함
- (3)에서는 ClientRegistrationRepository를 Bean으로 등록함
    - ClientRegistrationRepository는 ClientRegistration을 저장하기 위한 Responsitory
    - Spring Boot의 자동 구성 기능을 이용할 경우, application.yml 파일에 설정된 구글의 Client ID와 Secret 정보를 기반으로 우리 눈에는 보이지 않지만 내부적으로 ClientRegistrationRepository Bean이 생성되는 반면, 지금은 우리가 Configuration을 통해 ClientRegistrationRepository Bean을 직접 등록하고 있음
    - (3-1)에서는 private 메서드인 clientRegistration()을 호출해서 ClientRegistration 인스턴스를 리턴 받음
    - (3-2)에서는 ClientRegistrationRepository 인터페이스의 구현 클래스인InMemoryClientRegistrationRepository의 인스턴스를 생성함
        - InMemoryClientRegistrationRepository는 ClientRegistration을 메모리에 저장함
- (4)는 ClientRegistration 인스턴스를 생성하는 private 메서드
    - (4-1)을 보면 Spring Security에서는 CommonOAuth2Provider라는 enum을 제공하는데 CommonOAuth2Provider 는 내부적으로 Builder 패턴을 이용해 ClientRegistration 인스턴스를 제공하는 역할을 함
    - ClientRegistration은 OAuth2 Client에 대한 등록 정보를 표현하는 객체로 구글의 API 콘솔에서 등록했던 OAuth Client ID에 대한 정보(Client ID, Secret 등)가 포함되어 있다고 생각하면 됨

<br>

#### 3. Spring Boot 자동 구성
- 만약 application.yml에 Client ID와 Client Secret만 추가하고, SecurityConfiguration 클래스가 존재하지 않아도 웹 브라우저에서 구글의 로그인 인증 화면은 정상적으로 표시되고, OAuth2 인증이 정상 동작하는 것을 확인할 수 있음
- build.gradle dependences {…}에 implementation 'org.springframework.boot:spring-boot-starter-oauth2-client' 를 추가하기만 하면 Spring Boot이 자동 구성을 통해 내부적으로 알아서 OAuth2의 기능을 활성화함
- 무조건적인 자동 구성보다는 명시적으로 특정 설정을 선언해서 유지보수 용이하고 가독성 있는 코드를 구성하는 것 역시 중요함

<br>

### 인증된 Authentication 정보 확인
- OAuth 2 인증이 성공적으로 수행되었는지 최종적으로 확인해 보는 것
<br>

#### 1. SecurityContext를 이용하는 방법

```java
// HomeController
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SampleHomeController {
    @GetMapping("/sample-oauth2")
    public String home() {
        var oAuth2User = (OAuth2User)SecurityContextHolder.getContext().getAuthentication().getPrincipal(); // (1)
        System.out.println(oAuth2User.getAttributes().get("email"));   // (2)
        return "sample-oauth2";
    }
}
```

<br>

- (1)에서는 SecurityContext에서 인증된 Authentication 객체를 통해 Principal 객체를 얻음
    - OAuth2로 로그인 인증을 수행했으므로 SecurityContext에 저장된 Principal은 OAuth2User 객체로 캐스팅할 수 있음
- (2)에서는 OAuth2User 객체에 저장되어 있는 사용자의 정보 중에서 getAttributes() 메서드를 통해 사용자의 이메일 정보를 얻고 있음
- 웹 브라우저에서 home() 핸들러 메서드로 request를 전송하면 OAuth 2 인증에 성공하기 전까지는 home() 핸들러 메서드가 호출되지 않음

<br>

#### 2. Authentication 객체를 핸들러 메서드 파라미터로 전달받는 방법

```java
// HomeController
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.client.OAuth2AuthorizedClientService;
import org.springframework.security.oauth2.core.OAuth2AccessToken;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SampleHomeController {
    @GetMapping("/sample-oauth2")
    public String home(Authentication authentication) {    // (1)
        var oAuth2User = (OAuth2User)authentication.getPrincipal();
        System.out.println(oAuth2User);
        System.out.println("User's email in Google: " + oAuth2User.getAttributes().get("email"));

        return "sample-oauth2";
    }
}
```

<br>

- (1)과 같이 인증된 Authenction을 핸들러 메서드의 파라미터로 전달받고 있음

<br>

#### 3. OAuth2User를 파라미터로 전달받는 방법

```java
// HomeController
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SampleHomeController {
    @GetMapping("/sample-oauth2")
    public String home(@AuthenticationPrincipal OAuth2User oAuth2User) {  // (1)
        System.out.println("User's email in Google: " + oAuth2User.getAttributes().get("email"));

        return "sample-oauth2";
    }
}
```

<br>

- (1)과 같이 @AuthenticationPrincipal 애너테이션을 이용해 OAuth2User 객체를 파라미터로 직접 전달받고 있음

<br>

### Authorization Server로부터 전달받은 Access Token 확인
- 구글의 OAuth2 인증이 성공적으로 수행되면 내부적으로 리소스 서버에 접근할 때 사용되는 Access Token을 전달받게 됨
- OAuth 2 인증 이후, 전달받은 Access Token의 정보를 확인할 수 있음

<br>

#### 1. OAuth2AuthorizedClientService를 DI 받는 방법

```java
// HomeController
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.client.OAuth2AuthorizedClientService;
import org.springframework.security.oauth2.core.OAuth2AccessToken;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SampleHomeController {
    private final OAuth2AuthorizedClientService authorizedClientService;

    // (1)
    public SampleHomeController(OAuth2AuthorizedClientService authorizedClientService) {
        this.authorizedClientService = authorizedClientService;
    }

    @GetMapping("/Sample-oauth2")
    public String home(Authentication authentication) {
        var authorizedClient = authorizedClientService.loadAuthorizedClient("google", authentication.getName()); // (2)

        // (3)
        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();
        System.out.println("Access Token Value: " + accessToken.getTokenValue());  // (3-1)
        System.out.println("Access Token Type: " + accessToken.getTokenType().getValue());  // (3-2)
        System.out.println("Access Token Scopes: " + accessToken.getScopes());       // (3-3)
        System.out.println("Access Token Issued At: " + accessToken.getIssuedAt());    // (3-4)
        System.out.println("Access Token Expires At: " + accessToken.getExpiresAt());  // (3-5)

        return "Sample-oauth2";
    }
}
```

<br>

- OAuth2AuthorizedClientService는 권한을 부여받은 Client(이하 OAuth2AuthorizedClient)를 관리하는 역할을 하는데 OAuth2AuthorizedClientService를 이용해서 OAuth2AuthorizedClient 가 보유하고 있는 Access Token에 접근할 수 있기 때문에 OAuth2AuthorizedClientService를 (1)과 같이 DI 받음
- (2)에서는 OAuth2AuthorizedClientService의 loadAuthorizedClient("google", authentication.getName())를 이용해 OAuth2AuthorizedClient 객체를 로드함
    - loadAuthorizedClient()를 호출하면 내부적으로 OAuth2AuthorizedClientRepository에서 OAuth2AuthorizedClient 를 조회함
- (3)에서는 authorizedClient.getAccessToken()를 이용해 OAuth2AccessToken 객체를 얻음
    - (3-1)에서는 Access Token의 문자열을 출력함
    - (3-2)에서는 Token의 타입을 출력함
    - (3-3)에서는 토큰으로 접근할 수 있는 리소스의 범위 목록을 출력함
    - (3-4)에서는 토큰의 발행 일시를 출력함
    - (3-5)에서는 토큰의 만료 일시를 출력함

<br>

#### 2. OAuth2AuthorizedClient를 핸들러 메서드의 파라미터로 전달받는 방법

```java
import org.springframework.security.oauth2.client.OAuth2AuthorizedClient;
import org.springframework.security.oauth2.client.annotation.RegisteredOAuth2AuthorizedClient;
import org.springframework.security.oauth2.core.OAuth2AccessToken;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SampleHomeController {
    @GetMapping("/Sample-oauth2")
    public String home(@RegisteredOAuth2AuthorizedClient("google") OAuth2AuthorizedClient authorizedClient) { // (1)

        OAuth2AccessToken accessToken = authorizedClient.getAccessToken();
        System.out.println("Access Token Value: " + accessToken.getTokenValue());
        System.out.println("Access Token Type: " + accessToken.getTokenType().getValue());
        System.out.println("Access Token Scopes: " + accessToken.getScopes());
        System.out.println("Access Token Issued At: " + accessToken.getIssuedAt());
        System.out.println("Access Token Expires At: " + accessToken.getExpiresAt());

        return "Sample-oauth2";
    }
}
```

<br>

- (1)과 같이 @RegisteredOAuth2AuthorizedClient 애너테이션을 이용해 아예 OAuth2AuthorizedClientRepository에 저장되어 있는 OAuth2AuthorizedClient를 파라미터로 전달받아서 Access Token 정보를 얻고 있음

<br>

- 두 가지 방법 중에서 어떤 방법을 사용해도 상관없지만 하나 이상의 핸들러 메서드에서 OAuth2AuthorizedClient를 사용해야 한다면 OAuth2AuthorizedClientService를 DI 받아서 사용하는 것이 바람직하다고 할 수 있음