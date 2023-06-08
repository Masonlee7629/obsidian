태그: #Spring 
연결문서: [Spring Security 기본 4](Spring%20Security%20기본%204.md)

# Spring Security

## 데이터베이스 연동 없는 로그인 인증
- InMemory User
    - InMemory User 등록을 위한 코드
    - userDetailsService() 메서드에서 설정한 두 개의 User가 InMemory User로 등록
    - 데이터베이스를 연동하지 않고, 메모리에 등록하는 방식

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.provisioning.UserDetailsManager;

@Configuration
public class SecurityConfiguration {
    ...
    ...

    @Bean
    public UserDetailsManager userDetailsService() {
        UserDetails user =
                User.withDefaultPasswordEncoder()
                        .username("mason@gmail.com")
                        .password("1111")
                        .roles("USER")
                        .build();

        UserDetails admin =
                User.withDefaultPasswordEncoder()
                        .username("admin@gmail.com")
                        .password("2222")
                        .roles("ADMIN")
                        .build();

        return new InMemoryUserDetailsManager(user, admin);
    }
}
```

<br>

### 회원 가입 톰을 통한 InMemory User 등록
1. Password Encoder Bean 등록
    - Password Encoder는 Spring Security에서 제공하는 패스워드 암호화 기능을 제공하는 컴포넌트
    - 회원 가입 폼에서 전달 받은 패스워드는 InMemory User로 등록되기 전에 암호화되어야 함
    - Password Encoder는 다양한 암호화 방식을 제공하며, Spring Security에서 지원하는 Password Encoder의 디폴트 암호화 알고리즘은 bcrypt
    - (1-1)의 PasswordEncoderFactories.createDelegatingPasswordEncoder();를 통해 DelegatingPasswordEncoder를 먼저 생성하는데, 이 DelegatingPasswordEncoder가 실질적으로 PasswordEncoder 구현 객체를 생성해줌

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.provisioning.UserDetailsManager;

@Configuration
public class SecurityConfiguration {
    ...
    ...

    @Bean
    public UserDetailsManager userDetailsService() {
        UserDetails user =
                User.withDefaultPasswordEncoder()
                        .username("mason@gmail.com")
                        .password("1111")
                        .roles("USER")
                        .build();

        UserDetails admin =
                User.withDefaultPasswordEncoder()
                        .username("admin@gmail.com")
                        .password("2222")
                        .roles("ADMIN")
                        .build();

        return new InMemoryUserDetailsManager(user, admin);
    }

    // (1)
    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();  // (1-1)
    }
}
```

<br>

2. MemberService Bean 등록을 위한 JavaConfiguration 구성
    - (1)에서 MemberService 인터페이스의 구현체인 InMemoryMemberService 클래스의 Bean 객체를 생성함
        - InMemoryMemberService 클래스는 데이터베이스 연동 없이 메모리에 Spring Security의 User를 등록해야 하므로 UserDetailsManager 객체가 필요함
        - User 등록 시, 패스워드를 암호화한 후에 등록해야 하므로 Spring Security에서 제공하는 PasswordEncoder 객체가 필요
        - 두 객체를 InMemoryMemberService 객체 생성 시, DI 해 줌

```java
// 예시용 MemberService 인터페이스 구현 코드
public interface MemberService {
    Member createMember(Member member);
}
```

<br>

```java
// InMemory User 등록을 위한 InMemoryMemberService 클래스
public class InMemoryMemberService implements MemberService {
    public Member createMember(Member member) {

        return null;
    }
}
```

<br>

```java
// JavaConfiguration 구성
import com.codestates.member.InMemoryMemberService;
import com.codestates.member.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.UserDetailsManager;

@Configuration
public class JavaConfiguration {
    // (1)
    @Bean
    public MemberService inMemoryMemberService(UserDetailsManager userDetailsManager, 
                                               PasswordEncoder passwordEncoder) {
        return new InMemoryMemberService(userDetailsManager, passwordEncoder);
    }
}
```

<br>

3. InMemoryMemberService 구현
    - InMemoryMemberService 클래스는 MemberService 인터페이스를 구현하는 구현 클래스임으로 (1)과 같이 implements MemberService를 지정함
    - (2)에서는 UserDetailsManager와 PasswordEncoder를 DI 받음
        - UserDetailsManager는 Spring Security의 User를 관리하는 관리자 역할을 합니다. 우리가 SecurityConfiguration에서 Bean으로 등록한 UserDetailsManager는 InMemoryUserDetailsManager이므로 여기서 DI 받은 UserDetailsManager 인터페이스의 하위 타입은InMemoryUserDetailsManager
        - PasswordEncoder는 Spring Security User를 등록할 때 패스워드를 암호화해 주는 클래스로 Spring Security 5에서는 InMemory User도 패스워드의 암호화가 필수이며, 따라서 DI 받은 PasswordEncoder를 이용해 User의 패스워드를 암호화 해야 함
    - Spring Security에서 User를 등록하기 위해서는 해당 User의 권한(Authority)을 지정해 주어야 하므로, (3)의 createAuthorities(Member.MemberRole.ROLE_USER.name());를 이용해 User의 권한 목록을 List&#60;GrantedAuthority&#62;로 생성함
        - Spring Security에서는 SimpleGrantedAuthority를 사용해 Role 베이스 형태의 권한을 지정할 때 'ROLE_ + 권한 명' 형태로 지정해 주어야 하며, 그렇지 않을 경우 적절한 권한 매핑이 이루어지지 않음
        - (3-1)에서는 Java의 Stream API를 이용해 생성자 파라미터로 해당 User의 Role을 전달하면서 SimpleGrantedAuthority 객체를 생성한 후, List&#60;SimpleGrantedAuthority&#62; 형태로 리턴해줌
    - (4)에서는 PasswordEncoder를 이용해 등록할 User의 패스워드를 암호화함
    - (5)에서는 Spring Security User로 등록하기 위해 UserDetails를 생성함
        - Spring Security에서는 Spring Security에서 관리하는 User 정보를 UserDetails로 관리함
    - (6)에서는 UserDetailsManager의 createUser() 메서드를 이용해서 User를 등록함

```java
import com.codestates.auth.utils.AuthorityUtils;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.UserDetailsManager;

import java.util.List;

public class InMemoryMemberService implements MemberService {  // (1)
    private final UserDetailsManager userDetailsManager;
    private final PasswordEncoder passwordEncoder;

    // (2)
    public InMemoryMemberService(UserDetailsManager userDetailsManager, PasswordEncoder passwordEncoder) {
        this.userDetailsManager = userDetailsManager;
        this.passwordEncoder = passwordEncoder;
    }

    public Member createMember(Member member) {
        // (3)
        List<GrantedAuthority> authorities = createAuthorities(Member.MemberRole.ROLE_USER.name());

        // (4)
        String encryptedPassword = passwordEncoder.encode(member.getPassword());

        // (5)
        UserDetails userDetails = new User(member.getEmail(), encryptedPassword, authorities);

        // (6)
        userDetailsManager.createUser(userDetails);

        return member;
    }

    private List<GrantedAuthority> createAuthorities(String... roles) {
        // (3-1)
        return Arrays.stream(roles)
                .map(role -> new SimpleGrantedAuthority(role))
                .collect(Collectors.toList());
    }
}
```

<br>

---

<br>

## 데이터베이스 연동을 통한 로그인 인증

### Custom UserDetailsService를 사용하는 방법
 - Spring Security에서는 User의 인증 정보를 테이블에 저장하고, 테이블에 저장된 인증 정보를 이용해 인증 프로세스를 진행할 수 있는 몇 가지 방법이 존재하는데 그중 한 가지 방법이 바로 Custom UserDetailsService를 이용하는 방법
 - 일반적으로 Spring Security에서는 인증을 시도하는 주체를 User(비슷한 의미로 Principal도 있음)라고 부름
 - Principal은 User의 더 구체적인 정보를 의미하며, 일반적으로 Spring Security에서의 Username을 의미함
<br>

1. SecurityConfiguration의 설정 변경 및 추가
    - InMemory User를 위한 설정들을 제거해야 함
    - (1)은 웹 브라우저에서 H2 웹 콘솔을 정상적으로 사용하기 위한 설정
        - Spring Security에서는 Clickjacking 공격을 막기 위해 기본적으로 frameOptions() 기능이 활성화되어 있으며 디폴트 값은 DENY로, HTML 태그를 이용한 페이지 렌더링을 허용하지 않겠다는 의미
        - (1)과 같이 .frameOptions().sameOrigin()을 호출하면 동일 출처로부터 들어오는 request만 페이지 렌더링을 허용함
    - (2)의 userDetailsService() 메서드는 InMemory User를 등록하는 역할을 하지만 이제 데이터베이스에서 User를 등록하고, 데이터베이스에 저장된 User의 인증 정보를 사용할 것이므로 userDetailsService() 메서드를 제거함

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.provisioning.UserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers().frameOptions().sameOrigin() // (1)
            .and()
            .csrf().disable()
            .formLogin()
            .loginPage("/auths/login-form")
            .loginProcessingUrl("/process_login")
            .failureUrl("/auths/login-form?error")
            .and()
            .logout()
            .logoutUrl("/logout")
            .logoutSuccessUrl("/")
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

   // ======================================================== 여기부터
   /**
    * InMemory User를 위한 설정이므로, 제거 대상.
    */
    @Bean
    public UserDetailsManager userDetailsService() {    // (2)
        UserDetails user =
                User.withDefaultPasswordEncoder()
                        .username("mason@gmail.com")
                        .password("1111")
                        .roles("USER")
                        .build();

        UserDetails admin =
                User.withDefaultPasswordEncoder()
                        .username("admin@gmail.com")
                        .password("2222")
                        .roles("ADMIN")
                        .build();

        return new InMemoryUserDetailsManager(user, admin);
    }
   // ======================================================== 여기까지 제거

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

<br>

2. JavaConfiguration의 Bean 등록 변경
    - DBMemberService는 내부에서 데이터를 데이터베이스에 저장하고, 패스워드를 암호화해야 하므로 (1-1)과 같이 MemberRepository와 PasswordEncoder 객체를 DI 해줌

```java
@Configuration
public class JavaConfiguration {
    // (1)
    @Bean
    public MemberService dbMemberService(MemberRepository memberRepository,
                                         PasswordEncoder passwordEncoder) {
        return new DBMemberService(memberRepository, passwordEncoder); (1-1)
    }
}
```

<br>

3. DBMemberService 구현
    - DBMemberService는 User의 인증 정보를 데이터베이스에 저장하는 역할을 하는데, Spring Security 입장에서는 User라고 부르는 정보는 회원 가입 시 등록하는 회원 정보 안에 포함되어 있다고 할 수 있음
    - (1)의 생성자를 통해 MemberRepository와 PasswordEncoder Bean 객체를 DI 받음
    - (2)에서 PasswordEncoder를 이용해 패스워드를 암호화함
    - (3)에서 암호화된 패스워드를 password 필드에 다시 할당함

```java
import com.codestates.exception.BusinessLogicException;
import com.codestates.exception.ExceptionCode;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

@Transactional
public class DBMemberService implements MemberService {
    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;

    // (1)
    public DBMemberService(MemberRepository memberRepository,
                             PasswordEncoder passwordEncoder) {
        this.memberRepository = memberRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());
        String encryptedPassword = passwordEncoder.encode(member.getPassword());  // (2)
        member.setPassword(encryptedPassword);    // (3)

        Member savedMember = memberRepository.save(member);

        System.out.println("# Create Member in DB");
        return savedMember;
    }
    
    ...
    ...
}
```

<br>

4. Custom UserDetailsService 구현
    - HelloUserDetailsService와 같은 Custom UserDetailsService를 구현하기 위해서는 (1)과 같이 UserDetailsService 인터페이스를 구현해야 함
    - HelloUserDetailsService는 데이터베이스에서 User를 조회하고, 조회한 User의 권한(Role) 정보를 생성하기 위해 (2)와 같이 MemberRepository와 HelloAuthorityUtils 클래스를 DI 받음
    - UserDetailsService 인터페이스를 implements 하는 구현 클래스는 (3)과 같이 loadUserByUsername(String username)이라는 추상 메서드를 구현해야 함
        - UserDetails는 UserDetailsService에 의해 로드(load)되어 인증을 위해 사용되는 핵심 User 정보를 표현하는 인터페이스
        - UserDetails 인터페이스의 구현체는 Spring Security에서 보안 정보 제공을 목적으로 직접 사용되지는 않고, Authentication 객체로 캡슐화되어 제공됨
    - (4)에서는 HelloAuthorityUtils를 이용해 데이터베이스에서 조회한 회원의 이메일 정보를 이용해 Role 기반의 권한 정보(GrantedAuthority) 컬렉션을 생성함
    - 데이터베이스에서 조회한 인증 정보와 (4)에서 생성한 권한 정보를 Spring Security에서는 아직 알지 못하기 때문에 Spring Security에 이 정보들을 제공해 주어야 하며, (5)에서는 UserDetails 인터페이스의 구현체인 User 클래스의 객체를 통해 제공함
        - (5)와 같이 데이터베이스에서 조회한 User 클래스의 객체를 리턴하면 Spring Security가 이 정보를 이용해 인증 절차를 수행함

```java
import com.codestates.auth.utils.HelloAuthorityUtils;
import com.codestates.exception.BusinessLogicException;
import com.codestates.exception.ExceptionCode;
import com.codestates.member.Member;
import com.codestates.member.MemberRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Optional;

@Component
public class HelloUserDetailsServiceV1 implements UserDetailsService {   // (1)
    private final MemberRepository memberRepository;
    private final HelloAuthorityUtils authorityUtils;

    // (2)
    public HelloUserDetailsServiceV1(MemberRepository memberRepository, HelloAuthorityUtils authorityUtils) {
        this.memberRepository = memberRepository;
        this.authorityUtils = authorityUtils;
    }

    // (3)
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Member> optionalMember = memberRepository.findByEmail(username);
        Member findMember = optionalMember.orElseThrow(() -> new BusinessLogicException(ExceptionCode.MEMBER_NOT_FOUND));

        // (4)
        Collection<? extends GrantedAuthority> authorities = authorityUtils.createAuthorities(findMember.getEmail());

        // (5)   
        return new User(findMember.getEmail(), findMember.getPassword(), authorities);
    }
}
```

<br>

- HelloAuthorityUtils
    - HelloUserDetailsService에서 Role 기반의 User 권한을 생성하기 위해 사용한 HelloAuthorityUtils 코드
    - (1)은 application.yml에 추가한 프로퍼티를 가져오는 표현식
        - (1)과 같이 @Value("${프로퍼티 경로}")의 표현식 형태로 작성하면 application.yml에 정의되어 있는 프로퍼티의 값을 클래스 내에서 사용할 수 있음
        - (1)에서는 application.yml에 미리 정의한 관리자 권한을 가질 수 있는 이메일 주소를 불러옴
        - application.yml 파일에 정의한 관리자용 이메일 주소는 회원 등록 시, 특정 이메일 주소에 관리자 권한을 부여할 수 있는지를 결정하기 위해 사용됨
    - (2)에서는 Spring Security에서 지원하는 AuthorityUtils 클래스를 이용해서 관리자용 권한 목록을 List&#60;GrantedAuthority&#62; 객체로 미리 생성함
    - (3)에서는 Spring Security에서 지원하는 AuthorityUtils 클래스를 이용해서 일반 사용 권한 목록을 List&#60;GrantedAuthority&#62; 객체로 미리 생성함
      (4)에서는 파라미터로 전달받은 이메일 주소가 application.yml 파일에서 가져온 관리자용 이메일 주소와 동일하다면 관리자용 권한인 List&#60;GrantedAuthority&#62; ADMIN_ROLES를 리턴함

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
public class HelloAuthorityUtils {
    // (1)
    @Value("${mail.address.admin}")
    private String adminMailAddress;

    // (2)
    private final List<GrantedAuthority> ADMIN_ROLES = AuthorityUtils.createAuthorityList("ROLE_ADMIN", "ROLE_USER");

    // (3)
    private final List<GrantedAuthority> USER_ROLES = AuthorityUtils.createAuthorityList("ROLE_USER");
    
    public List<GrantedAuthority> createAuthorities(String email) {
        // (4)
        if (email.equals(adminMailAddress)) {
            return ADMIN_ROLES;
        }
        return USER_ROLES;
    }
}
```

<br>

```java
// application.yml 파일의 관리자 이메일 주소 정의
...
...

mail:
  address:
    admin: admin@gmail.com
```

<br>

5. H2 웹 콘솔에서 등록한 회원 정보 확인 및 로그인 인증 테스트
    - 패스워드 정보가 암호화 되어있는 것을 확인할 수 있음
6. 개선된 Custom UserDetails 구현
    - 기존에는 loadUserByUsername() 메서드의 리턴 값으로 new User(findMember.getEmail(), findMember.getPassword(), authorities);을 리턴했지만 개선된 코드에서는 (1)과 같이 new HelloUserDetails(findMember);라는 Custom UserDetails 클래스의 생성자로 findMember를 전달하면서 코드가 조금 더 깔끔해짐
        - loadUserByUsername() 메서드 내부에서 User의 권한 정보를 생성하는 Collection&#60;? extends GrantedAuthority&#62; authorities = authorityUtils.createAuthorities(findMember); 코드가 사라지고, (2)에서 정의한 HelloUserDetails 클래스 내부로 포함됨
    - (2)의 HelloUserDetails 클래스는 UserDetails 인터페이스를 구현하고 있고 또한 Member 엔티티 클래스를 상속하고 있음
        - 이렇게 구성하면 데이터베이스에서 조회한 회원 정보를 Spring Security의 User 정보로 변환하는 과정과 User의 권한 정보를 생성하는 과정을 캡슐화할 수 있음
        - HelloUserDetails 클래스는 Member 엔티티 클래스를 상속하고 있으므로 HelloUserDetails를 리턴 받아 사용하는 측에서는 두 개 클래스의 객체를 모두 다 손쉽게 캐스팅해서 사용 가능하다는 장점이 있음
    - (2-3)에서는 HelloAuthorityUtils의 createAuthorities() 메서드를 이용해 User의 권한 정보를 생성하고 있음
        - 이 코드는 기존에는 loadUserByUsername() 메서드 내부에 있었지만 지금은 HelloUserDetails 클래스 내부에서 사용되도록 캡슐화됨
    - (2-4)에서는 Spring Security에서 인식할 수 있는 username을 Member 클래스의 email 주소로 채우고 있으며, getUsername()의 리턴 값은 null일 수 없음
    - 기타 UserDetails 인터페이스의 추상 메서드를 구현한 부분은 지금은 크게 중요하지 않은 부분이므로 모두 true값을 리턴하고 있음

```java
import com.codestates.auth.utils.HelloAuthorityUtils;
import com.codestates.exception.BusinessLogicException;
import com.codestates.exception.ExceptionCode;
import com.codestates.member.Member;
import com.codestates.member.MemberRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Optional;

@Component
public class HelloUserDetailsServiceV2 implements UserDetailsService {
    private final MemberRepository memberRepository;
    private final HelloAuthorityUtils authorityUtils;

    public HelloUserDetailsServiceV2(MemberRepository memberRepository, HelloAuthorityUtils authorityUtils) {
        this.memberRepository = memberRepository;
        this.authorityUtils = authorityUtils;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Member> optionalMember = memberRepository.findByEmail(username);
        Member findMember = optionalMember.orElseThrow(() -> new BusinessLogicException(ExceptionCode.MEMBER_NOT_FOUND));

        return new HelloUserDetails(findMember);  // (1) 개선된 부분
    }

    // (2) HelloUserDetails 클래스 추가
    private final class HelloUserDetails extends Member implements UserDetails { // (2-1)
        // (2-2)
        HelloUserDetails(Member member) {
            setMemberId(member.getMemberId());
            setFullName(member.getFullName());
            setEmail(member.getEmail());
            setPassword(member.getPassword());
        }

        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            return authorityUtils.createAuthorities(this.getEmail());  // (2-3) 리팩토링 포인트
        }

        // (2-4)
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

7. User의 Role을 DB에서 관리하기
    - User의 권한 정보를 데이터베이스에서 관리하기 위한 과정
        1. User의 권한 정보를 저장하기 위한 테이블 생성
        2. 회원 가입 시, User의 권한 정보(Role)를 데이터베이스에 저장하는 작업
        3. 로그인 인증 시, User의 권한 정보를 데이터베이스에서 조회하는 작업
<br>

- 1. User의 권한 정보를 저장하기 위한 테이블 생성
    - (1)과 같이 List, Set 같은 컬렉션 타입의 필드는 @ElementCollection 애너테이션을 추가하면 User 권한 정보와 관련된 별도의 엔티티 클래스를 생성하지 않아도 간단하게 매핑 처리가 됨
    - 한 명의 회원이 한 개 이상의 Role을 가질 수 있으므로, MEMBER 테이블과 MEMBER_ROLES 테이블은 1대N의 관계

```java
// Member 엔티티 클래스와 User의 권한 정보를 매핑한 코드
import com.codestates.audit.Auditable;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import javax.persistence.*;
import java.security.Principal;
import java.util.ArrayList;
import java.util.List;

@NoArgsConstructor
@Getter
@Setter
@Entity
public class Member extends Auditable implements Principal{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberId;

    @Column(length = 100, nullable = false)
    private String fullName;

    @Column(nullable = false, updatable = false, unique = true)
    private String email;

    @Column(length = 100, nullable = false)
    private String password;

    @Enumerated(value = EnumType.STRING)
    @Column(length = 20, nullable = false)
    private MemberStatus memberStatus = MemberStatus.MEMBER_ACTIVE;

    // (1) User의 권한 정보 테이블과 매핑되는 정보
    @ElementCollection(fetch = FetchType.EAGER)
    private List<String> roles = new ArrayList<>();

    public Member(String email) {
        this.email = email;
    }

    public Member(String email, String fullName, String password) {
        this.email = email;
        this.fullName= fullName;
        this.password = password;
    }

    @Override
    public String getName() {
        return getEmail();
    }

    public enum MemberStatus {
        MEMBER_ACTIVE("활동중"),
        MEMBER_SLEEP("휴면 상태"),
        MEMBER_QUIT("탈퇴 상태");

        @Getter
        private String status;

        MemberStatus(String status) {
           this.status = status;
        }
    }

    public enum MemberRole {
        ROLE_USER,
        ROLE_ADMIN
    }
}
```

<br>

- 2. 회원 가입 시, User의 권한 정보(Role)를 데이터베이스에 저장
    - (1)에서는 authorityUtils.createRoles(member.getEmail());를 통해 회원의 권한 정보(List&#60;String&#62; roles)를 생성한 뒤 member 객체에 넘겨주고 있음
    - (2)에서는 파라미터로 전달된 이메일 주소가 application.yml 파일의 mail.address.admin 프로퍼티에 정의된 이메일 주소와 동일하면 관리자 Role 목록(ADMIN_ROLES_STRING)을 리턴하고, 그 외에는 일반 사용자 Role 목록(USER_ROLES_STRING)을 리턴함

```java
// DBMemberService
import com.codestates.auth.utils.HelloAuthorityUtils;
import com.codestates.exception.BusinessLogicException;
import com.codestates.exception.ExceptionCode;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Transactional
public class DBMemberService implements MemberService {
    ...
    ...
  
    private final HelloAuthorityUtils authorityUtils;

    ...
    ...

    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());
        String encryptedPassword = passwordEncoder.encode(member.getPassword());
        member.setPassword(encryptedPassword);

        // (1) Role을 DB에 저장
        List<String> roles = authorityUtils.createRoles(member.getEmail());
        member.setRoles(roles);

        Member savedMember = memberRepository.save(member);

        return savedMember;
    }

    ...
    ...
}
```

<br>

```java
// createRoles() 메서드가 추가된 HelloAuthorityUtils 클래스의 코드
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.stream.Collectors;

@Component
public class HelloAuthorityUtils {
    @Value("${mail.address.admin}")
    private String adminMailAddress;

    ...
    ...

    private final List<String> ADMIN_ROLES_STRING = List.of("ADMIN", "USER");
    private final List<String> USER_ROLES_STRING = List.of("USER");

    ...
    ...

    // (2) DB 저장용
    public List<String> createRoles(String email) {
        if (email.equals(adminMailAddress)) {
            return ADMIN_ROLES_STRING;
        }
        return USER_ROLES_STRING;
    }
}
```

<br>

- 3. 로그인 인증 시, User의 권한 정보를 데이터베이스에서 조회하는 작업
    - (1)에서는 HelloUserDetails가 상속하고 있는 Member(extends Member)에 데이터베이스에서 조회한 List&#60;String&#62; roles를 전달함
    - (2)에서 다시 Member(extends Member)에 전달한 Role 정보를 authorityUtils.createAuthorities() 메서드의 파라미터로 전달해서 권한 목록(List&#60;GrantedAuthority&#62;)을 생성함
    - (3)을 보면 기존에는 application.yml 파일의 mail.address.admin 프로퍼티에 정의된 관리자용 이메일 주소를 기준으로 관리자 Role을 추가했지만, 지금은 그럴 필요가 없이 데이터베이스에서 가지고 온 Role 목록(List&#60;String&#62; roles)을 그대로 이용해서 권한 목록(authorities)을 생성함
        - 주의해야 할 것은 (4)와 같이 SimpleGrantedAuthority 객체를 생성할 때 생성자 파라미터로 넘겨주는 값이 “ USER" 또는 “ADMIN"으로 넘겨주면 안 되고 “ROLE_USER" 또는 “ROLE_ADMIN" 형태로 넘겨주어야 함

```java
// 개선된 HelloUserDetailsService
import com.codestates.auth.utils.HelloAuthorityUtils;
import com.codestates.exception.BusinessLogicException;
import com.codestates.exception.ExceptionCode;
import com.codestates.member.Member;
import com.codestates.member.MemberRepository;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Optional;

@Component
public class HelloUserDetailsServiceV3 implements UserDetailsService {
    private final MemberRepository memberRepository;
    private final HelloAuthorityUtils authorityUtils;

    public HelloUserDetailsServiceV3(MemberRepository memberRepository, HelloAuthorityUtils authorityUtils) {
        this.memberRepository = memberRepository;
        this.authorityUtils = authorityUtils;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Member> optionalMember = memberRepository.findByEmail(username);
        Member findMember = optionalMember.orElseThrow(() -> new BusinessLogicException(ExceptionCode.MEMBER_NOT_FOUND));

        return new HelloUserDetails(findMember);
    }

    private final class HelloUserDetails extends Member implements UserDetails {
        HelloUserDetails(Member member) {
            setMemberId(member.getMemberId());
            setFullName(member.getFullName());
            setEmail(member.getEmail());
            setPassword(member.getPassword());
            setRoles(member.getRoles());        // (1)
        }

        @Override
        public Collection<? extends GrantedAuthority> getAuthorities() {
            // (2) DB에 저장된 Role 정보로 User 권한 목록 생성
            return authorityUtils.createAuthorities(this.getRoles());
        }

        ...
        ...
    }

}
```

<br>

```java
// 데이터베이스에서 조회한 Role 정보를 기반으로 User의 권한 목록 생성하는 createAuthorities(List<String> roles) 메서드가 추가된 HelloAuthorityUtils 클래스의 코드
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.stream.Collectors;

@Component
public class HelloAuthorityUtils {
    @Value("${mail.address.admin}")
    private String adminMailAddress;

    private final List<GrantedAuthority> ADMIN_ROLES = AuthorityUtils.createAuthorityList("ROLE_ADMIN", "ROLE_USER");
    private final List<GrantedAuthority> USER_ROLES = AuthorityUtils.createAuthorityList("ROLE_USER");
    private final List<String> ADMIN_ROLES_STRING = List.of("ADMIN", "USER");
    private final List<String> USER_ROLES_STRING = List.of("USER");

    // 메모리 상의 Role을 기반으로 권한 정보 생성.
    public List<GrantedAuthority> createAuthorities(String email) {
        if (email.equals(adminMailAddress)) {
            return ADMIN_ROLES;
        }
        return USER_ROLES;
    }

    // (3) DB에 저장된 Role을 기반으로 권한 정보 생성
    public List<GrantedAuthority> createAuthorities(List<String> roles) {
       List<GrantedAuthority> authorities = roles.stream()
               .map(role -> new SimpleGrantedAuthority("ROLE_" + role)) // (4)
               .collect(Collectors.toList());
       return authorities;
    }

    ...
    ...
}
```

<br>

---

<br>

## Custom AuthenticationProvider를 사용하는 방법
- 직접 로그인 인증을 처리하는 방법
- Spring Security의 핵심 컴포넌트인 AuthenticationProvider를 이해하는 데 도움이 되고 또한 보안 요구 사항에 부합하는 적절한 인증 방식을 직접 구현해야 할 경우, Custom AuthenticationProvider가 필요할 수 있음
<br>

- Custom AuthenticationProvider인 HelloUserAuthenticationProvider
    - (1)과 같이 AuthenticationProvider 인터페이스의 구현 클래스로 정의함
        - 따라서 AuthenticationProvider의 구현 클래스로써의 HelloUserAuthenticationProvider를 구현해야 함
        - Spring Security는 AuthenticationProvider를 구현한 구현 클래스가 Spring Bean으로 등록되어 있다면 해당 AuthenticationProvider를 이용해서 인증을 진행함
        - 클라이언트 쪽에서 로그인 인증을 시도하면 구현한 HelloUserAuthenticationProvider가 직접 인증을 처리하게 됨
    - AuthenticationProvider 인터페이스의 구현 클래스는 authenticate(Authentication authentication) 메서드와 supports(Class&#60;?&#62; authentication) 메서드를 구현해야 함
        - (2)의 supports(Class&#60;?&#62; authentication) 메서드는 구현하는 Custom AuthenticationProvider(HelloUserAuthenticationProvider)가 Username/Password 방식의 인증을 지원한다는 것을 Spring Security에 알려주는 역할을 함
        - supports() 메서드의 리턴값이 true일 경우, Spring Security는 해당 AuthenticationProvider의 authenticate() 메서드를 호출해서 인증을 진행함
    - (3)의 authenticate(Authentication authentication)에서 직접 작성한 인증 처리 로직을 이용해 사용자의 인증 여부를 결정함
        - (3-1)에서 authentication을 캐스팅하여 UsernamePasswordAuthenticationToken을 얻음
        - UsernamePasswordAuthenticationToken 객체에서 (3-2)와 같이 해당 사용자의 Username을 얻은 후, 존재하는지 체크함
        - Username이 존재한다면 (3-3)과 같이 userDetailsService를 이용해 데이터베이스에서 해당 사용자를 조회함
        - (3-4)에서 로그인 정보에 포함된 패스워드(authToken.getCredentials())와 데이터베이스에 저장된 사용자의 패스워드 정보가 일치하는지를 검증함
        - (3-4)의 검증 과정을 통과했다면 로그인 인증에 성공한 사용자이므로 (3-5)와 같이 해당 사용자의 권한을 생성함
        - (3-6)과 같이 인증된 사용자의 인증 정보를 리턴값으로 전달함

```java
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Optional;

@Component
public class HelloUserAuthenticationProvider implements AuthenticationProvider {   // (1)
    private final HelloUserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    public HelloUserAuthenticationProvider(HelloUserDetailsService userDetailsService, 
                                           PasswordEncoder passwordEncoder) {
        this.userDetailsService = userDetailsService;
        this.passwordEncoder = passwordEncoder;
    }


    // (3)
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        UsernamePasswordAuthenticationToken authToken = (UsernamePasswordAuthenticationToken) authentication;  // (3-1)

        // (3-2)
        String username = authToken.getName();
        Optional.ofNullable(username).orElseThrow(() -> new UsernameNotFoundException("Invalid User name or User Password"));

        // (3-3)
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);

        String password = userDetails.getPassword();
        verifyCredentials(authToken.getCredentials(), password);    // (3-4)

        Collection<? extends GrantedAuthority> authorities = userDetails.getAuthorities();  // (3-5)

        // (3-6)
        return UsernamePasswordAuthenticationToken.authenticated(username, password, authorities);
    }

    // (2) HelloUserAuthenticationProvider가 Username/Password 방식의 인증을 지원한다는 것을 Spring Security에 알려준다.
    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.equals(authentication);
    }

    private void verifyCredentials(Object credentials, String password) {
        if (!passwordEncoder.matches((String)credentials, password)) {
            throw new BadCredentialsException("Invalid User name or User Password");
        }
    }
}
```

<br>

- 개선된 HelloUserAuthenticationProvider
    - Custom AuthenticationProvider를 이용할 경우에는 회원가입 전 인증 실패 시, “Whitelebel Error Page”가 표시됨
        - MemberService에서 등록된 회원 정보가 없으면, BusinessLogicException을 throw 하는데 이 BusinessLogicException이 Cusotm AuthenticationProvider를 거쳐 그대로 Spring Security 내부 영역으로 throw 되기 때문
        - Spring Security에서는 인증 실패 시, AuthenticationException이 throw 되지 않으면 Exception에 대한 별도의 처리를 하지 않고, 서블릿 컨테이너인 톰캣 쪽으로 이 처리를 넘김
        - 결국 서블릿 컨테이너 영역에서 해당 Exception에 대해 “/error” URL로 포워딩하는데 특별히 “/error” URL로 포워딩되었을 때 보여줄 뷰 페이지를 별도로 구성하지 않았기 때문에 디폴트 페이지인 “Whitelebel Error Page”를 브라우저에 표시하는 것
        - 해결 방법으로는 Cusotm AuthenticationProvider에서 Exception이 발생할 경우, 이 Exception을 catch 해서 AuthenticationException으로 rethrow를 해줌
    - (1)에서 UsernameNotFoundException을 throw 하도록 수정되었는데, UsernameNotFoundException은 AuthenticationException을 상속하는 하위 Exception이기 때문에 이 UsernameNotFoundException이 throw되면 Spring Security 쪽에서 정상적으로 catch해서 정상적인 인증 실패 화면으로 리다이렉트 시켜줌
        - Custom AuthenticationProvider에서 AuthenticationException이 아닌 Exception이 발생할 경우에는 꼭 AuthenticationException을 rethrow 하도록 코드를 구성해야 함
    - AuthenticationProvider란?
        - AuthenticationProvider는 Spring Security에서 클라이언트로부터 전달받은 인증 정보를 바탕으로 인증된 사용자인지에 대한 인증 처리를 수행하는 Spring Security 컴포넌트
        - AuthenticationProvider는 인터페이스 형태로 정의되어 있으며, Spring Security에서는 AnonymousAuthenticationProvider, DaoAuthenticationProvider, JwtAuthenticationProvider, RememberMeAuthenticationProvider, OAuth2LoginAuthenticationProvider 등 다양한 유형의 AuthenticationProvider 구현체를 제공함

```java
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.Optional;

@Component
public class HelloUserAuthenticationProvider implements AuthenticationProvider {
    private final HelloUserDetailsService userDetailsService;
    private final PasswordEncoder passwordEncoder;

    public HelloUserAuthenticationProvider(HelloUserDetailsService userDetailsService,
                                           PasswordEncoder passwordEncoder) {
        this.userDetailsService = userDetailsService;
        this.passwordEncoder = passwordEncoder;
    }

    // V2: AuthenticationException을 rethrow 하는 개선 코드
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        UsernamePasswordAuthenticationToken authToken = (UsernamePasswordAuthenticationToken) authentication;

        String username = authToken.getName();
        Optional.ofNullable(username).orElseThrow(() -> new UsernameNotFoundException("Invalid User name or User Password"));
        try {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            String password = userDetails.getPassword();
            verifyCredentials(authToken.getCredentials(), password);

            Collection<? extends GrantedAuthority> authorities = userDetails.getAuthorities();
            return UsernamePasswordAuthenticationToken.authenticated(username, password, authorities);
        } catch (Exception ex) {
            throw new UsernameNotFoundException(ex.getMessage()); // (1) AuthenticationException으로 다시 throw 한다.
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.equals(authentication);
    }

    private void verifyCredentials(Object credentials, String password) {
        if (!passwordEncoder.matches((String)credentials, password)) {
            throw new BadCredentialsException("Invalid User name or User Password");
        }
    }
}
```