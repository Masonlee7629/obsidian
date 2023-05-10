태그: #Spring
연결문서: [Spring MVC - API 계층 2](Spring%20MVC%20-%20API%20계층%202.md)

# Controller

# Controller 클래스 설계 및 구조 생성

[##_Image|kage@bW6IGZ/btsaQLkHOpU/0COLi2zkgu068LrKs4TCPk/img.png|CDM|1.3|{"originWidth":1920,"originHeight":1080,"style":"alignCenter","width":null}_##]

  
  

위 그림은 계층형 아키텍처에서 API 계층의 모습이다.  
API 계층은 클라이언트의 요청을 직접적으로 전달 받는 계층이다.  
  

애플리케이션을 제작하기 위해서 실질적으로 가장 먼저 해야되는 일은 **애플리케이션의 경계를 설정하는 것**과 애플리케이션 기능 구현을 위한 요구 사항을 수집하는 일이다.

### 패키지 구조 생성

Spring Boot 기반의 애플리케이션에서 주로 사용되는 Java 패키지 구조는 기능 기반 패키지 구조(package-by-feature)와 계층 기반 패키지 구조(package-by-layer)가 있다.  
  

#### 기능 기반 패키지 구조(package-by-feature)

기능 기반 패키지 구조란 말 그대로 애플리케이션의 패키지를 애플리케이션에서 구현해야 하는 기능을 기준으로 패키지를 구성하는 것이다.  
  

이렇게 나누어진 패키지 안에는 하나의 기능을 완성하기 위한 계층별(API 계층, 서비스 계층, 데이터 액세스 계층) 클래스들이 모여있다.  
  

[##_Image|kage@PX549/btsaEhYVe3x/sNvDJKnrkH5yZ9t4K2LOJ0/img.png|CDM|1.3|{"originWidth":178,"originHeight":247,"style":"alignCenter","width":null}_##]

#### 계층 기반 패키지 구조(package-by-layer)

계층 기반 패키지 구조란 패키지를 하나의 계층(Layer)으로 보고 클래스들을 계층별로 묶어서 관리하는 구조를 말한다.  
  

[##_Image|kage@dXOLY9/btsayWnwiVQ/GZnLw5YeqPZOhepAddnev1/img.png|CDM|1.3|{"originWidth":171,"originHeight":296,"style":"alignCenter","width":null}_##]

### Controller 설계

REST API 기반의 애플리케이션에선 일반적으로 애플리케이션이 제공해야 될 기능을 리소스로 분류한다.  
그리고 리소스에 해당하는 Controller 클래스를 작성한다.

### 엔트리포인트(Entrypoint) 클래스 작성

Spring Boot 기반의 애플리케이션이 정상적으로 실행되기 위해서 가장 먼저 해야될 일은 main() 메서드가 포함된 애플리케이션의 엔트리포인트(애플리케이션 시작점)를 작성하는 것이다.  
  

'Spring Initializr'를 통해 생성한 프로젝트에는 엔트리포인트 클래스가 이미 작성되어 있다.

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication   // (1)
public class SpringBootApplication {

    public static void main(String[] args) {
    // (2)
        SpringApplication.run(SpringBootApplication.class, args);
    }

}
```

(1) @SpringBootApplication  
@SpringBootApplication 은 코드 상에서는 보이지 않지만 내부적으로 세가지 일을 해준다.

-   자동 구성을 활성화한다.
-   애플리케이션 패키지 내에서 @Component가 붙은 클래스를 검색(scan)한 후, Spring Bean으로 등록하는 기능을 활성화한다.
-   @Configuration 이 붙은 클래스를 자동으로 찾아주고, 추가적으로 Spring Bean을 등록하는 기능을 활성화한다.

(2) SpringApplication.run(SpringBootApplication.class, args);  
Spring 애플리케이션을 부트스트랩하고, 실행하는 역할을 한다.

-   부트스트랩(Bootstrap)
    -   애플리케이션이 실행되지 전에 여러가지 설정 작업을 수행하여 실행 가능한 애플리케이션으로 만드는 단계를 의미한다.

### Controller 구조 작성

```
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController                   // (1)
@RequestMapping("/v1/members")    // (2)
public class MemberController {
}
```

(1) @RestController

-   Spring MVC에서는 특정 클래스에 @RestController를 추가하면 해당 클래스가 REST API의 리소스(자원, Resource)를 처리하기 위한 API 엔드포인트로 동작함을 정의한다.
-   @RestController가 추가된 클래스는 애플리케이션 로딩 시, Spring Bean으로 등록해준다.

(2) @RequestMapping

-   @RequestMapping은 클라이언트의 요청과 클라이언트 요청을 처리하는 핸들러 메서드(Handler Method)를 매핑해주는 역할을 한다.
-   Controller 클래스 레벨에 추가하여 클래스 전체에 사용되는 공통 URL(Base URL) 설정한다.

---

## 핸들러 메서드(Handler Method)

Controller 클래스 안에 구현된 클라이언트의 요청을 처리하는 메서드를 의미한다.

### 핸들러 메서드 적용

핸들러 메서드를 작성하기 전에 먼저 각 핸들러 메서드에서 필요한 기능별 정보들을 정의해야 한다.

#### 핸들러 메서드 작성

-   @PostMapping
    -   클라이언트의 요청 데이터(Request Body)를 서버에 생성(등록)할 때 사용하는 애너테이션
    -   핸들러 메서드 레벨에 추가
    -   일반적으로 POST Method를 처리하는 핸들러 메서드는 데이터 생성 후 클라이언트에 생성한 데이터를 리턴하는 것이 관례

-   @GetMapping
    -   클라이언트가 서버에 리소스를 조회할 때 사용하는 애너테이션
    -   핸들러 메서드 레벨에 추가
    -   @GetMapping("/{member-id}")와 같이 애트리뷰트를 사용하여 전체 HTTP URI의 일부를 지정 가능
    -   클라이언트에서 @GetMapping("/{member-id}") 애너테이션을 사용한 핸들러 메서드에 요청을 보낼 경우 최종 URI : “/{member-id}”

-   @PatchMapping
    -   클라이언트의 요청 데이터로 서버의 데이터를 수정하는 애너테이션
    -   핸들러 메서드 레벨에 추가
    -   @PatchMapping(“/{member-id}”) 와 같이 애트리뷰트를 사용하여 해당 URI에 등록된 데이터 수정

-   @DeleteMapping
    -   클라이언트가 서버의 리소스를 삭제할 때 사용하는 애너테이션
    -   핸들러 메서드 레벨에 추가
    -   @DeleteMapping(“/{member-id}”) 와 같이 애트리뷰트를 사용하여 해당 URI에 등록된 데이터 삭제

-   @RequestParam
    -   핸들러 메서드의 파라미터 종류 중 하나
    -   주로 클라이언트에서 전송하는 요청 데이터를 쿼리 파라미터, 폼 데이터, x-www-form-urlencoded 형식으로 전송하면 이를 서버에서 전달 받을 때 사용하는 애너테이션

-   @PathVariable
    -   핸들러 메서드의 파라미터 종류 중 하나
    -   @PathVariable의 괄호 안에 입력한 문자열 값은 @GetMapping(“/{member-id}”) 처럼 중괄호({ }) 안의 문자열과 동일해야 함
    -   두 문자열이 다르면 MissingPathVariableException이 발생

---

## 응답 데이터에 ResponseEntity 적용

-   리턴 값으로 ResponseEntity 객체를 리턴
-   응답 데이터로 Map 객체를 리턴하면 Spring MVC 내부적으로 JSON 형식의 데이터를 생성
-   클래스 레벨의 @RequestMapping 에 ‘produces’ 애트리뷰트를 지정할 필요가 없음

**생성자 파라미터로 응답 데이터와 HTTP 응답 상태를 전달하여 클라이언트의 요청을 서버가 어떻게 처리했는지 확인이 가능하다.**