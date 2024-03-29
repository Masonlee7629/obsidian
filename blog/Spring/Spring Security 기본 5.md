태그: #Spring 
연결문서: [Spring Security 기본 6](Spring%20Security%20기본%206.md)

# Spring Security

## Filter와 FilterChain 구현

### Filter

![](image_15.png)

<br>

- 서블릿 필터(Servlet Filter)는 서블릿 기반 애플리케이션의 엔드포인트에 요청이 도달하기 전에 중간에서 요청을 가로챈 후 어떤 처리를 할 수 있도록 해주는 Java의 컴포넌트
- 클라이언트가 서버 측 애플리케이션으로 요청을 전송하면 제일 먼저 Servlet Filter를 거치게 되며, Filter에서의 처리가 모두 완료되면 DispatcherServlet에서 클라이언트의 요청을 핸들러에 매핑하기 위한 다음 작업을 진행함

<br>

### FilterChain
- 여러 개의 Filter가 체인을 형성하고 있는 Filter의 묶음을 의미함

<br>

### Filter와 FilterChain의 특성
- Servlet FilterChain은 요청 URI path를 기반으로 HttpServletRequest를 처리하기 때문에 클라이언트가 서버 측 애플리케이션에 요청을 전송하면 서블릿 컨테이너는 요청 URI의 경로를 기반으로 어떤 Filter와 어떤 Servlet을 매핑할지 결정함
- Filter는 Filter Chain 안에서 순서를 지정할 수 있으며 지정한 순서에 따라서 동작하게 할 수 있음
- Filter Chain에서 Filter의 순서는 매우 중요하며 Spring Boot에서 여러 개의 Filter를 등록하고 순서를 지정하기 위해서는 두 가지 방법을 적용할 수 있음
    1. Spring Bean으로 등록되는 Filter에 @Order 애너테이션을 추가하거나 Orderd 인터페이스를 구현해서 Filter의 순서를 지정할 수 있음
    2. FilterRegistrationBean을 이용해 Filter의 순서를 명시적으로 지정할 수 있음

<br>

### Filter 인터페이스

- Servlet Filter의 기본 구조
    - (1)의 init() 메서드에서는 생성한 Filter에 대한 초기화 작업을 진행할 수 있음
    - (2)의 doFilter() 메서드에서는 해당 Filter가 처리하는 실질적인 로직을 구현함
        - (2-1)에는 request를 이용해 (2-2)의 chain.doFilter(request, response)가 호출되기 전에 할 수 있는 전처리 작업에 대한 코드를 구현할 수 있음
        - (2-3)에는 response를 이용해 (2-2)의 chain.doFilter(request, response)가 호출된 이후에 할 수 있는 후처리 작업에 대한 코드를 구현할 수 있음
    - (3)의 destroy() 메서드는 Filter가 컨테이너에서 종료될 때 호출되는데 주로 Filter가 사용한 자원을 반납하는 처리 등의 로직을 작성하고자 할 때 사용됨

```java
public class FirstFilter implements Filter {
     // (1) 초기화 작업
     public void init(FilterConfig filterConfig) throws ServletException {
        
     }
     
     // (2)
     public void doFilter(ServletRequest request,
                          ServletResponse response,
                          FilterChain chain)
                          throws IOException, ServletException {
        // (2-1) 이곳에서 request(ServletRequest)를 이용해 다음 Filter로 넘어가기 전처리 작업을 수행한다.

        // (2-2)
        chain.doFilter(request, response);

        // (2-3) 이곳에서 response(ServletResponse)를 이용해 response에 대한 후처리 작업을 할 수 있다.
     }
     
     // (3)
     public void destroy() {
        // (5) Filter가 사용한 자원을 반납하는 처리
     }
  }
```

<br>

---

<br>

### Filter 예제

#### 1. 첫 번째 Filter 구현

```java
import javax.servlet.*;
import java.io.IOException;

public class FirstFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
        System.out.println("FirstFilter 생성됨");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("========First 필터 시작========");
        chain.doFilter(request, response);
        System.out.println("========First 필터 종료========");
    }

    @Override
    public void destroy() {
        System.out.println("FirstFilter Destory");
        Filter.super.destroy();
    }
}
```

<br>

#### 2. FirstFilter를 적용하기 위한 FilterConfiguration 구성

```java
import book.study.security.FirstFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfiguration {

    @Bean
    public FilterRegistrationBean<FirstFilter> firstFilterRegister()  {
        FilterRegistrationBean<FirstFilter> registrationBean = new FilterRegistrationBean<>(new FirstFilter());
        return registrationBean;
    }
}
```

<br>

#### 3. 애플리케이션 실행

- 애플리케이션 실행 시, 가장 먼저 init() 메서드가 실행되며 아래와 같은 로그가 출력됨

```java
FirstFilter 생성됨
```

<br>

- 컨트롤러의 핸들러 메서드로 요청을 보낼 경우
    - doFilter -> controller 동작 -> destroy 메서드의 형태로 Filter가 동작하면서 아래 코드와 유사한 로그가 출력됨

```java
========First 필터 시작========
Controller 호출
========First 필터 종료========
```

<br>

#### 4. 두 번째 Filter 구현

```java
import javax.servlet.*;
import java.io.IOException;

public class SecondFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
        System.out.println("SecondFilter가 생성됨");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("==========Second 필터 시작==========");
        chain.doFilter(request, response);
        System.out.println("==========Second 필터 종료==========");
    }

    @Override
    public void destroy() {
        System.out.println("SecondFilter가 사라짐");
        Filter.super.destroy();
    }
}
```

<br>

#### 5. FilterConfiguration에 두 번째 Filter 등록
- 두 개의 Filter가 지정된 순서로 실행되도록 (1), (2)와 같이 registrationBean.setOrder() 메서드로 순서를 지정할 수 있음
- registrationBean.setOrder()의 파라미터로 지정한 숫자가 적은 숫자일수록 먼저 실행됨

```java
import book.study.security.FirstFilter;
import book.study.security.SecondFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Config {

    @Bean
    public FilterRegistrationBean<FirstFilter> firstFilterRegister()  {
        FilterRegistrationBean<FirstFilter> registrationBean = new FilterRegistrationBean<>(new FirstFilter());
        registrationBean.setOrder(1); // (1)
        return registrationBean;
    }

    @Bean
    public FilterRegistrationBean<SecondFilter> secondFilterRegister()  {
        FilterRegistrationBean<SecondFilter> registrationBean = new FilterRegistrationBean<>(new SecondFilter());
        registrationBean.setOrder(2); // (2)
        return registrationBean;
    }

}
```

<br>

#### 6. 애플리케이션 재실행

```java
========First 필터 시작========
==========Second 필터 시작==========
Controller 호출
==========Second 필터 종료==========
========First 필터 종료========
```