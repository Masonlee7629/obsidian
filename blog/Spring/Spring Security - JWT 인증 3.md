태그: #Spring 
연결문서: [Spring Security - JWT 인증 4](Spring%20Security%20-%20JWT%20인증%204.md)

# Spring Security에서의 JWT 인증

## JWT 적용을 위한 사전 작업

### 의존 라이브러리 추가

```java
// build.gradle

...

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	implementation 'org.mapstruct:mapstruct:1.5.2.Final'
	annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.2.Final'
	implementation 'org.springframework.boot:spring-boot-starter-mail'
	implementation 'com.google.code.gson:gson'

	implementation 'org.springframework.boot:spring-boot-starter-security' // (1)

  // (2) JWT 기능을 위한 jjwt 라이브러리
	implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
	runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
	runtimeOnly	'io.jsonwebtoken:jjwt-jackson:0.11.5'
}

...
```

<br>

### 애플리케이션 실행
- 애플리케이션을 실행한 후, http://localhost:8080으로 접속하면 Spring Security에서 제공하는 로그인 화면이 보여야 함
- 로그인 화면에서 Username과 Password를 입력해야 함
    - Username : user
    - Password : intelliJ 로그에 출력되는 Using generated security password: ... ... 부분을 복사 및 붙여넣기 함
        - intelliJ 로그에 출력되는 Password는 애플리케이션을 실행할 때마다 바뀜
- 정상적으로 로그인에 성공했다면 Whitelabel Error Page가 출력됨
    - 로그인 인증에는 성공했지만 기본적으로 Controller 같은 엔드포인트가 없어서 발생하는 404 에러로 Spring Security에 문제가 있어서 발생한 에러는 아님

<br>

### SecurityConfiguration 추가
- Spring Security를 이용한 보안 강화를 위해 최소한의 보안 구성을 함
<br>

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers().frameOptions().sameOrigin() // (1)
            .and()
            .csrf().disable()        // (2)
            .cors(withDefaults())    // (3)
            .formLogin().disable()   // (4)
            .httpBasic().disable()   // (5)
            .authorizeHttpRequests(authorize -> authorize
                    .anyRequest().permitAll()                // (6)
            );
        return http.build();
    }

    // (7)
    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    // (8)
    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("*"));   // (8-1)
        configuration.setAllowedMethods(Arrays.asList("GET","POST", "PATCH", "DELETE"));  // (8-2)

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();   // (8-3)
        source.registerCorsConfiguration("\/**", configuration);      // (8-4)     주의 사항: 포스트 표시 오류를 막고자 '/**'를 '\/**'로 표기했으니 실제 코드 구현 시에는 '\(역슬래시)'를 빼야 함
        return source;
    }
}
```

<br>

- H2 웹 콘솔의 화면 자체가 내부적으로 태그를 사용하고 있기 때문에 개발 환경에서는 H2 웹 콘솔을 정상적으로 사용할 수 있도록 (1)과 같이 .frameOptions().sameOrigin() 을 추가함
    - .frameOptions().sameOrigin() 을 호출하면 동일 출처로부터 들어오는 request만 페이지 렌더링을 허용함
- (2)와 같이 CSRF(Cross-Site Request Forgery) 공격에 대한 Spring Security에 대한 설정을 비활성화함
    - 로컬 환경에서는 CSRF 공격에 대한 설정이 필요하지 않음
    - 만약, csrf().disable() 설정하지 않는다면 403 에러로 인해 정상적인 접속이 불가능함
- (3)에서는 CORS 설정을 추가함
    - .cors(withDefaults()) 일 경우, corsConfigurationSource라는 이름으로 등록된 Bean을 이용함
    - ORS를 처리하는 가장 쉬운 방법은 CorsFilter를 사용하는 것인데 CorsConfigurationSource Bean을 제공함으로써 CorsFilter를 적용할 수 있음
- CSR(Client Side Rendering) 방식에서 주로 사용하는 JSON 포맷으로 Username과 Password를 전송하는 방식을 사용할 것이므로 (4)와 같이 폼 로그인 방식을 비활성화함
- HTTP Basic 인증은 request를 전송할 때마다 Username/Password 정보를 HTTP Header에 실어서 인증을 하는 방식으로 현재 사용하지 않을 것으므로 (5)와 같이 HTTP Basic 인증 방식을 비활성화함
    - 폼 로그인과 HTTP Basic 인증을 disable하면 해당 인증과 관련된 Security Filter(UsernamePasswordAuthenticationFilter, BasicAuthenticationFilter 등)가 비활성화됨
- (6)에서는 JWT를 적용하기 전이므로 우선은 모든 HTTP request 요청에 대해서 접근을 허용하도록 설정함
    - JWT 적용 후, URL 별로 적절한 권한을 적용할 예정
- (7)에서 PasswordEncoder Bean 객체를 생성함
- (8)에서는 CorsConfigurationSource Bean 생성을 통해 구체적인 CORS 정책을 설정함
    - (8-1)에서 setAllowedOrigins()을 통해 모든 출처(Origin)에 대해 스크립트 기반의 HTTP 통신을 허용하도록 설정하며, 이 설정은 운영 서버 환경에서 요구사항에 맞게 변경이 가능함
    - (8-2)에서는 setAllowedMethods()를 통해 파라미터로 지정한 HTTP Method에 대한 HTTP 통신을 허용함
    - (8-3)에서는 CorsConfigurationSource 인터페이스의 구현 클래스인 UrlBasedCorsConfigurationSource 클래스의 객체를 생성함
    - (8-4)에서는 모든 URL에 앞에서 구성한 CORS 정책(CorsConfiguration)을 적용함

<br>

### 회원 가입 로직 수정
- Spring Security 적용으로 회원 등록 시, 회원의 인증과 관련된 정보(패스워드. 사용자 권한)를 추가하기 위함
<br>

1. Dto 클래스에 패스워드 필드 추가
    - 실제 서비스에서는 사용자가 회원 가입 시, 패스워드를 한 번만 입력하는 것이 아니라 사용자가 입력한 패스워드가 맞는지 재확인하기 위해 패스워드 입력 확인 필드가 추가로 존재하는 경우가 대부분이고, 입력한 두 패스워드가 일치하는지를 검증하는 로직이 필요함
    - 패스워드의 생성 규칙(대/소문자, 패스워드 길이, 특수 문자 포함 여부 등)에 대한 유효성 검증도 실시함

```java
public class MemberDto {
    @Getter
    @AllArgsConstructor
    public static class Post {
        @NotBlank
        @Email
        private String email;

        // (1) 패스워드 필드 추가
        @NotBlank
        private String password;

        @NotBlank(message = "이름은 공백이 아니어야 합니다.")
        private String name;

        @Pattern(regexp = "^010-\\d{3,4}-\\d{4}$",
                message = "휴대폰 번호는 010으로 시작하는 11자리 숫자와 '-'로 구성되어야 합니다.")
        private String phone;
    }

    ...
}
```

<br>

2. 엔티티 클래스에 패스워드 필드 추가
<br>

```java
@NoArgsConstructor
@Getter
@Setter
@Entity
public class Member extends Auditable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberId;

    @Column(nullable = false, updatable = false, unique = true)
    private String email;

    // (1) 추가
    @Column(length = 100, nullable = false)
    private String password;

    ...
    // (2) 추가
    @ElementCollection(fetch = FetchType.EAGER)
    private List<String> roles = new ArrayList<>();

    ...
}
```

- (1)과 같이 Member 엔티티 클래스에 패스워드 필드를 추가
    - password는 암호화되어 저장되기 때문에 열의 길이는 100으로 지정하였으며, 패스워드 입력 규칙에 따라서 password 길이는 달라질 수 있음
- (2)에서는 @ElementCollection 애너테이션을 이용해 사용자 등록 시, 사용자의 권한을 등록하기 위한 권한 테이블을 생성함

<br>

3. 사용자 등록 시, 패스워드와 사용자 권한 저장
<br>

```java
@Transactional
@Service
public class MemberService {
    private final MemberRepository memberRepository;
    private final ApplicationEventPublisher publisher;

    // (1) 추가
    private final PasswordEncoder passwordEncoder;
    private final CustomAuthorityUtils authorityUtils;

    // (2) 생성자 DI용 파라미터 추가
    public MemberService(MemberRepository memberRepository,
                         ApplicationEventPublisher publisher,
                         PasswordEncoder passwordEncoder,
                         CustomAuthorityUtils authorityUtils) {
        this.memberRepository = memberRepository;
        this.publisher = publisher;
        this.passwordEncoder = passwordEncoder;
        this.authorityUtils = authorityUtils;
    }

    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());

        // (3) 추가: Password 암호화
        String encryptedPassword = passwordEncoder.encode(member.getPassword());
        member.setPassword(encryptedPassword);

        // (4) 추가: DB에 User Role 저장
        List<String> roles = authorityUtils.createRoles(member.getEmail());
        member.setRoles(roles);

        Member savedMember = memberRepository.save(member);

        publisher.publishEvent(new MemberRegistrationApplicationEvent(this, savedMember));
        return savedMember;
    }

    ...
}
```

- (1)과 (2)에서는 PasswordEncoder와 CustomAuthorityUtils 클래스를 DI 받도록 필드를 추가함
- (3)에서는 패스워드를 단방향 암호화함
- (4)에서는 등록하는 사용자의 권한 정보를 생성함