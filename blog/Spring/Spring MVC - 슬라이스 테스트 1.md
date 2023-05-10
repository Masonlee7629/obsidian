태그: #Spring
연결문서: [Spring MVC - 슬라이스 테스트 2](Spring%20MVC%20-%20슬라이스%20테스트%202.md)

# 슬라이스 테스트(Slice Test)

하나의 애플리케이션은 계층별로 역할이 있고, 계층별로 서로 연동되기 때문에 각각의 계층 별로 잘 동작하는지 테스트를 진행한 후에 마지막으로 통합 테스트를 통해서 계층 간의 연동에 문제가 없는지 확인해야 비로소 개발자의 테스트 작업이 마무리 되는것이라고 할 수 있다.  
  

이처럼 개발자가 각 계층에 구현해 놓은 기능들이 잘 동작하는지 특정 계층만 잘라서 테스트하는 것을 **슬라이스 테스트(Slice Test)**라고 한다.  
  

> **참고사항**  
> 통합 테스트는 아니지만 QA 부서에서 본격적으로 전체적인 기능 테스트를 진행하기 전에 애플리케이션의 특정 수정 사항으로 인해 영향을 받을 수 있는 범위에 한해서 제한된 테스트를 진행하기도 한다. 테스트 세계에서는 이를 스모크 테스트(Smoke Test)라고 부른다.

## API 계층 테스트

API 계층의 테스트 대상은 대부분 클라이언트의 요청을 받아들이는 핸들러인 Controller이다.

### Controller 테스트를 위한 테스트 클래스 구조

```
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest       // (1)
@AutoConfigureMockMvc  // (2)
public class ControllerTestDefaultStructure {
    // (3)
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void postMemberTest() {
        // given (4) 테스트용 request body 생성

        // when (5) MockMvc 객체로 테스트 대상 Controller 호출

        // then (6) Controller 핸들러 메서드에서 응답으로 수신한 HTTP Status 및 response body 검증 
    }
}
```

1.  (1)의 `@SpringBootTest` 애너테이션을 Spring Boot 기반의 애플리케이션을 테스트하기 위한 Application Context를 생성한다. Application Context에는 애플리케이션에 필요한 Bean 객체들이 등록되어 있다.
2.  (2)의 `@AutoConfigureMockMvc` 애너테이션은 Controller 테스트를 위한 애플리케이션의 자동 구성 작업을 해준다. (3)의 `MockMvc` 같은 기능을 사용하기 위해서는 반드시 해당 애너테이션을 추가해야 한다.
3.  (3)에서 DI로 주입 받은 `MockMvc`는 Tomcat 같은 서버를 실행하지 않고, Spring 기반 애플리케이션의 Controller를 테스트할 수 있는 완벽한 환경을 지원해주는 일종의 Spring MVC 테스트 프레임워크이다.
4.  Controller를 테스트하기 위해서는 (4)의 단계에서 테스트용 request body를 만들어 준다.  
    **Given-When-Then 패턴에서 Given에 해당된다.**
5.  (5)에서 `MockMvc` 객체를 통해 요청 URI와 HTTP 메서드 등을 지정하고, (4)에서 만든 테스트용 request body를 추가한 뒤에 request를 수행한다.  
    **Given-When-Then 패턴에서 When에 해당된다.**
6.  (6)에서 Controller에서 전달 받은 HTTP Status와 request body 데이터를 통한 검증 작업을 진행한다.  
    **Given-When-Then 패턴에서 Then에 해당된다.**

### Controller 테스트 예시

#### `postMember()` 핸들러 메서드 테스트

```
import com.google.gson.Gson;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.transaction.annotation.Transactional;

import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.startsWith;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.header;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class MemberControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private Gson gson;

    @Test
    void postMemberTest() throws Exception {
        // given  (1)
        MemberDto.Post post = new MemberDto.Post("mason@gmail.com",
                                                        "메이슨",
                                                    "010-1234-5678");
        String content = gson.toJson(post); // (2)

        // when
        ResultActions actions =
                mockMvc.perform(                        // (3)
                                    post("/members")  // (4)
                                        .accept(MediaType.APPLICATION_JSON) // (5)
                                        .contentType(MediaType.APPLICATION_JSON) // (6)
                                        .content(content)   // (7)
                                );

        // then
        actions
                .andExpect(status().isCreated()) // (8)
                .andExpect(header().string("Location", is(startsWith("/members/"))));  // (9)
    }
}
```

위 코드는 MemberController의 postMember() 핸들러 메서드를 테스트 하는 테스트 케이스이다.  
  

Controller를 테스트하는 기본 구조에 테스트 하고자하는 로직들을 포함을 시킨 것이다.  
  

Given-When-Then 구조로 나누었기 때문에 단계적으로 접근하면 테스트 케이스를 조금 더 수월하게 작성할 수 있다.

1.  Given
    -   (1)의 코드는 Given에 해당하며 request body에 포함시키는 요청 데이터와 동일한 역할을 한다.
    -   (2)에서 Gson이라는 JSON 변환 라이브러리를 이용해서 (1)에서 생성한 `MemberDto.Post` 객체를 JSON 포맷으로 변환해준다.
    -   Gson 라이브러리를 사용하기 위해서는 build.gradle의 `dependencies`에`implementation 'com.google.code.gson:gson'`을 추가해야 한다.
2.  When
    -   MockMvc로 테스트 대상 Controller의 핸들러 메서드에 요청을 전송하기 위해서는 (3)과 같이 `perform()` 메서드를 호출해야 하며 `perform()` 메서드 내부에 Controller 호출을 위한 세부적인 정보들이 포함된다.
    -   (4) ~ (7) 까지는 HTTP request에 대한 정보이며, MockMvcRequestBuilders 클래스를 이용해서 빌더 패턴을 통해 request 정보를 채워 넣을 수 있다.
        -   (4)에서 `post()` 메서드를 통해 HTTP POST METHOD와 request URL을 설정한다.
        -   (5)에서 `accept()` 메서드를 통해 클라이언트 쪽에서 리턴 받을 응답 데이터 타입으로 JSON 타입을 설정한다.
        -   (6)에서 `contentType()` 메서드를 통해 서버 쪽에서 처리 가능한 Content Type으로 JSON 타입을 설정한다.
        -   (7)에서 `content()` 메서드를 통해 request body 데이터를 설정한다. request body에 전달하는 데이터는 (2)에서 GSON 라이브러리를 통해 변환된 JSON 문자열이다.
3.  Then
    -   MockMvc의 `perform()` 메서드는 `ResultActions` 타입의 객체를 리턴하는데, 이 `ResultActions` 객체를 이용하여, 전송한 request에 대한 검증을 수행할 수 있다.
    -   (8)에서 `andExpect()` 메서드를 통해 파라미터로 입력한 매처(Matcher)로 예상되는 기대 값을 검증할 수 있다. (8)에서는 `status().isCreated()`를 통해 response status가 201(Created)인지 매치시킨다.
    -   (9)에서 `header().string("Location", is(startsWith("/v11/members/")))`을 통해 HTTP header에 추가된 Location의 문자열 값이 `"/members/"`로 시작하는지 검증한다. Location header가 예상하는 값과 일치한다는 것은 백엔드 측에 리소스가 잘 생성되었다는 것을 의미한다.

#### `getMember()` 핸들러 메서드 테스트

```
import com.google.gson.Gson;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.util.UriComponentsBuilder;

import java.net.URI;

import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.startsWith;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;

@Transactional
@SpringBootTest
@AutoConfigureMockMvc
class MemberControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private Gson gson;

    ...
    ...

    @Test
    void getMemberTest() throws Exception {
        // =================================== (1) postMember()를 이용한 테스트 데이터 생성 시작
        // given
        MemberDto.Post post = new MemberDto.Post("mason@gmail.com","Mason","010-1111-1111");
        String postContent = gson.toJson(post);

        ResultActions postActions =
                mockMvc.perform(
                        post("/members")
                                .accept(MediaType.APPLICATION_JSON)
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(postContent)
                );
        // =================================== (1) postMember()를 이용한 테스트 데이터 생성 끝

        // (2)
        String location = postActions.andReturn().getResponse().getHeader("Location"); // "/members/1"

        // when / then
        mockMvc.perform(
                        get(location)      // (3)
                                .accept(MediaType.APPLICATION_JSON)
                )
                .andExpect(status().isOk())    // (4)
                .andExpect(jsonPath("$.data.email").value(post.getEmail()))   // (5)
                .andExpect(jsonPath("$.data.name").value(post.getName()))     // (6)
                .andExpect(jsonPath("$.data.phone").value(post.getPhone()));  // (7)
    }
}
```

위 코드는 MemberController의 `getMember()` 핸들러 메서드를 테스트 하는 테스트 케이스이다.

1.  Given
    -   (1)의 코드 영역은 `postMember()` 메서드를 테스트할 때와 동일한 코드이다. (1)을 통해서 테스트 데이터를 백엔드 서버 측의 데이터베이스에 먼저 저장한다.
    -   (2)에서는 `postMember()`의 response에 전달되는 Location header 값을 가져오는 로직이다. (2)와 같이 `postActions.andReturn().getResponse().getHeader("Location")`로 접근해서 Location header의 값을 얻어올 수 있다.
2.  When
    -   (3)에서는 (2)에서 얻은 Location header의 값을 `get(location)`으로 전달한다. Location header에서 얻게되는 값이 (1)에서 등록한 resource(회원 정보)의 위치`(”/members/1”)`를 의미하기 때문에 `get(…)`의 URI로 전달하면 됩니다.
3.  Then
    -   (4)에서는 기대하는 HTTP Status가 `200 OK`인지를 검증한다.
    -   (5)에서는 `jsonPath()` 메서드를 통해 response body(JSON 형식)의 각 프로퍼티 중에서 응답으로 전달 받는 email 값이 request body로 전송한 email과 일치하는지 검증한다.
    -   (6)에서는 `jsonPath()` 메서드를 통해 response body(JSON 형식)의 각 프로퍼티 중에서 응답으로 전달 받는 name 값이 request body로 전송한 name과 일치하는지 검증한다.
    -   (7)에서는 `jsonPath()` 메서드를 통해 response body(JSON 형식)의 각 프로퍼티 중에서 응답으로 전달 받는 phone 값이 request body로 전송한 phone과 일치하는지 검증한다.

> -   response body 응답 데이터에 포함된 한글이 깨질 경우

```
server:
  servlet:
    encoding:
      force-response: true
```

response body를 출력할 때, JSON 데이터에서 한글이 깨져 보일 경우, application.yml 파일에 설정을 추가한다.

### Contoller 테스트 예시의 문제점

예시대로 Controller 테스트를 진행할 경우 Controller만 테스트하는 것이 아니라 애플리케이션의 전체 로직을 모두 실행하게 된다.  
즉, 실제 테스트 해야할 계층은 API 계층이지만 서비스 계층이나 데이터 액세스 계층까지 불필요한 로직이 수행된다는 것이다.  
따라서 완전한 슬라이스 테스트라고 보기에는 힘들다.  
해당 문제는 **Mock(가짜) 객체를 사용해 계층 간의 연결을 끊어줌으로써 해결이 가능하다.**

> **@WebMvcTest를 이용한 Controller 테스트**  
> Spring 에서는 Controller를 테스트 하기 위한 전통적인 방법으로 `@WebMvcTest` 애너테이션을 사용할 수 있다.  
> 하지만 `@WebMvcTest` 애너테이션을 사용할 경우, Controller에서 의존하는 컴포넌트들을 모두 일일이 설정해 주어야 하는 불편함이 있다.  
> 예를 들어 MemberController에서 사용되는 MemberService Bean, MemberMapper Bean 객체 등을 테스트 클래스에서 사용할 수 있도록 설정해 주어야 한다.  
> 또한 때에 따라서 데이터액세스 계층에서 의존하는 설정이나 의존 객체들도 모두 설정해 주어야 할 수도 있다.  
> 이런 이유로 예시에서는 `@SpringBootTest`, `@AutoConfigureMockMvc`를 이용해서 Controller 테스트를 위한 구성의 복잡함을 해결하고 있다.  
> `@WebMvcTest`와 `@SpringBootTest`는 각각 장단점이 존재하며, 상황에 맞게 적절하게 사용할 수 있다.

