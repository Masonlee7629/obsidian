태그: #Spring 
연결문서: [Spring WebFlux - 리액티브 프로그래밍 1](Spring%20WebFlux%20-%20리액티브%20프로그래밍%201.md)

# Spring Security에서의 OAuth2 인증

## OAuth 2와 JWT를 이용한 샘플 애플리케이션 구현
- 이번에는 Frontend와 Backend가 분리된 CSR(Client Side Rendering) 방식의 애플리케이션에 Google의 OAuth2 인증 시스템을 적용해볼 것
- CSR(Client Side Rendering) 방식의 애플리케이션에 OAuth2 인증 시스템을 도입할 경우에도 마찬가지로 OAuth2 인증 시스템을 통해 인증에 성공한 사용자에 대한 자격 증명 정보를 JWT로 제공해 줄 수 있음
- CSR(Client Side Rendering) 방식의 애플리케이션에 OAuth 2 + JWT를 제대로 잘 적용하기 위해서는 먼저 OAuth 2의 인증 처리 흐름과 JWT를 통한 자격 증명 정보 제공 시점에 대해 이해하는 것이 중요함

<br>

### Frontend와 Backend 간의 OAuth 2 인증 처리 흐름

![](image_37.png)

<br>

1. Resource Owner가 웹 브라우저에서 ‘Google 로그인 링크’를 클릭함
2. Frontend 애플리케이션에서 Backend 애플리케이션의 http://localhost:8080/oauth2/authorization/google 로 request를 전송하며, 이 URI의 request는 OAuth2LoginAuthenticationFilter 가 처리함
3. Google의 로그인 화면을 요청하는 URI로 리다이렉트 함
    - 이때 Authorization Server가 Backend 애플리케이션 쪽으로 Authorization Code를 전송할 Redirect URI(http://localhost:8080/login/oauth2/code/google) 를 쿼리 파라미터로 전달함
    - Redirect URI는 Spring Security가 내부적으로 제공함
4. Google 로그인 화면을 오픈함
5. Resource Owner가 Google 로그인 인증 정보를 입력해서 로그인을 수행함
6. 로그인에 성공하면 (3)에서 전달한 Backend Redirect URI(http://localhost:8080/login/oauth2/code/google) 로 Authorization Code를 요청함
7. Authorization Server가 Backend 애플리케이션에게 Authorization Code를 응답으로 전송함
8. Backend 애플리케이션이 Authorization Server에 Access Token을 요청함
9. Authorization Server가 Backend 애플리케이션에게 Access Token을 응답으로 전송
    - 여기서 Access Token은 Google Resource Server에게 Resource를 요청하는 용도로 사용됨
10. Backend 애플리케이션이 Resource Server에 User Info를 요청함
    - 여기서의 User Info는 Resource Owner에 대한 이메일 주소, 프로필 정보 등을 의미함
11. Resource Server가 Backend 애플리케이션에 User Info를 응답으로 전송함
12. Backend 애플리케이션은 JWT로 구성된 Access Token과 Refresh Token을 생성한 후, Frontend 애플리케이션에 JWT(Access Token과 Refresh Token)를 전달하기 위해 Frontend 애플리케이션(http://localhost?access_token={jwt-access-token}&refresh_token={jwt-refresh-token}) 으로 Redirect함
<br>

- 동작 흐름이 아주 복잡해 보이지만 (6)부터 (11)까지는 Spring Security에서 내부적으로 알아서 처리해 주기 때문에 기본적으로는 건드릴 필요가 없음
- OAuth 2 인증 처리가 정상적으로 동작하는지 확인하기 위해서는 웹서버에서 실제로 실행되는 Frontend 애플리케이션이 필요함

<br>

### Frontend 애플리케이션 준비

#### 1. 아파치 웹서버 설치 - Windows OS 사용자
- Windows OS 사용자는 아래의 순서대로 Frontend 애플리케이션 실행 환경을 준비함
    - 링크에서 아파치 웹서버를 다운로드함
        - [https://www.apachelounge.com/download/](https://www.apachelounge.com/download/)
    - 다운로드한 파일의 압축을 해제함
    - Apache24 디렉토리를 C:\ 디렉토리로 이동함(최종 경로는 C:\Apache24)
    - httpd.conf 파일을 메모장 등의 에디터로 오픈함
    - ServerName을 주석 해제하고 수정함
        - ServerName localhost:80
        - 나머지는 디폴트 값을 그대로 사용
    - 윈도우 키 + S를 누른 후, cmd를 검색해서 마우스 오른쪽 버튼을 눌러 명령 프롬프트(cmd) 창을 관리자 모드로 실행함
    - 명령 프롬프트 창에서 C:\Apache24\bin 디렉토리로 이동함
    - httpd.exe -k install 명령을 입력함
    - C:\Apache24/bin 경로 내에 있는 ApacheMonitor.exe를 더블 클릭해서 아파치 웹서버를 실행함
        - 아파치 웹서버는 바탕화면 오른쪽 하단의 빠른 실행 창에서 실행/중지할 수 있음
    - 웹브라우저에서 http://localhost 로 접속하여 아파치 웹서버가 정상적으로 실행이 되는 것을 확인함

<br>

#### 2. Frontend 샘플 애플리케이션을 아파치 웹서버에 배포
- 세 개 HTML 파일을 에디터로 작성한 후, 각 운영체제에 맞는 경로의 디렉토리로 위치시켜야 함
    - Windows OS 사용자 : C:\Apache24\htdocs
    - Mac OS 사용자 : /Library/WebServer/Documents
        - 현재 위치의 하위 디렉토리가 아닌 루트(/)의 하위 경로
        - 해당 위치에서 파일을 직접 생성할 땐 반드시 sudo 명령어를 붙여야 함
        - 다른 경로에서 해당 위치로 드래그 앤 드롭을 이용해 이동할 땐 암호와 함께 허용 여부를 확인하는 메시지가 뜰 수 있음
<br>

- index.html
    - index.html에서 Google로 로그인 버튼을 클릭하면 Backend 애플리케이션으로 request가 전송되고, Goolge 로그인 화면이 오픈됨
<br>

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>OAuth2 + JWT Frontend</title>
</head>
<body>
    <h2>Welcome to OAuth 2.0 + JWT Spring Security</h2>
    <a href="http://localhost:8080/oauth2/authorization/google">Google로 로그인</a>
</body>
</html>
```

<br>

- receive-token.html
    - receive-token.html 은 Backend 애플리케이션에서 전달받은 JWT Access Token과 Refresh Token을 웹브라우저의 LocalStorage에 저장한 후, my-page.html 이동함
<br>

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>OAuth2 + JWT My page</title>
</head>
<body>
    <script type="text/javascript">
        let accessToken = (new URL(location.href)).searchParams.get('access_token');
        let refreshToken = (new URL(location.href)).searchParams.get('refresh_token');

        localStorage.setItem("accessToken", accessToken)
        localStorage.setItem("refreshToken", refreshToken)

        location.href = 'my-page.html'
    </script>
</body>
</html>
```

<br>

- my-page.html
    - my-page.html에서는 LocalStorage에 저장된 JWT Access Token과 Refresh Token을 로드해서 웹 브라우저에 표시함
<br>

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>OAuth2 + JWT My page</title>
</head>
<body>
    <h2>My Page</h2>
    <h3>아래의 토큰을 이용해서 Backend 애플리케이션의 리소스를 요청할 수 있습니다.</h3>
    <p>
        <span>Access Token: </span><span id="accessToken" style="color: blue"></span>
    </p>
    <p>
        <span>Refresh Token: </span><span id="refreshToken" style="color: blue"></span>
    </p>
    <script type="text/javascript">
        let accessToken = localStorage.getItem('accessToken')
        let refreshToken = localStorage.getItem('refreshToken');

        document.getElementById("accessToken").textContent = accessToken;
        document.getElementById("refreshToken").textContent = refreshToken;
    </script>
</body>
</html>
```

<br>

### Backend 애플리케이션에 OAuth 2 인증 기능 적용

#### 1. JwtTokenizer 추가
- 제일 먼저 할 일은 JWT를 생성하고 JWT를 검증해 주는 JwtTokenizer를 구현하는 것
<br>

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.io.Encoders;
import io.jsonwebtoken.security.Keys;
import lombok.Getter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Calendar;
import java.util.Date;
import java.util.Map;

@Component
public class JwtTokenizer {
    @Getter
    @Value("${jwt.key.secret}")
    private String secretKey;

    @Getter
    @Value("${jwt.access-token-expiration-minutes}")
    private int accessTokenExpirationMinutes;

    @Getter
    @Value("${jwt.refresh-token-expiration-minutes}")
    private int refreshTokenExpirationMinutes;

    public String encodeBase64SecretKey(String secretKey) {
        return Encoders.BASE64.encode(secretKey.getBytes(StandardCharsets.UTF_8));
    }

    public String generateAccessToken(Map<String, Object> claims,
                                      String subject,
                                      Date expiration,
                                      String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(Calendar.getInstance().getTime())
                .setExpiration(expiration)
                .signWith(key)
                .compact();
    }

    public String generateRefreshToken(String subject, Date expiration, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        return Jwts.builder()
                .setSubject(subject)
                .setIssuedAt(Calendar.getInstance().getTime())
                .setExpiration(expiration)
                .signWith(key)
                .compact();
    }

    // 검증 후, Claims을 반환하는 용도
    public Jws<Claims> getClaims(String jws, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        Jws<Claims> claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(jws);
        return claims;
    }

    // 단순히 검증만 하는 용도로 쓰일 경우
    public void verifySignature(String jws, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(jws);
    }

    public Date getTokenExpiration(int expirationMinutes) {
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.MINUTE, expirationMinutes);
        Date expiration = calendar.getTime();

        return expiration;
    }

    private Key getKeyFromBase64EncodedKey(String base64EncodedSecretKey) {
        byte[] keyBytes = Decoders.BASE64.decode(base64EncodedSecretKey);
        Key key = Keys.hmacShaKeyFor(keyBytes);

        return key;
    }
}
```

<br>

#### 2. application.yml 설정

```java
spring:
  h2:
    console:
      enabled: true
      path: /h2
  datasource:
    url: jdbc:h2:mem:test
  jpa:
    hibernate:
      ddl-auto: create  # (1) 스키마 자동 생성
    show-sql: true      # (2) SQL 쿼리 출력
    properties:
      hibernate:
        format_sql: true  # (3) SQL pretty print
  sql:
    init:
      data-locations: classpath*:db/h2/data.sql
  security:
    oauth2:
      client:
        registration:
          google:
            clientId: ${G_CLIENT_ID}
            clientSecret: ${G_CLIENT_SECRET}
            scope:
              - email                              // (1)
              - profile                            // (2)
logging:
  level:
    org:
      springframework:
        orm:
          jpa: DEBUG
server:
  servlet:
    encoding:
      force-response: true
mail:
  address:
    admin: admin@gmail.com
jwt:
  key:
    secret: ${JWT_SECRET_KEY}               # 민감한 정보는 시스템 환경 변수에서 로드함
  access-token-expiration-minutes: 30
  refresh-token-expiration-minutes: 420
```

<br>

- (1), (2)와 같이 scope 값을 직접 지정하면 해당 범위만큼의 Resource를 Client(백엔드 애플리케이션)에 제공함
- (1)은 Resource Owner의 이메일 정보를 의미하고, (2)는 Resource Owner의 프로필 정보를 의미함

<br>

#### 3. JwtVerificationFilter 추가
- JwtVerificationFilter는 OAuth 2 인증에 성공하면 Frontend 애플리케이션 쪽에서 request를 전송할 때마다 Authorization header에 실어 보내는 Access Token에 대한 검증을 수행하는 Filter
<br>

```java
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.auth.utils.CustomAuthorityUtils;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;
import java.util.Map;

public class JwtVerificationFilter extends OncePerRequestFilter {  // (1)
    private final JwtTokenizer jwtTokenizer;
    private final CustomAuthorityUtils authorityUtils;

    // (2)
    public JwtVerificationFilter(JwtTokenizer jwtTokenizer,
                                 CustomAuthorityUtils authorityUtils) {
        this.jwtTokenizer = jwtTokenizer;
        this.authorityUtils = authorityUtils;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        Map<String, Object> claims = verifyJws(request); // (3)
        setAuthenticationToContext(claims);      // (4)

        filterChain.doFilter(request, response); // (5)
    }

    // (6)
    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
        String authorization = request.getHeader("Authorization");  // (6-1)

        return authorization == null || !authorization.startsWith("Bearer");  // (6-2)
    }

    private Map<String, Object> verifyJws(HttpServletRequest request) {
        String jws = request.getHeader("Authorization").replace("Bearer ", ""); // (3-1)
        String base64EncodedSecretKey = jwtTokenizer.encodeBase64SecretKey(jwtTokenizer.getSecretKey()); // (3-2)
        Map<String, Object> claims = jwtTokenizer.getClaims(jws, base64EncodedSecretKey).getBody();   // (3-3)

        return claims;
    }

    private void setAuthenticationToContext(Map<String, Object> claims) {
        String username = (String) claims.get("username");   // (4-1)
        List<GrantedAuthority> authorities = authorityUtils.createAuthorities((List)claims.get("roles"));  // (4-2)
        Authentication authentication = new UsernamePasswordAuthenticationToken(username, null, authorities);  // (4-3)
        SecurityContextHolder.getContext().setAuthentication(authentication); // (4-4)
    }
	
@Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // (1)
        try {
            Map<String, Object> claims = verifyJws(request);
            setAuthenticationToContext(claims);
        } catch (SignatureException se) {
            request.setAttribute("exception", se);
        } catch (ExpiredJwtException ee) {
            request.setAttribute("exception", ee);
        } catch (Exception e) {
            request.setAttribute("exception", e);
        }

        filterChain.doFilter(request, response);
    }
}
```

<br>

#### 4. AuthenticationSuccessHandler 구현
- AuthenticationSuccessHandler는 OAuth 2 인증에 성공하면 호출되는 핸들러
- 여기에서 JWT를 생성하고, Frontend 쪽으로 JWT를 전송하기 위해 Redirect 하는 로직을 구현함
- OAuth2MemberSuccessHandler 클래스는 OAuth 2 인증 후, Frontend 애플리케이션 쪽으로 JWT를 전송하는 핵심 역할을 담당함
<br>

```java
import com.codestates.member.entity.Member;
import com.codestates.member.service.MemberService;
import com.codestates.oauth2_jwt.jwt.JwtTokenizer;
import com.codestates.oauth2_jwt.utils.CustomAuthorityUtils;
import com.codestates.stamp.Stamp;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;
import org.springframework.util.LinkedMultiValueMap;
import org.springframework.util.MultiValueMap;
import org.springframework.web.util.UriComponentsBuilder;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URI;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class OAuth2MemberSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {   // (1)
    private final JwtTokenizer jwtTokenizer;
    private final CustomAuthorityUtils authorityUtils;
    private final MemberService memberService;

    // (2)
    public OAuth2MemberSuccessHandler(JwtTokenizer jwtTokenizer,
                                      CustomAuthorityUtils authorityUtils,
                                      MemberService memberService) {
        this.jwtTokenizer = jwtTokenizer;
        this.authorityUtils = authorityUtils;
        this.memberService = memberService;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        var oAuth2User = (OAuth2User)authentication.getPrincipal();
        String email = String.valueOf(oAuth2User.getAttributes().get("email")); // (3)
        List<String> authorities = authorityUtils.createRoles(email);           // (4)

        saveMember(email);  // (5)
        redirect(request, response, email, authorities);  // (6)
    }

    private void saveMember(String email) {
        Member member = new Member(email);
        member.setStamp(new Stamp());
        memberService.createMember(member);
    }

    private void redirect(HttpServletRequest request, HttpServletResponse response, String username, List<String> authorities) throws IOException {
        String accessToken = delegateAccessToken(username, authorities);  // (6-1)
        String refreshToken = delegateRefreshToken(username);     // (6-2)

        String uri = createURI(accessToken, refreshToken).toString();   // (6-3)
        getRedirectStrategy().sendRedirect(request, response, uri);   // (6-4)
    }

    private String delegateAccessToken(String username, List<String> authorities) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", username);
        claims.put("roles", authorities);

        String subject = username;
        Date expiration = jwtTokenizer.getTokenExpiration(jwtTokenizer.getAccessTokenExpirationMinutes());

        String base64EncodedSecretKey = jwtTokenizer.encodeBase64SecretKey(jwtTokenizer.getSecretKey());

        String accessToken = jwtTokenizer.generateAccessToken(claims, subject, expiration, base64EncodedSecretKey);

        return accessToken;
    }

    private String delegateRefreshToken(String username) {
        String subject = username;
        Date expiration = jwtTokenizer.getTokenExpiration(jwtTokenizer.getRefreshTokenExpirationMinutes());
        String base64EncodedSecretKey = jwtTokenizer.encodeBase64SecretKey(jwtTokenizer.getSecretKey());

        String refreshToken = jwtTokenizer.generateRefreshToken(subject, expiration, base64EncodedSecretKey);

        return refreshToken;
    }

    private URI createURI(String accessToken, String refreshToken) {
        MultiValueMap<String, String> queryParams = new LinkedMultiValueMap<>();
        queryParams.add("access_token", accessToken);
        queryParams.add("refresh_token", refreshToken);

        return UriComponentsBuilder
                .newInstance()
                .scheme("http")
                .host("localhost")
//                .port(80)
                .path("/receive-token.html")
                .queryParams(queryParams)
                .build()
                .toUri();
    }

}
```

<br>

- (1)과 같이 SimpleUrlAuthenticationSuccessHandler를 상속하면 Redirect를 손쉽게 할 수 있는 getRedirectStrategy().sendRedirect() 같은 API를 사용할 수 있음
- (2)와 같이 필요한 객체를 DI 받음
- (3)에서는 Authentication 객체로부터 얻어낸 OAuth2User 객체로부터 Resource Owner의 이메일 주소를 얻음
- (4)에서는 CustomAuthorityUtils를 이용해 권한 정보를 생성함
- (5)에서는 Resource Owner의 이메일 주소를 DB에 저장함
    - OAuth 2의 특성상 Resource Owner의 크리덴셜(Credential)을 Backend 애플리케이션에서 관리하지는 않지만 Backend 애플리케이션의 Resource와 연관 관계를 맺기 위해서 최소한의 정보는 Backend 애플리케이션 쪽에서 관리해도 무방함
- (6)에서는 Access Token과 Refresh Token을 생성해서 Frontend 애플리케이션에 전달하기 위해 Redirect함
    - (6-1)과 (6-2)에서는 JWT Access Token과 Refresh Token을 생성함
    - (6-3)에서는 Frontend 애플리케이션 쪽의 URL을 생성함
        - createURI() 메서드에서 UriComponentsBuilder를 이용해 Access Token과 Refresh Token을 포함한 URL을 생성하고 있음
        - UriComponentsBuilder에서 Port 설정을 하지 않으면 기본값은 80 포트
    - (6-4)에서는 SimpleUrlAuthenticationSuccessHandler에서 제공하는 sendRedirect() 메서드를 이용해 Frontend 애플리케이션 쪽으로 리다이렉트 함

<br>

##### SecurityConfigiguration 설정

```java
import com.codestates.member.service.MemberService;
import com.codestates.oauth2_jwt.auth.filter.JwtVerificationFilter;
import com.codestates.oauth2_jwt.auth.handler.MemberAccessDeniedHandler;
import com.codestates.oauth2_jwt.auth.handler.MemberAuthenticationEntryPoint;
import com.codestates.oauth2_jwt.jwt.JwtTokenizer;
import com.codestates.oauth2_jwt.auth.handler.OAuth2MemberSuccessHandler;
import com.codestates.oauth2_jwt.utils.CustomAuthorityUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.client.web.OAuth2LoginAuthenticationFilter;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {
    private final JwtTokenizer jwtTokenizer;
    private final CustomAuthorityUtils authorityUtils;
    private final MemberService memberService;

    public SecurityConfiguration(JwtTokenizer jwtTokenizer,
                                   CustomAuthorityUtils authorityUtils,
                                   MemberService memberService) {
        this.jwtTokenizer = jwtTokenizer;
        this.authorityUtils = authorityUtils;
        this.memberService = memberService;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers().frameOptions().sameOrigin()
            .and()
            .csrf().disable()
            .cors(withDefaults())
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .formLogin().disable()
            .httpBasic().disable()
            .exceptionHandling()  // 추가
            .authenticationEntryPoint(new MemberAuthenticationEntryPoint())  // 추가
            .accessDeniedHandler(new MemberAccessDeniedHandler())            // 추가
            .and()
            .apply(new CustomFilterConfigurer())  // 추가
            .and()
                .authorizeHttpRequests(authorize -> authorize // url authorization 전체 추가
//                        .antMatchers(HttpMethod.POST, "/*/members").permitAll()    // OAuth 2로 로그인하므로 회원 정보 등록 필요 없음.
//                        .antMatchers(HttpMethod.PATCH, "/*/members/**").hasRole("USER") // OAuth 2로 로그인하므로 회원 정보 수정 필요 없음.
//                        .antMatchers(HttpMethod.GET, "/*/members").hasRole("ADMIN")  // OAuth 2로 로그인하므로 회원 정보 수정 필요 없음.
//                        .antMatchers(HttpMethod.GET, "/*/members/**").hasAnyRole("USER", "ADMIN")  // OAuth 2로 로그인하므로 회원 정보 수정 필요 없음.
//                        .antMatchers(HttpMethod.DELETE, "/*/members/**").hasRole("USER") // OAuth 2로 로그인하므로 회원 정보 수정 필요 없음.
                        .antMatchers(HttpMethod.POST, "/*/coffees").hasRole("ADMIN")
                        .antMatchers(HttpMethod.PATCH, "/*/coffees/**").hasRole("ADMIN")
                        .antMatchers(HttpMethod.GET, "/*/coffees/**").hasAnyRole("USER", "ADMIN")
                        .antMatchers(HttpMethod.GET, "/*/coffees").permitAll()
                        .antMatchers(HttpMethod.DELETE, "/*/coffees").hasRole("ADMIN")
                        .antMatchers(HttpMethod.POST, "/*/orders").hasRole("USER")
                        .antMatchers(HttpMethod.PATCH, "/*/orders").hasAnyRole("USER", "ADMIN")
                        .antMatchers(HttpMethod.GET, "/*/orders/**").hasAnyRole("USER", "ADMIN")
                        .antMatchers(HttpMethod.DELETE, "/*/orders").hasRole("USER")
                        .anyRequest().permitAll()
            )
            .oauth2Login(oauth2 -> oauth2
                    .successHandler(new OAuth2MemberSuccessHandler(jwtTokenizer, authorityUtils, memberService))  // (1)
            );

        return http.build();
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("*"));
        configuration.setAllowedMethods(Arrays.asList("GET","POST", "PATCH", "DELETE"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("\/**", configuration); // 주의 사항: 컨텐츠 표시 오류로 인해 '/**'를 '\/**'로 표기했으니 실제 코드 구현 시에는 '\(역슬래시)'를 빼야 함

        return source;
    }
    
    // 추가
    public class CustomFilterConfigurer extends AbstractHttpConfigurer<CustomFilterConfigurer, HttpSecurity> {
        @Override
        public void configure(HttpSecurity builder) throws Exception {
            JwtVerificationFilter jwtVerificationFilter = new JwtVerificationFilter(jwtTokenizer, authorityUtils);
            
            builder.addFilterAfter(jwtVerificationFilter, OAuth2LoginAuthenticationFilter.class); // (2)
        }
    }
}
```

<br>

- (1)에서는 OAuth2 로그인 설정에 .successHandler()를 통해 OAuth2 인증이 성공한 뒤 실행되는 핸들러를 추가함
    - OAuth2MemberSuccessHandler 객체를 생성하면서 OAuth2MemberSuccessHandler에서 필요한 의존 객체를 DI 하고 있는 것을 확인할 수 있음
- (2)와 같이 JwtVerificationFilter를 OAuth2LoginAuthenticationFilter 뒤에 추가함
- OAuth 2 인증을 사용하므로 Backend 애플리케이션 쪽에서는 MemberController를 사용할 일이 현재로서는 없으므로 MemberController의 핸들러 메서드 쪽 URI에 대한 접근 권한은 주석 처리함

<br>

#### 5. 기타 수정된 코드
- 기타 수정된 코드는 회원 정보와 관련된 코드
- 제삼자인 써드 파티 애플리케이션(Google 서비스)의 OAuth 2 인증 시스템을 사용하기 때문에 회원 정보를 등록하거나 수정할 필요가 없으므로 이와 관련된 MemberController, MemberDto, MemberService, Member 엔티티 클래스에서 회원 정보를 등록 및 수정하는 로직의 대부분이 제거되거나 수정해야 함

<br>

### 애플리케이션 테스트
- Backend 애플리케이션을 IDE에서 실행하고, Frontend 쪽 아파치 웹서버가 실행되어 있는지 확인한 후, 웹 브라우저에 http://localhost 를 입력해서 Frontend 애플리케이션의 화면을 오픈함
- Frontend 애플리케이션의 메인 페이지에서 구글 로그인 인증 화면이 표시되고, 로그인 인증에 성공하면 JWT의 Access Token과 Refresh Token이 표시되어야 함
- JWT Access Token은 Backend 애플리케이션의 Resource를 요청하는 핸들러 메서드를 호출할 때 Authorization header에 추가해서 사용하며, Refresh Token은 Access Token이 만료되었을 때, Access Token을 새로 발급 받고자 할 때 사용할 수 있음
- Google과 같은 OAuth2 인증 시스템을 자체적으로 구축하기 위해 Authorization Server와 Resource Server를 구현할 수도 있음
    - 이 경우, Authorization Server와 Resource Server 간에도 JWT를 이용할 수 있으며, Spring Security에서는 Authorization Server와 Resource Server 간의 통신에 JWT를 이용할 수 있는 API를 제공함
- JWT에 대한 서명을 대칭키 방식으로 진행했지만 Authorization Server와 Resource Server 간에 주고받는 JWT의 보안성을 강화하기 위해 비대칭키 방식의 서명도 사용할 수 있음
    - 비대칭키로 JWT를 암복호화하는 방식은 OAuth2MemberSuccessHandler에서 Frontend 애플리케이션 쪽으로 JWT를 쿼리파라미터로 추가한 뒤 리다이렉트 할 경우에도 사용할 수 있음