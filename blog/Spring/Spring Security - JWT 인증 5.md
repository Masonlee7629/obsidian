태그: #Spring 
연결문서: [Spring Security - OAuth2 인증 1](Spring%20Security%20-%20OAuth2%20인증%201.md)

# Spring Security에서의 JWT 인증

## JWT를 이용한 자격 증명 및 검증 구현

### JWT 검증 기능 구현

#### 1. JWT 검증 필터 구현
- JWT 검증을 위해 가장 먼저 해야 할 작업은 JWT를 검증하는 전용 Security Filter를 구현하는 것
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
}
```

<br>

- Spring Security에서는 (1)과 같이 OncePerRequestFilter를 확장해서 request 당 한 번만 실행되는 Security Filter를 구현할 수 있음
    - JWT의 검증은 request 당 단 한 번만 수행하면 되기 때문에 JWT 전용 Filter로 만들기에는 OncePerRequestFilter를 이용하는 것이 적절하다고 볼 수 있음
    - 인증과 관련된 Filter는 성공이냐 실패냐를 단 한 번만 판단하면 되며, 성공도 아니고 실패도 아닌 어중간한 결과는 존재하지 않음
- (2)와 같이 JwtTokenizer와 CustomAuthorityUtils를 DI 받음
    - JwtTokenizer는 JWT를 검증하고 Claims(토큰에 포함된 정보)를 얻는 데 사용됨
    - CustomAuthorityUtils는 JWT 검증에 성공하면 Authentication 객체에 채울 사용자의 권한을 생성하는 데 사용됨
- (3)의 verifyJws() 메서드는 JWT를 검증하는 데 사용되는 private 메서드
    - (3-1)에서는 request의 header에서 JWT를 얻고 있음
        - (3-1)에서의 JWT는 클라이언트가 response header로 전달받은 JWT를 request header에 추가해서 서버 측에 전송한 것
        - String 클래스의 replace() 메서드를 이용해 “Bearer" 부분을 제거함
        - (3-1)에서 변수명을 jws로 지정한 이유는 서명된 JWT를 JWS(JSON Web Token Signed)라고 부르기 때문
    - (3-2)에서는 JWT 서명(Signature)을 검증하기 위한 Secret Key를 얻음
    - (3-3)에서는 JWT에서 Claims를 파싱 함
        - JWT에서 Claims를 파싱 할 수 있다는 의미는 내부적으로 서명(Signature) 검증에 성공했다는 의미
        - verify() 같은 검증 메서드가 따로 존재하는 것이 아니라 Claims가 정상적으로 파싱이 되면 서명 검증 역시 자연스럽게 성공했다는 것
- (4)의 setAuthenticationToContext() 메서드는 Authentication 객체를 SecurityContext에 저장하기 위한 private 메서드
    - (4-1)에서는 JWT에서 파싱 한 Claims에서 username을 얻음
    - (4-2)에서는 JWT의 Claims에서 얻은 권한 정보를 기반으로 List<GrantedAuthority를 생성함
    - (4-3)에서는 username과 List<GrantedAuthority를 포함한 Authentication 객체를 생성함
    - (4-4)에서는 SecurityContext에 Authentication 객체를 저장함
- 문제없이 JWT의 서명 검증에 성공하고, Security Context에 Authentication을 저장한 뒤에는 (5)와 같이 다음(Next) Security Filter를 호출함
- (6)은 OncePerRequestFilter의 shouldNotFilter()를 오버라이드 한 것으로, 특정 조건에 부합하면(true이면) 해당 Filter의 동작을 수행하지 않고 다음 Filter로 건너뛰도록 해줌
    - (6-1)에서 Authorization header의 값을 얻음
    - (6-2)에서는 Authorization header의 값이 null이거나 Authorization header의 값이 “Bearer”로 시작하지 않는다면 해당 Filter의 동작을 수행하지 않도록 정의함
        - JWT가 Authorization header에 포함되지 않았다면 JWT 자격증명이 필요하지 않은 리소스에 대한 요청이라고 판단하고 다음(Next) Filter로 처리를 넘기는 것
        - 만약 JWT 자격 증명이 필요한 리소스 요청인데 실수로 JWT를 포함하지 않았다 하더라도 이 경우에는 Authentication이 정상적으로 SecurityContext에 저장되지 않은 상태이기 때문에 다른 Security Filter를 거쳐 결국 Exception을 던지게 될 것

<br>

#### 2. SecurityConfiguration 설정 업데이트
- JwtVerificationFilter를 사용하기 위해서는 두 가지 설정을 SecurityConfigruation에 추가해야 함
    - 세션 정책 설정 추가
    - JwtVerificationFilter 추가
- JwtVerificationFilter에서 JWT의 자격 검증에 성공하게 되면 인증된 Authentication 객체를 SecurityContext에 저장함
- stateless한 애플리케이션을 유지하기 위해 세션 유지 시간을 아주 짧게 가져가기 위한(거의 무상태) 설정을 SecurityConfigruation에 추가할 필요가 있음
<br>

```java
import com.codestates.auth.filter.JwtAuthenticationFilter;
import com.codestates.auth.filter.JwtVerificationFilter;
import com.codestates.auth.handler.MemberAuthenticationFailureHandler;
import com.codestates.auth.handler.MemberAuthenticationSuccessHandler;
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.auth.utils.CustomAuthorityUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

import static org.springframework.security.config.Customizer.withDefaults;

/**
 * SessionCreationPolicy 설정 추가
 */
@Configuration
public class SecurityConfiguration {
    private final JwtTokenizer jwtTokenizer;
    private final CustomAuthorityUtils authorityUtils; // 추가

    public SecurityConfigurationV6(JwtTokenizer jwtTokenizer,
                                   CustomAuthorityUtils authorityUtils) {
        this.jwtTokenizer = jwtTokenizer;
        this.authorityUtils = authorityUtils;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers().frameOptions().sameOrigin()
            .and()
            .csrf().disable()
            .cors(withDefaults())
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)  // (1) 추가
            .and()
            .formLogin().disable()
            .httpBasic().disable()
            .apply(new CustomFilterConfigurer())
            .and()
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().permitAll()
            );
        return http.build();
    }

    ...

    public class CustomFilterConfigurer extends AbstractHttpConfigurer<CustomFilterConfigurer, HttpSecurity> {
        @Override
        public void configure(HttpSecurity builder) throws Exception {
            AuthenticationManager authenticationManager = builder.getSharedObject(AuthenticationManager.class);

            JwtAuthenticationFilter jwtAuthenticationFilter = new JwtAuthenticationFilter(authenticationManager, jwtTokenizer);
            jwtAuthenticationFilter.setFilterProcessesUrl("/v11/auth/login");
            jwtAuthenticationFilter.setAuthenticationSuccessHandler(new MemberAuthenticationSuccessHandler());
            jwtAuthenticationFilter.setAuthenticationFailureHandler(new MemberAuthenticationFailureHandler());

            JwtVerificationFilter jwtVerificationFilter = new JwtVerificationFilter(jwtTokenizer, authorityUtils);  // (2) 추가

            builder
                .addFilter(jwtAuthenticationFilter)
                .addFilterAfter(jwtVerificationFilter, JwtAuthenticationFilter.class);   // (3)추가
        }
    }
}
```

<br>

- (1)의 .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)를 통해서 세션을 생성하지 않도록 설정함
    - SessionCreationPolicy의 설정값으로는 총 네 개의 값을 사용할 수 있음
        - SessionCreationPolicy.&#42;ALWAYS&#42;
            - 항상 세션을 생성함
        - SessionCreationPolicy.NEVER
            - 세션을 생성하지 않지만 만약에 이미 생성된 세션이 있다면 사용함
        - SessionCreationPolicy.&#42;IF_REQUIRED&#42;
            - 필요한 경우에만 세션을 생성함
        - SessionCreationPolicy.&#42;STATELESS&#42;
            - 세션을 생성하지 않으며, SecurityContext 정보를 얻기 위해 결코 세션을 사용하지 않음
- (2)에서는 JwtVerificationFilter의 인스턴스를 생성하면서 JwtVerificationFilter에서 사용되는 객체들을 생성자로 DI 해줌
- (3)에서는 JwtVerificationFilter를 JwtAuthenticationFilter 뒤에 추가함
    - JwtVerificationFilter는 JwtAuthenticationFilter에서 로그인 인증에 성공한 후 발급받은 JWT가 클라이언트의 request header(Authorization 헤더)에 포함되어 있을 경우에만 동작함

<br>

### 서버 측 리소스에 역할(Role) 기반 권한 적용

```java
import com.codestates.auth.filter.JwtAuthenticationFilter;
import com.codestates.auth.filter.JwtVerificationFilter;
import com.codestates.auth.handler.MemberAccessDeniedHandler;
import com.codestates.auth.handler.MemberAuthenticationEntryPoint;
import com.codestates.auth.handler.MemberAuthenticationFailureHandler;
import com.codestates.auth.handler.MemberAuthenticationSuccessHandler;
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.auth.utils.CustomAuthorityUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {
    ...

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
            .apply(new CustomFilterConfigurer())
            .and()
            .authorizeHttpRequests(authorize -> authorize
                    .antMatchers(HttpMethod.POST, "/*/members").permitAll()         // (1) 추가
                    .antMatchers(HttpMethod.PATCH, "/*/members/**").hasRole("USER")  // (2) 추가
                    .antMatchers(HttpMethod.GET, "/*/members").hasRole("ADMIN")     // (3) 추가
                    .antMatchers(HttpMethod.GET, "/*/members/**").hasAnyRole("USER", "ADMIN")  // (4) 추가
                    .antMatchers(HttpMethod.DELETE, "/*/members/**").hasRole("USER")  // (5) 추가
                    .anyRequest().permitAll()
            );
        return http.build();
    }
    
    ...
}
```

<br>

- 회원 등록의 경우, 접근 권한 여부와 상관없이 누구나 접근이 가능해야 하므로 (1)과 같이 회원등록에 사용되는 URL(”/members”)과 HTTP Method(여기서는 POST)에 해당된다면 접근을 허용함
- 회원 정보 수정의 경우, (2)와 같이 일반 사용자(USER) 권한만 가진 사용자만 접근이 가능하도록 허용함
    - .antMatchers(HttpMethod.PATCH, "/&#42;/members/&#42;")에서 ‘&#42;&#42;’는 하위 URL로 어떤 URL이 오더라도 매치가 된다는 의미
- 모든 회원 정보의 목록은 (3)과 같이 관리자(ADMIN) 권한을 가진 사용자만 접근이 가능하여야 할 것
- 특정 회원에 대한 정보 조회는 (4)와 같이 일반 사용자(USER)와 관리자(ADMIN) 권한을 가진 사용자 모두 접근이 가능도록 허용함
- 특정 회원을 삭제하는 요청은 (5)와 같이 해당 사용자가 탈퇴 같은 처리를 할 수 있어야 하므로 일반 사용자(USER) 권한만 가진 사용자만 접근이 가능하도록 허용

<br>

### JWT 검증 테스트
1. JWT를 Authorization header에 포함하지 않을 경우
    - JWT를 Authorization header에 포함하지 않은 채 MemberController의 getMember() 핸들러 메서드에 request를 전송한 결과로는 Spring Security에서 403 status를 전달한 것을 볼 수 있음
    - JWT가 Authorization header에 포함되지 않으면 `JwtVerificationFilter`를 건너뛰게 되고, 나머지 Security Filter에서 권한 체크를 하면서 적절한 권한이 부여되지 않았기 때문에 403 status가 전달됨
2. 유효하지 않은 JWT를 Authorization header에 포함할 경우
    - 유효하지 않은 JWT를 Authorization header에 포함하여 MemberController의 getMember() 핸들러 메서드에 request를 전송한 결과로 Spring Security에서 403 status를 전달한 것을 볼 수 있음
    - 유효하지 않은 JWT의 경우 접근 권한에 대한 에러를 나타내는 403 status보다는 JWT의 검증에 실패했기 때문에 자격 증명에 실패한 것과 같으므로 UNAUTHORIZED를 의미하는 401 status가 더 적절함
3. 권한이 부여되지 않은 리소스에 request를 전송할 경우
    - USER 권한만 부여된 사용자의 JWT를 Authorization header에 추가하고, ADMIN 권한에만 접근이 허용된 MemberController의 getMembers() 핸들러 메서드에 request를 전송할 때의 결과로는, JwtVerificationFilter에서 JWT의 자격 증명은 정상적으로 수행되었지만 ADMIN 권한이 없는 사용자이므로 403 status가 전달됨

<br>

### 예외 처리

#### 1. JwtVerificationFilter에 예외 처리 로직 추가
- JwtVerificationFilter의 경우, 클라이언트로부터 전달받은 JWT의 Claims를 얻는 과정에서 내부적으로 JWT에 대한 서명(Signature)을 검증함
<br>

```java
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.auth.utils.CustomAuthorityUtils;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.security.SignatureException;
import lombok.extern.slf4j.Slf4j;
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

public class JwtVerificationFilter extends OncePerRequestFilter {
    ...

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
    
    ...
}
```

<br>

- (1)과 같이 try~catch 문으로 특정 예외 타입의 Exception이 catch 되면 해당 Exception을 request.setAttribute("exception", Exception 객체)와 같이 HttpServletRequest의 애트리뷰트(Attribute)로 추가됨
- 추가된 애트리뷰트는 AuthenticationEntryPoint에서 사용할 수 있음
- JwtVerificationFilter 예외 처리의 키포인트는 일반적으로 알고 있는 예외 처리 방식과는 다르게 Exception을 catch한 후에 Exception을 다시 throw 한다든지 하는 처리를 하지 않고, 단순히 request.setAttribute()를 설정하는 일밖에 하지 않는다는 것
- 이런 식으로 예외 처리를 하게 되면 예외 발생 시 SecurityContext에 클라이언트의 인증 정보(Authentication 객체)가 저장되지 않음
- SecurityContext에 클라이언트의 인증 정보(Authentication 객체)가 저장되지 않은 상태로 다음(next) Security Filter 로직을 수행하다 보면 결국에는 Filter 내부에서 AuthenticationException이 발생하게 되고, 이 AuthenticationException은 바로 아래에서 설명하는 AuthenticationEntryPoint가 처리하게 됨
- SecurityContext에 클라이언트의 인증 정보가 채워지지 않은 상태에서 Security Filter 로직을 수행하게 되면 Security Filter 체인의 Filter 내부에서 AuthenticationException이 발생함

<br>

#### 2. AuthenticationEntryPoint 구현
- AuthenticationEntryPoint는 SignatureException, ExpiredJwtException 등 Exception 발생으로 인해 SecurityContext에 Authentication이 저장되지 않을 경우 등 AuthenticationException이 발생할 때 호출되는 핸들러 같은 역할을 함
<br>

```java
// AuthenticationEntryPoint를 구현한 MemberAuthenticationEntryPoint
import com.codestates.auth.utils.ErrorResponder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
public class MemberAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        Exception exception = (Exception) request.getAttribute("exception");
        ErrorResponder.sendErrorResponse(response, HttpStatus.UNAUTHORIZED);

        logExceptionMessage(authException, exception);
    }

    private void logExceptionMessage(AuthenticationException authException, Exception exception) {
        String message = exception != null ? exception.getMessage() : authException.getMessage();
        log.warn("Unauthorized error happened: {}", message);
    }
}
```

<br>

- MemberAuthenticationEntryPoint 클래스는 인증 과정에서 AuthenticationException 이 발생할 경우 호출되며, 처리하고자 하는 로직을 commence() 메서드에 구현함
- 인증 과정에서 AuthenticationException 발생하면 ErrorResponse를 생성해서 클라이언트에게 전송함

<br>

```java
// ErrorResponder
import com.codestates.response.ErrorResponse;
import com.google.gson.Gson;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class ErrorResponder {
    public static void sendErrorResponse(HttpServletResponse response, HttpStatus status) throws IOException {
        Gson gson = new Gson();
        ErrorResponse errorResponse = ErrorResponse.of(status);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(status.value());
        response.getWriter().write(gson.toJson(errorResponse, ErrorResponse.class));
    }
}
```

<br>

- ErrorResponder 클래스는 ErrorResponse를 출력 스트림으로 생성하는 역할을 함

<br>

#### 3. AccessDeniedHandler 구현
- AccessDeniedHandler는 인증에는 성공했지만 해당 리소스에 대한 권한이 없으면 호출되는 핸들러
<br>

```java
import com.codestates.auth.utils.ErrorResponder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Component
public class MemberAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        ErrorResponder.sendErrorResponse(response, HttpStatus.FORBIDDEN);
        log.warn("Forbidden error happened: {}", accessDeniedException.getMessage());
    }
}
```

<br>

- MemberAccessDeniedHandler클래스는 요청한 리소스에 대해 적절한 권한이 없으면 호출되는 핸들러로서, 처리하고자 하는 로직을 handle() 메서드에 구현함
- 적절한 권한인지 확인하는 과정에서 AccessDeniedException이 발생하면 ErrorResponse를 생성해서 클라이언트에게 전송함

<br>

#### 4. SecurityConfiguration에 AuthenticationEntryPoint 및 AccessDeniedHandler 추가

```java
import com.codestates.auth.filter.JwtAuthenticationFilter;
import com.codestates.auth.filter.JwtVerificationFilter;
import com.codestates.auth.handler.MemberAccessDeniedHandler;
import com.codestates.auth.handler.MemberAuthenticationEntryPoint;
import com.codestates.auth.handler.MemberAuthenticationFailureHandler;
import com.codestates.auth.handler.MemberAuthenticationSuccessHandler;
import com.codestates.auth.jwt.JwtTokenizer;
import com.codestates.auth.utils.CustomAuthorityUtils;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.Arrays;

import static org.springframework.security.config.Customizer.withDefaults;

/**
 * authenticationEntryPoint와 accessDeniedHandler 추가
 */
@Configuration
public class SecurityConfiguration {
   ...

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
            .exceptionHandling()
            .authenticationEntryPoint(new MemberAuthenticationEntryPoint())  // (1) 추가
            .accessDeniedHandler(new MemberAccessDeniedHandler())            // (2) 추가
            .and()
            .apply(new CustomFilterConfigurer())
            .and()
            .authorizeHttpRequests(authorize -> authorize
                    .antMatchers(HttpMethod.POST, "/*/members").permitAll()
                    .antMatchers(HttpMethod.PATCH, "/*/members/**").hasRole("USER")
                    .antMatchers(HttpMethod.GET, "/*/members").hasRole("ADMIN")
                    .antMatchers(HttpMethod.GET, "/*/members/**").hasAnyRole("USER", "ADMIN")
                    .antMatchers(HttpMethod.DELETE, "/*/members/**").hasRole("USER")
                    .anyRequest().permitAll()
            );
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("*"));
        configuration.setAllowedMethods(Arrays.asList("GET","POST", "PATCH", "DELETE"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("\/**", configuration);         // 주의 사항: 포스팅 표시 오류로 인해 '/**'를 '\/**'로 표기했으나 실제 코드 구현 시에는 '\(역슬래시)'를 빼야 함
        return source;
    }

    public class CustomFilterConfigurer extends AbstractHttpConfigurer<CustomFilterConfigurer, HttpSecurity> {
        @Override
        public void configure(HttpSecurity builder) throws Exception {
            AuthenticationManager authenticationManager = builder.getSharedObject(AuthenticationManager.class);

            JwtAuthenticationFilter jwtAuthenticationFilter = new JwtAuthenticationFilter(authenticationManager, jwtTokenizer);
            jwtAuthenticationFilter.setFilterProcessesUrl("/v11/auth/login");
            jwtAuthenticationFilter.setAuthenticationSuccessHandler(new MemberAuthenticationSuccessHandler());
            jwtAuthenticationFilter.setAuthenticationFailureHandler(new MemberAuthenticationFailureHandler());

            JwtVerificationFilter jwtVerificationFilter = new JwtVerificationFilter(jwtTokenizer, authorityUtils);

            builder
                .addFilter(jwtAuthenticationFilter)
                .addFilterAfter(jwtVerificationFilter, JwtAuthenticationFilter.class);
        }
    }
}
```

<br>

- SecurityConfiguration에서는 (1), (2)와 같이 MemberAuthenticationEntryPoint와 MemberAccessDeniedHandler가 추가함