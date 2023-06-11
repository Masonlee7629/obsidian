태그: #Spring 
연결문서: [Spring Security - JWT 인증 5](Spring%20Security%20-%20JWT%20인증%205.md)

# Spring Security에서의 JWT 인증

## JWT 자격 증명을 위한 로그인 인증 구현

### 기본적인 로그인 인증 흐름
1. 클라이언트가 서버 측에 로그인 인증을 요청함(Username/Password를 서버 측에 전송)
2. 로그인 인증을 담당하는 Security Filter(JwtAuthenticationFilter)가 클라이언트의 로그인 인증 정보를 수신함
3. Security Filter가 수신한 로그인 인증 정보를 AuthenticationManager에게 전달해 인증 처리를 위임함
4. AuthenticationManager가 Custom UserDetailsService(MemberDetailsService)에게 사용자의 UserDetails 조회를 위임함
5. Custom UserDetailsService(MemberDetailsService)가 사용자의 크리덴셜을 DB에서 조회한 후, AuthenticationManager에게 사용자의 UserDetails를 전달함
6. AuthenticationManager가 로그인 인증 정보와 UserDetails의 정보를 비교해 인증 처리함
7. JWT 생성 후, 클라이언트의 응답으로 전달함

<br>

### JWT 자격 증명을 위한 로그인 인증 구현

#### 1. Custom UserDetailsService 구현
- Spring Security에서 사용자의 로그인 인증을 처리하는 가장 단순하고 효과적인 방법은 데이터베이스에서 사용자의 크리덴셜을 조회한 후, 조회한 크리덴셜을 AuthenticationManager에게 전달하는 Custom UserDetailsService를 구현하는 것
<br>

```java
// MemberDetailsService
import com.codestates.auth.utils.CustomAuthorityUtils;
import com.codestates.exception.BusinessLogicException;
import com.codestates.exception.ExceptionCode;
import com.codestates.member.entity.Member;
import com.codestates.member.repository.MemberRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Optional;

@Component
public class MemberDetailsService implements UserDetailsService {
    private final MemberRepository memberRepository;
    private final CustomAuthorityUtils authorityUtils;

    public MemberDetailsService(MemberRepository memberRepository, CustomAuthorityUtils authorityUtils) {
        this.memberRepository = memberRepository;
        this.authorityUtils = authorityUtils;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Member> optionalMember = memberRepository.findByEmail(username);
        Member findMember = optionalMember.orElseThrow(() -> new BusinessLogicException(ExceptionCode.MEMBER_NOT_FOUND));

        return new MemberDetails(findMember);
    }

    private final class MemberDetails extends Member implements UserDetails {
        // (1)
        MemberDetails(Member member) {
            setMemberId(member.getMemberId());
            setEmail(member.getEmail());
            setPassword(member.getPassword());
            setRoles(member.getRoles());
        }

        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            return authorityUtils.createAuthorities(this.getRoles());
        }

        @Override
        public String getUsername() {
            return getEmail();
        }

        @Override
        public boolean isAccountNonExpired() {
            return true;
        }

        @Override
        public boolean isAccountNonLocked() {
            return true;
        }

        @Override
        public boolean isCredentialsNonExpired() {
            return true;
        }

        @Override
        public boolean isEnabled() {
            return true;
        }
    }
}
```

<br>

#### 2. LoginDTO 클래스 생성
- 클라이언트가 전송한 Username/Password 정보를 Security Filter에서 사용할 수 있도록 역직렬화(Deserialization) 하기 위한 DTO 클래스
- 클라이언트의 Username과 Passoword 정보만 담는 단순한 DTO 클래스
<br>

```java
@Getter
public class LoginDto {
    private String username;
    private String password;
}
```

<br>

#### 3. JWT를 생성하는 JwtTokenizer 구현
- JwtTokenizer 클래스는 로그인 인증에 성공한 클라이언트에게 JWT를 생성 및 발급하고 클라이언트의 요청이 들어올 때마다 전달된 JWT를 검증하는 역할을 함
<br>

```java
// JwtTokenizer
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

// (1)
@Component
public class JwtTokenizer {
    @Getter
    @Value("${jwt.key}")
    private String secretKey;       // (2)

    @Getter
    @Value("${jwt.access-token-expiration-minutes}")
    private int accessTokenExpirationMinutes;        // (3)

    @Getter
    @Value("${jwt.refresh-token-expiration-minutes}")
    private int refreshTokenExpirationMinutes;          // (4)

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

    public Jws<Claims> getClaims(String jws, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        Jws<Claims> claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(jws);
        return claims;
    }

    public void verifySignature(String jws, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(jws);
    }

    // (5)
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

- (1)에서는 JwtTokenizer 클래스를 Spring Container(ApplicationContext)에 Bean으로 등록하기 위해 @Component 애너테이션을 추가함
- (2), (3), (4)는 JWT 생성 시 필요한 정보이며, 해당 정보는 application.yml 파일에서 로드함
    - (2)는 JWT 생성 및 검증 시 사용되는 Secret Key 정보
    - (3)은 Access Token에 대한 만료 시간 정보
    - (4)는 Refresh Token에 대한 만료 시간 정보
- (5)의 getTokenExpiration() 메서드는 JWT의 만료 일시를 지정하기 위한 메서드로 JWT 생성 시 사용됨

<br>

```
// application.yml
...

jwt:
  key: ${JWT_SECRET_KEY}         # 민감한 정보는 시스템 환경 변수에서 로드함
  access-token-expiration-minutes: 30
  refresh-token-expiration-minutes: 420
```

<br>

- JWT의 서명에 사용되는 Secret Key 정보는 민감한(sensitive) 정보이므로 시스템 환경 변수의 변수로 등록함
    - ${JWT_SECRET_KEY}는 단순한 문자열이 아니라 OS의 시스템 환경 변수의 값을 읽어오는 일종의 표현식
    - 사용 중인 운영체제에 따라 아래 안내를 참고하여 환경변수를 설정함
    - 환경변수를 설정한 이후엔 사용중인 IntelliJ IDE를 반드시 Restart해야 함
        - 시스템 환경 변수에 등록한 변수를 사용할 때는 applicatioin.yml 파일의 프로퍼티 명과 동일한 문자열을 사용하지 않도록 주의해야 함
        - 시스템 환경 변수와 application.yml에 정의한 프로퍼티 명의 문자열이 동일할 경우 application.yml 파일에 정의된 프로퍼티를 클래스의 필드에서 참조할 때(예: ${jwt.key.secret}) 시스템 환경 변수의 값으로 채워지므로 개발자가 의도하지 않은 값으로 채워질 수 있음
        - 따라서 가급적 시스템 환경 변수의 값도 application.yml에서 먼저 로드한 뒤에 application.yml에서 일관성 있게 프로퍼티 값을 읽어오는 것이 좋음
        - 시스템 환경 변수에 정의한 변수명과 application.yml에 정의한 프로퍼티명의 이름을 각각 다르게 추가하면 디버깅 시, application.yml에서 값을 로드하는지 시스템 환경 변수에서 값을 로드하는지 직관적으로 알 수 있음
- access-token-expiration-minutes는 Access Token의 만료 시간이며, 30분으로 설정하였음
- refresh-token-expiration-minutes는 Refresh Token의 만료 시간이며, 420분으로 설정하였음
- Access Token과 Refresh Token의 만료 시간은 애플리케이션 서비스를 제작하면서 적절하게 설정하면 되는 값이기 때문에 딱히 적절한 시간이 정해져 있는 것은 아니므로 애플리케이션의 요구 사항에 맞게 적절한 값을 지정하면 됨
- 일반적으로 보안상 Access Token의 만료 시간이 Refresh Token의 만료 시간보다 짧은 것이 권장되며, 보안 강화를 이유로 Refresh Token을 제공하지 않는 애플리케이션도 있음

<br>

#### 4. Custom Security Filter 구현
- Custom Filter는 클라이언트의 로그인 인증 정보를 직접적으로 수신하여 인증 처리의 엔트리포인트(Entrypoint) 역할을 함
<br>

```java
import com.codestates.auth.dto.LoginDto;
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.member.entity.Member;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.SneakyThrows;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.*;

public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {  // (1)
    private final AuthenticationManager authenticationManager;
    private final JwtTokenizer jwtTokenizer;

    // (2)
    public JwtAuthenticationFilter(AuthenticationManager authenticationManager, JwtTokenizer jwtTokenizer) {
        this.authenticationManager = authenticationManager;
        this.jwtTokenizer = jwtTokenizer;
    }

    // (3)
    @SneakyThrows
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) {

        ObjectMapper objectMapper = new ObjectMapper();    // (3-1)
        LoginDto loginDto = objectMapper.readValue(request.getInputStream(), LoginDto.class); // (3-2)

        // (3-3)
        UsernamePasswordAuthenticationToken authenticationToken =
                                                new UsernamePasswordAuthenticationToken(loginDto.getUsername(), loginDto.getPassword());

        return authenticationManager.authenticate(authenticationToken);  // (3-4)
    }

    // (4)
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) {
        Member member = (Member) authResult.getPrincipal();  // (4-1)

        String accessToken = delegateAccessToken(member);   // (4-2)
        String refreshToken = delegateRefreshToken(member); // (4-3)

        response.setHeader("Authorization", "Bearer " + accessToken);  // (4-4)
        response.setHeader("Refresh", refreshToken);                   // (4-5)
    }

    // (5)
    private String delegateAccessToken(Member member) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", member.getEmail());
        claims.put("roles", member.getRoles());

        String subject = member.getEmail();
        Date expiration = jwtTokenizer.getTokenExpiration(jwtTokenizer.getAccessTokenExpirationMinutes());

        String base64EncodedSecretKey = jwtTokenizer.encodeBase64SecretKey(jwtTokenizer.getSecretKey());

        String accessToken = jwtTokenizer.generateAccessToken(claims, subject, expiration, base64EncodedSecretKey);

        return accessToken;
    }

    // (6)
    private String delegateRefreshToken(Member member) {
        String subject = member.getEmail();
        Date expiration = jwtTokenizer.getTokenExpiration(jwtTokenizer.getRefreshTokenExpirationMinutes());
        String base64EncodedSecretKey = jwtTokenizer.encodeBase64SecretKey(jwtTokenizer.getSecretKey());

        String refreshToken = jwtTokenizer.generateRefreshToken(subject, expiration, base64EncodedSecretKey);

        return refreshToken;
    }
}
```

<br>

- (1)에서는 UsernamePasswordAuthenticationFilter를 상속하고 있음
    - UsernamePasswordAuthenticationFilter는 폼 로그인 방식에서 사용하는 디폴트 Security Filter로써, 폼 로그인이 아니더라도 Username/Password 기반의 인증을 처리하기 위해 UsernamePasswordAuthenticationFilter를 확장해서 구현할 수 있음
- (2)에서는 AuthenticationManager와 JwtTokenizer를 DI 받음
    - DI 받은 AuthenticationManager는 로그인 인증 정보(Username/Password)를 전달받아 UserDetailsService와 인터랙션 한 뒤 인증 여부를 판단함
    - DI 받은 JwtTokenizer는 클라이언트가 인증에 성공할 경우, JWT를 생성 및 발급하는 역할을 함
- (3)의 attemptAuthentication()는 메서드는 메서드 내부에서 인증을 시도하는 로직을 구현함
    - (3-1)에서는 클라이언트에서 전송한 Username과 Password를 DTO 클래스로 역직렬화(Deserialization) 하기 위해 ObjectMapper 인스턴스를 생성함
    - (3-2)에서는 objectMapper.readValue(request.getInputStream(), LoginDto.class)를 통해 ServletInputStream을 LoginDto 클래스의 객체로 역직렬화(Deserialization)함
    - (3-3)에서는 Username과 Password 정보를 포함한 UsernamePasswordAuthenticationToken을 생성함
    - (3-4)에서는 UsernamePasswordAuthenticationToken을 AuthenticationManager에게 전달하면서 인증 처리를 위임함
- (4)의 successfulAuthentication() 메서드는 클라이언트의 인증 정보를 이용해 인증에 성공할 경우 호출됨
    - (4-1)에서 authResult.getPrincipal()로 Member 엔티티 클래스의 객체를 얻음
        - AuthenticationManager 내부에서 인증에 성공하면 인증된 Authentication 객체가 생성되면서 principal 필드에 Member 객체가 할당됨
    - (4-2)에서 delegateAccessToken(member) 메서드를 이용해 Access Token을 생성함
    - (4-3)에서 delegateRefreshToken(member) 메서드를 이용해 Refresh Token을 생성함
    - (4-4)에서 response header(Authorization)에 Access Token을 추가합니다. Access Token은 클라이언트 측에서 백엔드 애플리케이션 측에 요청을 보낼 때마다 request header에 추가해서 클라이언트 측의 자격을 증명하는 데 사용됨
    - (4-5)에서 response header(Refresh)에 Refresh Token을 추가하며, Refresh Token은 Access Token이 만료될 경우, 클라이언트 측이 Access Token을 새로 발급 받기 위해 클라이언트에게 추가적으로 제공될 수 있으며 Refresh Token을 Access Token과 함께 클라이언트에게 제공할지 여부는 애플리케이션의 요구 사항에 따라 달라질 수 있음
- (5)와 (6)은 Access Token과 Refresh Token을 생성하는 구체적인 로직

<br>

#### Custom Filter 추가를 위한 SecurityConfiguration 설정 추가
- Custom Filter인 JwtAuthenticationFilter의 구현이 끝났다면 JwtAuthenticationFilter를 Spring Security Filter Chain에 추가해서 로그인 인증을 처리하도록 해야 함
<br>

```java
import com.codestates.auth.filter.JwtAuthenticationFilter;
import com.codestates.auth.jwt.JwtTokenizer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {
    private final JwtTokenizer jwtTokenizer;

    public SecurityConfiguration(JwtTokenizer jwtTokenizer) {
        this.jwtTokenizer = jwtTokenizer;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers().frameOptions().sameOrigin()
            .and()
            .csrf().disable()
            .cors(withDefaults())
            .formLogin().disable()
            .httpBasic().disable()
            .apply(new CustomFilterConfigurer())   // (1)
            .and()
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().permitAll()
            );
        return http.build();
    }

    ...
    
    // (2)
    public class CustomFilterConfigurer extends AbstractHttpConfigurer<CustomFilterConfigurer, HttpSecurity> {  // (2-1)
        @Override
        public void configure(HttpSecurity builder) throws Exception {  // (2-2)
            AuthenticationManager authenticationManager = builder.getSharedObject(AuthenticationManager.class);  // (2-3)

            JwtAuthenticationFilter jwtAuthenticationFilter = new JwtAuthenticationFilter(authenticationManager, jwtTokenizer);  // (2-4)
            jwtAuthenticationFilter.setFilterProcessesUrl("/auth/login");          // (2-5)

            builder.addFilter(jwtAuthenticationFilter);  // (2-6)
        }
    }
}
```

- Spring Security에서는 (1)과 같이 apply() 메서드를 사용하여 개발자가 직접 Custom Configurer를 구성해 Spring Security의 Configuration을 커스터마이징(customizations) 할 수 있음
- (2)는 Custom Configurer인 CustomFilterConfigurer 클래스로 JwtAuthenticationFilter를 등록하는 역할을 함
    - (2-1)과 같이 AbstractHttpConfigurer를 상속해서 Custom Configurer를 구현할 수 있음
        - AbstractHttpConfigurer<CustomFilterConfigurer, HttpSecurity>와 같이 AbstractHttpConfigurer를 상속하는 타입과 HttpSecurityBuilder를 상속하는 타입을 제너릭 타입으로 지정할 수 있음
    - (2-2)와 같이 configure() 메서드를 오버라이드해서 Configuration을 커스터마이징할 수 있음
    - (2-3)과 같이 getSharedObject(AuthenticationManager.class)를 통해 AuthenticationManager의 객체를 얻을 수 있음
        - getSharedObject()를 통해서 Spring Security의 설정을 구성하는 SecurityConfigurer 간에 공유되는 객체를 얻을 수 있음
    - (2-4)에서 JwtAuthenticationFilter를 생성하면서 JwtAuthenticationFilter에서 사용되는 AuthenticationManager와 JwtTokenizer를 DI 해줌
    - (2-5)에서는 setFilterProcessesUrl() 메서드를 통해 디폴트 request URL인 “/login”을 “/auth/login”으로 변경함
    - (2-6)에서 addFilter() 메서드를 통해 JwtAuthenticationFilter를 Spring Security Filter Chain에 추가함

<br>

### 로그인 인증 테스트
1. 회원 가입 요청
    - Postman에서 회원 가입 request를 전송해서 회원 가입을 함
2. 로그인 인증 요청
    - SecurityConfiguration에서 변경한 URL(/auth/login)로 로그인 인증 request를 전송해야 함
    - 애플리케이션에서 로그인 인증이 성공적으로 수행되면 Postman 하단부의 Headers 탭에서 Authorization 키의 값으로 Access Token이, Refresh 키의 값으로 Refresh Token이 포함됨
    - 클라이언트 쪽에서는 서버 측의 리소스를 사용하기 위한 request를 전송할 때마다 전달받은 JWT를 request header에 포함 후, 클라이언트의 자격 증명 정보로 사용하면 됨

<br>

### 로그인 인증 성공 및 실패에 따른 추가 처리
- Spring Security에서는 Username/Password 기반의 로그인 인증에 성공했을 때, 로그를 기록한다거나 로그인에 성공한 사용자 정보를 response로 전송하는 등의 추가 처리를 할 수 있는 핸들러(AuthenticationSuccessHandler)를 지원하며, 로그인 인증 실패 시에도 마찬가지로 인증 실패에 대해 추가 처리를 할 수 있는 핸들러(AuthenticationFailureHandler)를 지원함

<br>

#### 1. AuthenticationSuccessHandler 구현
- 로그인 인증 성공 시 추가 작업을 할 수 있음
<br>

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
public class MemberAuthenticationSuccessHandler implements AuthenticationSuccessHandler {  // (1)
    // (2)
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response,
                                        Authentication authentication) throws IOException {
        // 인증 성공 후, 로그를 기록하거나 사용자 정보를 response로 전송하는 등의 추가 작업을 할 수 있음
        log.info("# Authenticated successfully!");
    }
}
```

<br>

- 개발자가 직접 정의하는 Custom AuthenticationSuccessHandler는 (1)과 같이 AuthenticationSuccessHandler 인터페이스를 구현해야 함
    - AuthenticationSuccessHandler 인터페이스에는 onAuthenticationSuccess() 추상 메서드가 정의되어 있으며, onAuthenticationSuccess() 메서드를 구현해서 추가 처리를 함
- (2)에서는 단순히 로그만 출력하고 있지만 Authentication 객체에 사용자 정보를 얻은 후, HttpServletResponse로 출력 스트림을 생성하여 response를 전송할 수 있음

<br>

#### 2. AuthenticationFailureHandler 구현
- 로그인 인증 실패 시 추가 작업을 할 수 있음
<br>

```java
import com.codestates.response.ErrorResponse;
import com.google.gson.Gson;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
public class MemberAuthenticationFailureHandler implements AuthenticationFailureHandler {  // (1)
    @Override
    public void onAuthenticationFailure(HttpServletRequest request,
                                        HttpServletResponse response,
                                        AuthenticationException exception) throws IOException {
        // 인증 실패 시, 에러 로그를 기록하거나 error response를 전송할 수 있음
        log.error("# Authentication failed: {}", exception.getMessage());

        sendErrorResponse(response);  // (2)
    }
    
    private void sendErrorResponse(HttpServletResponse response) throws IOException {
        Gson gson = new Gson();     // (2-1)
        ErrorResponse errorResponse = ErrorResponse.of(HttpStatus.UNAUTHORIZED); // (2-2)
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);    // (2-3)
        response.setStatus(HttpStatus.UNAUTHORIZED.value());          // (2-4)
        response.getWriter().write(gson.toJson(errorResponse, ErrorResponse.class));   // (2-5)
    }
}
```

<br>

- 개발자가 직접 정의하는 Custom AuthenticationFailureHandler는 (1)과 같이 AuthenticationFailureHandler 인터페이스를 구현해야 함
    - AuthenticationSuccessHandler 인터페이스에는 onAuthenticationFailure() 추상 메서드가 정의되어 있으며, onAuthenticationFailure() 메서드를 구현해서 추가 처리를 함
- (2)에서는 바로 아래에 있는 sendErrorResponse() 메서드를 호출해 출력 스트림에 Error 정보를 담고 있음
    - (2-1)에서는 Error 정보가 담긴 객체(ErrorResponse)를 JSON 문자열로 변환하는 데 사용되는 Gson 라이브러리의 인스턴스를 생성함
    - (2-2)에서는 ErrorResponse 객체를 생성하며, ErrorResponse.of() 메서드로 HttpStatus.UNAUTHORIZED 상태 코드를 전달함
    - (2-3)에서는 response의 Content Type이 “application/json”이라는 것을 클라이언트에게 알려줄 수 있도록 MediaType.APPLICATION_JSON_VALUE를 HTTP Header에 추가함
    - (2-4)에서는 response의 status가 401 임을 클라이언트에게 알려줄 수 있도록 HttpStatus.UNAUTHORIZED.value()을 HTTP Header에 추가함
    - (2-5)에서는 Gson을 이용해 ErrorResponse 객체를 JSON 포맷 문자열로 변환 후, 출력 스트림을 생성함

<br>

#### 3. AuthenticationSuccessHandler와 AuthenticationFailureHandler 추가
- AuthenticationSuccessHandler 인터페이스와 AuthenticationFailureHandler 인터페이스의 구현 클래스를 JwtAuthenticationFilter에 등록하면 로그인 인증 시, 두 핸들러를 사용할 수 있음
<br>

```java
import com.codestates.auth.filter.JwtAuthenticationFilter;
import com.codestates.auth.jwt.JwtTokenizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import com.codestates.auth.handler.MemberAuthenticationFailureHandler;
import com.codestates.auth.handler.MemberAuthenticationSuccessHandler;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

import static org.springframework.security.config.Customizer.withDefaults;

/**
 * JwtAuthenticationFilter 추가
 */
@Configuration
public class SecurityConfiguration {
    private final JwtTokenizer jwtTokenizer;
   
    public SecurityConfiguration(JwtTokenizer jwtTokenizer) {
        this.jwtTokenizer = jwtTokenizer;
    }

    ...
    ...

    public class CustomFilterConfigurer extends AbstractHttpConfigurer<CustomFilterConfigurer, HttpSecurity> {
        @Override
        public void configure(HttpSecurity builder) throws Exception {
            AuthenticationManager authenticationManager = builder.getSharedObject(AuthenticationManager.class);

            JwtAuthenticationFilter jwtAuthenticationFilter = new JwtAuthenticationFilter(authenticationManager, jwtTokenizer);
            jwtAuthenticationFilter.setFilterProcessesUrl("/auth/login");
            jwtAuthenticationFilter.setAuthenticationSuccessHandler(new MemberAuthenticationSuccessHandler());  // (1) 추가
            jwtAuthenticationFilter.setAuthenticationFailureHandler(new MemberAuthenticationFailureHandler());  // (2) 추가
            builder.addFilter(jwtAuthenticationFilter);
        }
    }
}
```

<br>

- (1), (2)와 같이 JwtAuthenticationFilter에 등록함
    - new 키워드 사용 이유
        - AuthenticationSuccessHandler와 AuthenticationFailureHandler 인터페이스의 구현 클래스가 다른 Security Filter에서 사용이 된다면 ApplicationContext에 Bean으로 등록해서 DI 받는 게 맞음
        - 일반적으로 인증을 위한 Security Filter마다 AuthenticationSuccessHandler와 AuthenticationFailureHandler의 구현 클래스를 각각 생성할 것이므로 new 키워드를 사용해서 객체를 생성해도 무방함

<br>

#### 4. AuthenticationSuccessHandler 호출

```java
// JwtAuthenticationFilter(AuthenticationSuccessHandler 호출 코드 추가)
import com.codestates.auth.dto.LoginDto;
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.member.entity.Member;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.SneakyThrows;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class JwtAuthenticationFilter extends UsernamePasswordAuthenticationFilter {
    ...

    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws ServletException, IOException {
        Member member = (Member) authResult.getPrincipal();

        String accessToken = delegateAccessToken(member);
        String refreshToken = delegateRefreshToken(member);

        response.setHeader("Authorization", "Bearer " + accessToken);
        response.setHeader("Refresh", refreshToken);

        this.getSuccessHandler().onAuthenticationSuccess(request, response, authResult);  // (1) 추가
    }

    ...
}
```

<br>

- 로그인 인증에 성공하고, JWT를 생성해서 response header에 추가한 뒤, (1)과 같이 AuthenticationSuccessHandler의 onAuthenticationSuccess() 메서드를 호출함
- onAuthenticationSuccess() 메서드를 호출하면 앞에서 구현한 MemberAuthenticationSuccessHandler의 onAuthenticationSuccess() 메서드가 호출됨
- AuthenticationFailureHandler는 별도의 코드를 추가하지 않아도 로그인 인증에 실패하면 개발자가 구현한 MemberAuthenticationFailureHandler의 onAuthenticationFailure() 메서드가 알아서 호출됨