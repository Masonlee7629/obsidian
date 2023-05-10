태그: #Spring
연결문서: [Spring MVC - API 계층 1](Spring%20MVC%20-%20API%20계층%201.md)

# Spring MVC 아키텍처

## Spring MVC

Spring에서 지원하는 모든 기능들을 포함해서 Spring Framework라고 부른다.  
  

Spring의 모듈 중에는 웹 계층을 담당하는 몇 가지 모듈이 있다. 특히 **서블릿(Servlet) API**를 기반으로 클라이언트의 요청을 처리하는 모듈이 있는데, 이 모듈의 이름이 Spring-webmvc이다.  
  

개발자들 사이에서는 **Spring Web MVC**를 줄여서 **Spring MVC**라고 부르며, Spring MVC가 **웹 프레임워크**의 한 종류이기 때문에 Spring MVC 프레임워크라고도 부른다.

-   Spring MVC는 클라이언트의 요청을 편리하게 처리해주는 프레임 워크이다.
-   서블릿(Servlet)
    -   서블릿은 클라이언트의 요청을 처리하도록 특정 규약에 맞추어서 Java 코드로 작성하는 클래스 파일이다.  
        그리고 **아파치 톰캣(Apache Tomcat)**은 이러한 서블릿들이 웹 애플리케이션으로 실행이 되도록 해주는 서블릿 컨테이너(Servlet Container) 중 하나이다.

### Model

Model은 Spring MVC에서 M에 해당된다.  
  

Spring MVC 기반의 웹 애플리케이션이 클라이언트의 요청을 전달 받으면 요청 사항을 처리하기 위한 작업을 한다..  
  

이렇게 처리한 작업의 결과 데이터를 클라이언트에게 응답으로 돌려줘야 하는데, 이 때 클라이언트에게 응답으로 돌려주는 **작업의 처리 결과 데이터를 Model**이라고 한다.

---

### View

View는 Spring MVC에서 V에 해당된다.  
  

View는 Model 데이터를 이용해서 웹 브라우저 같은 클라이언트 애플리케이션의 화면에 보여지는 리소스(Resource)를 제공하는 역할을 한다.  
  

Spring MVC에는 다양한 View 기술이 포함되어 있고, View의 형태는 다음과 같다.

-   HTML 페이지의 출력
    -   클라이언트 애플리케이션에 보여지는 HTML 페이지를 직접 랜더링해서 클라이언트 측에 전송하는 방식이다.
    -   기본적인 HTML 태그로 구성된 페이지에 Model 데이터를 채워 넣은 후, 최종적인 HTML 페이지를 만들어서 클라이언트 측에 전송해준다.
    -   Spring MVC에서 지원하는 HTML 페이지 출력 기술에는 Thymeleaf, FreeMaker, JSP + JSTL, Tiles 등이 있다.
-   PDF, Excel 등의 문서 형태로 출력
    -   Model 데이터를 가공해서 PDF 문서나 Excel 문서를 만들어서 클라이언트 측에 전송하는 방식이다.
    -   문서 내에서 데이터가 동적으로 변경되어야 하는 경우 사용할 수 있는 방식이다.
-   XML, JSON 등 특정 형식의 포맷으로의 변환
    -   Model 데이터를 특정 프로토콜 형태로 변환해서 변환된 데이터를 클라이언트 측에 전송하는 방식이다.
    -   이 방식은 특정 형식의 데이터만 전송하고, 프론트엔드 측에서 이 데이터를 기반으로 HTML 페이지를 만드는 방식이다.
    -   장점
        -   프론트엔드 영역과 백엔드 영역이 명확하게 구분되므로 개발 및 유지보수가 상대적으로 용이하다.
        -   프론트엔드 측에서 비동기 클라이언트 애플리케이션을 만드는 것이 가능해진다.

---

### Controller

Controller는 Spring MVC에서 C에 해당된다.  
  

**Controller는 클라이언트 측의 요청을 직접적으로 전달 받는 엔드포인트(Endpoint)로써 Model과 View의 중간에서 상호 작용을 해주는 역할을 한다.**  
  

즉, 클라이언트 측의 요청을 전달 받아서 비즈니스 로직을 거친 후에 Model 데이터가 만들어지면, 이 Model 데이터를 View로 전달하는 역할을 한다.

---

## Spring MVC의 동작 방식과 구성 요소

[##_Image|kage@I2lDz/btsaR1gy2lX/Eu6uOba2dgnbN11nSyCA20/img.png|CDM|1.3|{"originWidth":1041,"originHeight":600,"style":"alignCenter","width":null}_##]

(1) 클라이언트가 요청을 전송하면 DispatcherServlet이란 클래스에 요청이 전달된다.

(2) DispatcherServlet은 **클라이언트의 요청을 처리할 Controller에 대한 검색**을 HandlerMapping 인터페이스에게 요청한다.

(3) HandlerMapping은 **클라이언트 요청과 매핑되는 핸들러 객체를 다시 DispatcherServlet에게 리턴**해준다.  
핸들러 객체는 해당 핸들러의 Handler 메서드 정보를 포함하고 있다. Handler 메서드는 Controller 클래스 안에 구현된 요청 처리 메서드를 의미한다.

(4) 실제로 클라이언트의 요청을 처리할 Handler 메서드를 찾아서 호출해야 하며, DispatcherServlet은 Handler 메서드를 직접 호출하지 않고, HandlerAdpater에게 Handler 메서드 호출을 위임한다.

(5) HandlerAdapter는 DispatcherServlet으로부터 전달 받은 Controller 정보를 기반으로 해당 Controller의 Handler 메서드를 호출한다.

(6) Controller의 Handler 메서드는 비즈니스 로직 처리 후 리턴 받은 Model 데이터를 HandlerAdapter에게 전달한다.

(7) HandlerAdapter는 전달받은 Model 데이터와 View 정보를 다시 DispatcherServlet에게 전달한다.

(8) DispatcherServlet은 전달 받은 View 정보를 다시 ViewResolver에게 전달해서 View 검색을 요청한다.

(9) ViewResolver는 View 정보에 해당하는 View를 찾아서 View를 다시 리턴해준다.

(10) DispatcherServlet은 ViewResolver로부터 전달 받은 View 객체를 통해 Model 데이터를 넘겨주면서 클라이언트에게 전달할 응답 데이터 생성을 요청한다.

(11) View는 응답 데이터를 생성해서 다시 DispatcherServlet에게 전달한다.

(12) DispatcherServlet은 View로부터 전달 받은 응답 데이터를 최종적으로 클라이언트에게 전달한다.

#### DispatcherServlet의 역할

Spring MVC의 요청 처리 흐름을 보면 DispatcherServlet이 굉장히 많을 일을 하는 것처럼 보인다.  
  

클라이언트로부터 요청을 전달 받으면 HandlerMapping, HandlerAdapter, ViewResolver, View 등 대부분의 Spring MVC 구성 요소들과 상호 작용을 하고 있는 것을 볼 수 있다.  
  

그런데 DispatcherServlet이 굉장히 바빠보이지만 실제로 요청에 대한 처리는 다른 구성 요소들에게 위임(Delegate)하고 있다.  
  

이처럼 DispatcherServlet이 애플리케이션의 가장 앞단에 배치되어 다른 구성요소들과 상호작용하면서 클라이언트의 요청을 처리하는 패턴을 Front Controller Pattern이라고 한다.

#### Spring MVC의 요청 처리 흐름 간략화

1.  DispatcherServlet은 HandlerMapping 인터페이스에게 Controller의 검색을 위임한다.
2.  DispatcherServlet은 검색된 Controller 정보를 토대로 HandlerAdapter 인터페이스에게 Controller 클래스내에 있는 Handler 메서드의 호출을 위임한다.
3.  HandlerAdapter 인터페이스는 Controller 클래스의 Handler 메서드를 호출한다.
4.  DispatcherServlet은 ViewResolver에게 View의 검색을 위임한다.
5.  DispatcherServlet은 View에게 Model 데이터를 포함한 응답 데이터 생성을 위임한다.
6.  DispatcherServlet은 최종 응답 데이터를 클라이언트에게 전달한다.