태그: #Spring 
연결문서: [Spring Security - JWT 인증 1](Spring%20Security%20-%20JWT%20인증%201.md)

# Spring Security

## Spring Security의 권한 처리 흐름

### Spring Security의 컴포넌트로 보는 권한 부여(Authorization) 처리 흐름

![](image_18.png)

<br>

- Spring Security Filter Chain에서 URL을 통해 사용자의 액세스를 제한하는 권한 부여 Filter는 AuthorizationFilter
- AuthorizationFilter는 먼저 (1)과 같이 SecurityContextHolder로부터 Authentication을 획득함
- (2)와 같이 SecurityContextHolder로부터 획득한 Authentication과 HttpServletRequest를 AuthorizationManager에게 전달함
- AuthorizationManager는 권한 부여 처리를 총괄하는 매니저 역할을 하는 인터페이스이고, RequestMatcherDelegatingAuthorizationManager는 AuthorizationManager를 구현하는 구현체 중 하나
    - RequestMatcherDelegatingAuthorizationManager는 RequestMatcher 평가식을 기반으로 해당 평가식에 매치되는 AuthorizationManager에게 권한 부여 처리를 위임하는 역할을 함
    - RequestMatcherDelegatingAuthorizationManager가 직접 권한 부여 처리를 하는 것이 아니라 RequestMatcher를 통해 매치되는 AuthorizationManager 구현 클래스에게 위임만 함
- RequestMatcherDelegatingAuthorizationManager 내부에서 매치되는 AuthorizationManager 구현 클래스가 있다면 해당 AuthorizationManager 구현 클래스가 사용자의 권한을 체크함(3)
- 적절한 권한이라면 (4)와 같이 다음 요청 프로세스를 계속 이어감
- 만약 적절한 권한이 아니라면 (5)와 같이 AccessDeniedException이 throw되고 ExceptionTranslationFilter가 AccessDeniedException을 처리함

<br>

### Spring Security의 권한 부여 컴포넌트

#### AuthorizationFilter
- URL을 통해 사용자의 액세스를 제한하는 권한 부여 Filter이며, Spring Security 5.5 버전부터 FilterSecurityInterceptor를 대체함

<br>

```java
public class AuthorizationFilter extends OncePerRequestFilter {

	private final AuthorizationManager<HttpServletRequest> authorizationManager;
  
  ...

  // (1)
	public AuthorizationFilter(AuthorizationManager<HttpServletRequest> authorizationManager) {
		Assert.notNull(authorizationManager, "authorizationManager cannot be null");
		this.authorizationManager = authorizationManager;
	}

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		AuthorizationDecision decision = this.authorizationManager.check(this::getAuthentication, request); // (2)
		this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, decision);
		if (decision != null && !decision.isGranted()) {
			throw new AccessDeniedException("Access Denied");
		}
		filterChain.doFilter(request, response);
	}

  ...
}
```

<br>

- (1)과 같이 AuthorizationFilter 객체가 생성될 때, AuthorizationManager를 DI 받음
    - DI 받은 AuthorizationManager를 통해 권한 부여 처리를 진행함
- (2)와 같이 DI 받은 AuthorizationManager의 check() 메서드를 호출해 적절한 권한 부여 여부를 체크함
    - AuthorizationManager의 check() 메서드는 AuthorizationManager 구현 클래스에 따라 권한 체크 로직이 다름
    - URL 기반으로 권한 부여 처리를 하는 AuthorizationFilter는 AuthorizationManager의 구현 클래스로 RequestMatcherDelegatingAuthorizationManager를 사용함

<br>

#### AuthorizationManager
- 권한 부여 처리를 총괄하는 매니저 역할을 하는 인터페이스
- AuthorizationManager 인터페이스는 check() 메서드 하나만 정의되어 있으며, Supplier와 제너릭 타입의 객체를 파라미터로 가짐

<br>

```java
@FunctionalInterface
public interface AuthorizationManager<T> {
  ...

	@Nullable
	AuthorizationDecision check(Supplier<Authentication> authentication, T object);

}
```

<br>

#### RequestMatcherDelegatingAuthorizationManager
- AuthorizationManager의 구현 클래스 중 하나이며, 직접 권한 부여 처리를 수행하지 않고 RequestMatcher를 통해 매치되는 AuthorizationManager 구현 클래스에게 권한 부여 처리를 위임함

<br>

```java
public final class RequestMatcherDelegatingAuthorizationManager implements AuthorizationManager<HttpServletRequest> {

  ...

	@Override
	public AuthorizationDecision check(Supplier<Authentication> authentication, HttpServletRequest request) {
		if (this.logger.isTraceEnabled()) {
			this.logger.trace(LogMessage.format("Authorizing %s", request));
		}

    // (1)
		for (RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>> mapping : this.mappings) {

			RequestMatcher matcher = mapping.getRequestMatcher(); // (2)
			MatchResult matchResult = matcher.matcher(request);
			if (matchResult.isMatch()) {   // (3)
				AuthorizationManager<RequestAuthorizationContext> manager = mapping.getEntry();
				if (this.logger.isTraceEnabled()) {
					this.logger.trace(LogMessage.format("Checking authorization on %s using %s", request, manager));
				}
				return manager.check(authentication,
						new RequestAuthorizationContext(request, matchResult.getVariables()));
			}
		}
		this.logger.trace("Abstaining since did not find matching RequestMatcher");
		return null;
	}
}
```

<br>

- check() 메서드의 내부에서 (1)과 같이 루프를 돌면서 RequestMatcherEntry 정보를 얻은 후에 (2)와 같이 RequestMatcher 객체를 얻음
- (3)과 같이 MatchResult.isMatch()가 true이면 AuthorizationManager 객체를 얻은 뒤, 사용자의 권한을 체크함
    - 여기서의 RequestMatcher는 SecurityConfiguration에서 .antMatchers("/orders/&#42;&#42;").hasRole("ADMIN") 와 같은 메서드 체인 정보를 기반으로 생성됨

<br>

---

<br>

## 접근 제어 표현식
- Spring Security에서는 웹 및 메서드 보안을 위해 표현식(Spring EL)을 사용할 수 있음

<br>

- hasRole(Stirng role)
    - 현재 보안 주체(principal)가 지정된 역할을 갖고 있는지 여부를 확인하고 가지고 있다면 true를 리턴함
    - hasRole(’admin’)처럼 파라미터로 넘긴 role이 ROLE_ 로 시작하지 않으면 기본적으로 추가함(DefaultWebSecurityExpressionHandler의 defaultRolePrefix를 수정하면 커스텀할 수 있음)
- hasAnyRole(String… roles)
    - 현재 보안 주체가 지정한 역할 중 1개라도 가지고 있으면 true를 리턴함(문자열 리스트를 콤마로 구분해서 전달)
    - Ex) hasAnyRole(’admin’, ‘user’)
- hasAuthority(String authority)
    - 현재 보안 주체가 지정한 권한을 갖고 있는지 여부를 확인하고 가지고 있다면 true를 리턴함
    - Ex) hasAuthority(’read’)
- hasAnyAuthority(String… authorities)
    - 현재 보안 주체가 지정한 권한 중 하나라도 있으면 true를 리턴함
    - Ex) hasAnyAuthority(’read’, ‘write’)
- principal
    - 현재 사용자를 나타내는 principal 객체에 직접 접근할 수 있음
- authentication
    - SecurityContext로 조회할 수 있는 현재 Authentication 객체에 직접 접근할 수 있음
- permitAll
    - 항상 true로 평가함
- denyAll
    - 항상 false로 평가함
- isAnonymous()
    - 현재 보안 주체가 익명 사용자면 true를 리턴함
- isRememberMe()
    - 현재 보안 주체가 remember-me 사용자면 true를 리턴함
- isAuthenticated()
    - 사용자가 익명이 아닌 경우 true를 리턴함
- isFullyAuthenticated()
    - 사용자가 익명 사용자나 remember-me 사용자가 아니면 true를 리턴함
- hasPermission(Object target, Object permission)
    - 사용자가 target에 해당 permission 권한이 있으면 true를 리턴
    - Ex) hasPermission(domainObject, ‘read’)
- hasPermission(Object targetId, String targetType, Object permission)
    - 사용자가 target에 해당 permission 권한이 있으면 true를 리턴함
    - Ex) hasPermission(1, ‘com.example.domain.Message’, ‘read’)