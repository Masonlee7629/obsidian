태그: #Spring 
연결문서: [Spring Security 기본 2](Spring%20Security%20기본%202.md)

# Spring Security
## Spring Security란?
### Spring Security의 개념
Spring Security는 Spring MVC 기반 애플리케이션의 인증(Authentication)과 인가(Authorization or 권한 부여) 기능을 지원하는 보안 프레임워크로써, Spring MVC 기반 애플리케이션에 보안을 적용하기 위한 사실상의 표준이다.  
<br>  

물론 Spring에서 지원하는 Interceptor나 Servlet Filter를 이용해서 보안 기능을 직접 구현할 수 있지만 웹 애플리케이션 보안의 대부분의 기능을 Spring Security에서 안정적으로 지원하고 있으므로 구조적으로 잘 만들어진 검증된 Spring Security를 이용하는 것이 안전한 선택이라고 볼 수 있다.

<br>

### Spring Security로 할 수 있는 보안 강화 기능
-   다양한 유형(폼 로그인 인증, 토큰 기반 인증, OAuth2 기반 인증, LDAP 인증)의 사용자 인증 기능 적용
-   애플리케이션 사용자의 역할(Role)에 따른 권한 레벨 적용
-   애플리케이션에서 제공하는 리소스에 대한 접근 제어
-   민감한 정보에 대한 데이터 암호화
-   SSL 적용
-   일반적으로 알려진 웹 보안 공격 차단

이 외에도 SSO, 클라이언트 인증서 기반 인증, 메서드 보안, 접근 제어 목록(Access Control List) 같은 보안을 위한 기능들을 지원한다.

<br>

### Spring Security 기본 용어
-   Principal(주체)
    -   Spring Security에서 사용되는 Principal은 애플리케이션에서 작업을 수행할 수 있는 사용자, 디바이스 또는 시스템이 될 수 있으며, 일반적으로 인증 프로세스가 성공적으로 수행된 사용자의 계정 정보를 의미한다.
-   Authentication(인증)
    -   Authentication은 애플리케이션을 사용하는 사용자가 본인이 맞음을 증명하는 절차를 의미한다.
    -   Authentication을 정상적으로 수행하기 위해서는 사용자를 식별하기 위한 정보가 필요한데 이를 Credential(신원 증명 정보)이라고 한다.
    -   특정 사이트에서 로그인을 위해 입력하는 패스워드 역시 로그인 아이디를 증명하기 위한 Credential이 된다.
-   Authorization(인가 또는 권한 부여)
    -   Authorization은 Authentication이 정상적으로 수행된 사용자에게 하나 이상의 권한(authority)을 부여하여 특정 애플리케이션의 특정 리소스에 접근할 수 있게 허가하는 과정을 의미한다.
    -   Authorization은 반드시 Authentication 과정 이후 수행되어야 하며 권한은 일반적으로 역할(Role) 형태로 부여된다.
-   Access Control(접근 제어)
    -   Access Control은 사용자가 애플리케이션의 리소스에 접근하는 행위를 제어하는 것을 의미한다.

<br>

---

<br>

## Spring Security를 사용해야 하는 이유
결론적으로는 애플리케이션의 보안을 강화하기 위한 솔루션으로 Spring Security 만 한 다른 프레임워크가 존재하지 않기 때문이다.  
<br>

물론 Apache Shiro, OACC 같은 Java 애플리케이션을 위한 보안 프레임워크가 존재하지만, Spring Security는 다른 보안 프레임워크가 제공하는 기능들을 모두 아우르는 기능을 지원하고 있으며, 또한 Spring 기반의 애플리케이션을 구현하는 개발자로서는 Spring과 궁합이 가장 잘 맞는 Spring Security를 사용하는 것은 자연스러운 일이라고 볼 수 있다.  
<br>

또한, Spring Security를 사용하면 Spring Security에서 지원하는 기본 옵션을 통해 대부분의 보안 요구 사항을 만족시킬 수 있다.  
<br>

만약 기본 옵션으로 만족시킬 수 없는 특정 보안 요구 사항을 만족시켜야 할 경우, 특정 보안 요구 사항을 만족시키기 위한 코드의 커스터마이징이 용이하고 유연한 확장이 가능하다.