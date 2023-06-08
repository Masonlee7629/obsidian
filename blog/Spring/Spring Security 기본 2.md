태그: #Spring 
연결문서: [Spring Security 기본 3](Spring%20Security%20기본%203.md)

# Spring Secutiry
## Spring Security 적용 방법
1.  의존 라이브러리 추가(build.gradle)

```java
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-security'
}
```

Spring Boot 기반의 애플리케이션에 Spring Security를 적용하기 위해서는 `spring-boot-starter-security` 스타터를 추가하면 된다.  
<br>

Spring Security는 디폴트 로그인 정보를 제공해 준다.
-   Username : user
-   Password : 애플리케이션 실행 시 출력되는 로그에서 확인 가능

```java
...

Using generated security password: **32593900-2c36-4cdb-9d15-ce8e5049481f**   // (1)
```

<br>

2. Spring Security Configuration 적용
Spring Security Configuration을 적용하면 원하는 인증 방식과 웹 페이지에 대한 접근 권한을 설정할 수 있다.
<br>

SecurityConfiguration 클래스에 생성하고, `@Configuration` 애너테이션을 추가해준다.
그 후, Spring Security에서 지원하는 인증과 권한 부여 설정을 한다.

- Spring Security Configuration의 기본 구조

```java
package com.codestates.config;

import org.springframework.context.annotation.Configuration;

@Configuration
public class SecurityConfiguration {
    
}
```

- InMemory User로 인증하기

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
public class SecurityConfiguration {
    @Bean
    public UserDetailsManager userDetailsService() {
        // (1)
        UserDetails userDetails =
                User.withDefaultPasswordEncoder()    // (1-1)
                        .username("mason@gmail.com") // (1-2)
                        .password("1234")            // (1-3)
                        .roles("USER")               // (1-4)
                        .build();
        // (2)
        return new InMemoryUserDetailsManager(userDetails);
    }
}
```

위 코드는 애플리케이션이 실행된 상태에서 사용자 인증을 위한 계정 정보를 메모리상에 고정된 값으로 설정한 예시이다.
<br>

- (1)의 `UserDetails` 인터페이스는 인증된 사용자의 핵심 정보를 포함하고 있으며, `UserDetails` 구현체인 (2)의 `User` 클래스를 이용해서 사용자의 인증 정보를 생성한다.
    - `withDefaultPasswordEncoder()` : 디폴트 패스워드 인코더를 이용해 사용자 패스워드를 암호화한다.
        - **Deprecated** 상태
            - API 문서에서 흔히 볼 수 있는 Deprecated라는 용어는 해당 API가 향후 버전에서는 더 이상 사용되지 않고 제거될 수 있다는 의미이기 때문에 Deprecated라고 표시된 API 사용은 권장되지 않는다.
            - `withDefaultPasswordEncoder()` 메서드의 Deprecated는 특이하게도 향후 버전에서 제거됨을 의미하기보다는 **Production 환경에서 인증을 위한 사용자 정보를 고정해서 사용하지 말라는 경고의 의미를 나타내고 있는 것**이니 반드시 테스트 환경이나 데모 환경에서만 사용할 수 있도록 해야 한다.
    - `username()` : 사용자의 usrname을 설정한다.
    - `password()` : 사용자의 password를 설정한다.
    - `roles()` : 사용자의 Role(역할)을 지정한다.
- Spring Security에서는 사용자의 핵심 정보를 포함한 `UserDetails`를 관리하는 `UserDetailsManager`라는 인터페이스를 제공한다.
    - 현재 메모리상에서 `UserDetails`를 관리하므로 `InMemoryUserDetailsManager`라는 구현체를 사용한다.
    - (2)와 같이 `new InMemoryUserDetailsManager(userDetails)`를 통해 `UserDetailsManager` 객체를 Bean으로 등록하면 Spring에서는 해당 Bean이 가지고 있는 사용자의 인증 정보가 클라이언트의 요청으로 넘어올 경우 정상적인 인증 프로세스를 수행한다.
- InMemory 방식은 **테스트 환경 또는 데모 환경에서만 유용하게 사용**할 수 있는 방식이다.

<br>

3. HTTP 보안 구성 기본

```java
@Configuration
public class SecurityConfiguration {
    // (1)
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // HttpSecurity를 통해 HTTP 요청에 대한 보안 설정을 구성한다.
        ...
        ...
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user =
                User.withDefaultPasswordEncoder()
                        .username("mason@gmail.com")
                        .password("1234")
                        .roles("USER")
                        .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

위 코드는 Spring Security에서 HTTP 보안을 설정하기 위한 기본 코드이다.
<br>

(1)과 같이 HttpSecurity를 파라미터로 가지고, SecurityFilterChain을 리턴하는 형태의 메서드를 정의하면 HTTP 보안 설정을 구성할 수 있다.<br>

파라미터로 지정한 HttpSecurity는 HTTP 요청에 대한 보안 설정을 구성하기 위한 핵심 클래스이다.

<br>

4. 커스텀 로그인 페이지 지정하기

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()                 // (1)
            .formLogin()                      // (2)
            .loginPage("/auths/login-form")   // (3)
            .loginProcessingUrl("/process_login")    // (4)
            .failureUrl("/auths/login-form?error")   // (5)
            .and()                                   // (6)
            .authorizeHttpRequests()                     // (7)
            .anyRequest()                            // (8)
            .permitAll();                            // (9)

        return http.build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user =
                User.withDefaultPasswordEncoder()
                        .username("mason@gmail.com")
                        .password("1234")
                        .roles("USER")
                        .build();
        return new InMemoryUserDetailsManager(user);
    }
}
```

위 코드는 Spring Security의 보안 구성 중에서 커스텀 로그인 페이지를 사용하기 위한 최소한의 설정만 추가한 코드이다.

- (1)은 **CSRF(Cross-Site Request Forgery) 공격에 대한 Spring Security에 대한 설정을 비활성화**한다.
    - Spring Security는 기본적으로 아무 설정을 하지 않으면 csrf() 공격을 방지하기 위해 클라이언트로부터 CSRF Token을 수신 후, 검증한다. 현재 로컬 환경에서의 Spring Security에 대한 예시로 진행하므로, CSRF 공격에 대한 설정이 필요하지 않다.
    - 만약 `csrf().disable()` 설정을 하지 않는다면 403 에러로 인해 정상적인 접속이 불가능하다.
- (2)의 `formLogin()`은 기본적인 인증 방법을 폼 로그인 방식으로 지정한다.
- (3)의 `loginPage("/auths/login-form")` 메서드를 통해 커스텀 로그인 페이지를 사용하도록 설정한다.
    - `loginPage()`의 파라미터인 `"/auths/login-form"`은 `AuthController`의 `loginForm()` 핸들러 메서드에 요청을 전송하는 요청 URL이다.
- (4)의 `loginProcessingUrl("/process_login")` 메서드를 통해 로그인 인증 요청을 수행할 요청 URL을 지정한다.
    - `loginProcessingUrl()`의 파라미터인 `"/process_login"`은 커스텀 로그인 페이지에서 form 태그의 action 속성에 지정한 URL과 동일하다.
- (5)의 `failureUrl("/auths/login-form?error")` 메서드를 통해 로그인 인증에 실패할 경우 어떤 화면으로 리다이렉트 할 것인가를 지정한다.
- (6)의 `and()` 메서드를 통해 Spring Security 보안 설정을 메서드 체인 형태로 구성할 수 있다.
- (7), (8), (9)를 통해서 클라이언트의 요청에 대해 접근 권한을 확인한다. 즉, 접근을 허용할 지 여부를 결정한다.
    - (7)의 `authorizeHttpRequests()` 메서드는 클라이언트의 요청이 들어오면 접근 권한을 확인하겠다고 정의한다.
    - (8)과 (9)의 `anyRequest().permitAll()`는 클라이언트의 모든 요청에 대해 접근을 허용한다.

<br>

5. request URI에 접근 권한 부여

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .formLogin()
            .loginPage("/auths/login-form")
            .loginProcessingUrl("/process_login")
            .failureUrl("/auths/login-form?error")
            .and()
            .exceptionHandling().accessDeniedPage("/auths/access-denied")   // (1)
            .and()
            .authorizeHttpRequests(authorize -> authorize                  // (2)
                    .antMatchers("/orders/**").hasRole("ADMIN")        // (2-1)
                    .antMatchers("/members/my-page").hasRole("USER")   // (2-2)
                    .antMatchers("⁄**").permitAll()                    // (2-3)
            );
        return http.build();
    }
```

- (1)에선 `exceptionHandling().accessDeniedPage("/auths/access-denied")`를 통해 권한이 없는 사용자가 특정 request URI에 접근할 경우 발생하는 403(Forbidden) 에러를 처리하기 위한 페이지를 설정한다.
    - `exceptionHandling()` 메서드는 메서드의 이름 그대로 Exception을 처리하는 기능을 하며, 리턴하는 `ExceptionHandlingConfigurer` 객체를 통해 구체적인 Exception 처리할 수 있다.
    - `accessDeniedPage()` 메서드는 403 에러 발생 시, 파라미터로 지정한 URL로 리다이렉트 되도록 해준다.
- (2)의 `authorizeHttpRequests()` 메서드는 람다 표현식을 통해 request URI에 대한 접근 권한을 부여할 수 있다.
    - `antMatchers()` 메서드는 이름 그대로 ant라는 빌드 툴에서 사용되는 Path Pattern을 이용해서 매치되는 URL을 표현한다.
    - (2-1)의 `.antMatchers("/orders/**").hasRole("ADMIN")`은 `ADMIN` Role을 부여받은 사용자만 `/orders`**로 시작하는 모든 URL에 접근할 수 있다는 의미**이다.
        - `/orders/**`에서 `**`은 `/orders`로 시작하는 모든 하위 URL을 포함한다.
        - 만약, `/orders/*`라는 URL을 지정했다면 `/orders`의 하위 URL의 depth가 1인 URL만 포함한다.
    - (2-2)의 `antMatchers("/members/my-page").hasRole("USER")`은 USER Role을 부여받은 사용자만 `/members/my-page` URL에 접근할 수 있음을 의미한다.
    - (2-3)의 `.antMatchers("/**").permitAll()`은 앞에서 지정한 URL 이외의 나머지 모든 URL은 Role에 상관없이 접근이 가능함을 의미한다.
        - 만약 순서를 바꿔 `antMatchers("/**").permitAll()`이 제일 앞에 위치하면 Spring Security에서는 Role에 상관없이 모든 request URL에 대한 접근을 허용하기 때문에 다음에 추가된 접근 권한을 부여하는 메서드는 제 기능을 하지 못하게 된다.
        - request URL 설정 오류를 방지하기 위해 항상 더 구체적인 URL 경로부터 접근 권한을 부여한 다음 덜 구체적인 URL 경로에 접근 권한을 부여해야 한다.

<br>

6. 관리자 권한을 가진 사용자 정보 추가

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfiguration {
    ...
    ...

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user =
                User.withDefaultPasswordEncoder()
                        .username("mason@gmail.com")
                        .password("1234")
                        .roles("USER")
                        .build();

        // (1)
        UserDetails admin =
                User.withDefaultPasswordEncoder()
                        .username("admin@gmail.com")
                        .password("4321")
                        .roles("ADMIN")
                        .build();

        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

<br>

7. 사용자 로그아웃 기능 설정하기

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .formLogin()
            .loginPage("/auths/login-form")
            .loginProcessingUrl("/process_login")
            .failureUrl("/auths/login-form?error")
            .and()
            .logout()                        // (1)
            .logoutUrl("/logout")            // (2)
            .logoutSuccessUrl("/")  // (3)
            .and()
            .exceptionHandling().accessDeniedPage("/auths/access-denied")
            .and()
            .authorizeHttpRequests(authorize -> authorize
                    .antMatchers("/orders/**").hasRole("ADMIN")
                    .antMatchers("/members/my-page").hasRole("USER")
                    .antMatchers("⁄**").permitAll()
            );
        return http.build();
    }

    ...
}
```

이전의 SecurityConfiguration 클래스에 로그아웃을 위한 설정이 추가된 코드이다.

- 로그아웃에 대한 추가 설정을 위해서는 (1)과 같이 `logout()`을 먼저 호출해야 한다. `logout()` 메서드는 로그아웃 설정을 위한 `LogoutConfigurer`를 리턴한다.
- (2)에서는 `logoutUrl("/logout")`을 통해 사용자가 로그아웃을 수행하기 위한 request URL을 지정한다.
- (3)에서는 로그아웃을 성공적으로 수행한 이후 리다이렉트 할 URL을 지정한다.