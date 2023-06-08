태그: #Spring 
연결문서: [Spring Security 기본 5](Spring%20Security%20기본%205.md)

# Spring Security

## Spring Security의 웹 요청 처리 흐름

### 보안이 적용된 웹 요청의 일반적인 처리 흐름

![](image_11.png)

- (1)에서 사용자가 보호된 리소스를 요청함
- (2)에서 인증 관리자 역할을 하는 컴포넌트가 사용자의 크리덴셜(Credential)을 요청함
    - 사용자의 크리덴셜(Credential)이란 해당 사용자를 증명하기 위한 구체적인 수단을 의미함
    - 일반적으로는 사용자의 패스워드가 크리덴셜에 해당함
- (3)에서 사용자는 인증 관리자에게 크리덴셜을 제공함
- (4)에서 인증 관리자는 크리덴셜 저장소에서 사용자의 크리덴셜을 조회함
- (5)에서 인증 관리자는 사용자가 제공한 크리덴셜과 크리덴셜 저장소에 저장된 크리덴셜을 비교해 검증 작업을 수행함
- (6) 유효한 크리덴셜이 아니라면 Exception을 Throw 함
- (7) 유효한 크리덴셜이라면 (8)에서 접근 결정 관리자 역할을 하는 컴포넌트는 사용자가 적절한 권한을 부여받았는지 검증함
- (9) 적절한 권한을 부여 받지 못한 사용자라면 Exception을 Throw 함
- (10) 적절한 권한을 부여 받은 사용자라면 보호된 리소스의 접근을 허용함

<br>

### 웹 요청에서의 서블릿 필터와 필터 체인의 역할

![](image_12.png)

<br>

- 사용자의 웹 요청이 Controller 같은 엔드포인트를 거쳐 접근하려는 리소스에 도달하기 전에 인증 관리자나 접근 결정 관리자 같은 컴포넌트가 중간에 웹 요청을 가로채 사용자의 크리덴셜과 접근 권한을 검증함
- 서블릿 기반 애플리케이션의 경우, 애플리케이션의 엔드포인트에 요청이 도달하기 전에 중간에서 요청을 가로챈 후 어떤 처리를 할 수 있는 적절한 포인트를 제공하는데 그것을 서블릿 필터(Servlet Filter)라고 함
    - 서블릿 필터는 자바에서 제공하는 API이며, javax.servlet 패키지에 인터페이스 형태로 정의되어 있음
    - javax.servlet.Filter 인터페이스를 구현한 서블릿 필터는 웹 요청(request)을 가로채어 어떤 처리(전처리)를 할 수 있으며, 또한 엔드포인트에서 요청 처리가 끝난 후 전달되는 응답(reponse)을 클라이언트에게 전달하기 전에 어떤 처리(후처리)를 할 수 있음
- 서블릿 필터는 각각의 필터들이 doFilter() 라는 메서드를 구현해야 하며, doFilter() 메서드 호출을 통해 필터 체인을 형성하게 됨
- 만약, Filter 인터페이스를 구현한 다수의 Filter 클래스를 구현했다면, 생성한 서블릿 필터에서 우리가 작성한 특별한 작업을 수행한 뒤, HttpServlet을 거쳐 DispatcherServlet에 요청이 전달되며, 반대로 DispatcherServlet에서 전달한 응답에 대해 역시 특별한 작업을 수행할 수 있음

<br>

### Spring Security에서의 필터 역할

![](image_13.png)

<br>

- DelegatingFilterProxy와 FilterChainProxy 클래스는 Filter 인터페이스를 구현하기 때문에 엄연히 서블릿 필터로써의 역할을 함
- DelegatingFilterProxy
    - 서블릿 필터와 연결되는 Spring Security만의 필터를 ApplicationContext에 Bean으로 등록한 후에 이 Bean들을 이용해서 보안과 관련된 여러 가지 작업을 처리하게 되는데 DelegatingFilterProxy 가 Bean으로 등록된 Spring Security의 필터를 사용하는 시작점
    - DelegatingFilterProxy라는 이름에서 알 수 있듯이 보안과 관련된 어떤 작업을 처리하는 것이 아니라 서블릿 컨테이너 영역의 필터와 ApplicationContext에 Bean으로 등록된 필터들을 연결해 주는 브리지 역할을 함

<br>

![](image_14.png)

- FilterChainProxy
    - Spring Security에서 보안을 위한 작업을 처리하는 필터의 모음
    - Spring Security의 Filter를 사용하기 위한 진입점으로, FilterChainProxy부터 Spring Security에서 제공하는 보안 필터들이 필요한 작업을 수행함
    - Spring Security의 Filter Chain은 URL 별로 여러 개 등록할 수 있으며, Filter Chain이 있을 때 어떤 Filter Chain을 사용할지는 FilterChainProxy가 결정하며, 가장 먼저 매칭된 Filter Chain을 실행함
- FilterChainProxy 예시
    - /api/&#42;&#42; 패턴의 Filter Chain이 있고, /api/message URL 요청이 전송하는 경우
        - /api/&#42;&#42; 패턴과 제일 먼저 매칭되므로, 디폴트 패턴인 /&#42;&#42;도 일치하지만 가장 먼저 매칭되는 /api/&#42;&#42; 패턴과 일치하는 Filter Chain만 실행함
    - /message/&#42;&#42; 패턴의 Filter Chain이 없는데 /message/ URL 요청을 전송하는 경우
        - 매칭되는 Filter Chain이 없으므로 디폴트 패턴인 /&#42;&#42; 패턴의 Filter Chain을 실행

<br>

### SpringSecurity에서 지원하는 Filter의 종류
- Spring Security는 보안을 위한 특정 작업을 수행하기 위한 다양한 Filter를 지원하는데 그 수가 굉장히 많음
- Spring Security의 Filter가 각각 어떤 역할을 수행하는지는 전부 다 알 필요는 없으며, 필요한 상황이 되었을 때 그때그때 적용해도 상관 없음
- Spring Security의 Filter는 항상 모든 Filter가 수행되는 것이 아니라 프로젝트 구성 및 설정에 따라 일부의 Filter만 활성화되어 있기 때문에 직접적으로 개발자가 핸들링할 필요가 없는 Filter들이 대부분으로 개발자가 Custom Filter를 작성하고 등록할 경우, 기존 필터들 사이에서 우선순위를 적용해 수행되어야 할 필요가 있는 경우에 참고해서 적용