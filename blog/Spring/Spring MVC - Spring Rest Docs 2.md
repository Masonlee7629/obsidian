태그: #Spring 
연결문서: [Spring MVC - Spring Rest Docs 3](Spring%20MVC%20-%20Spring%20Rest%20Docs%203.md)

# Spring Rest Docs

## Controller 테스트 케이스에 Spring RestDocs 적용

### API 문서 생성을 위한 슬라이스 테스트 케이스 작성

#### API 문서 생성을 위한 테스트 케이스 기본 구조

```
import com.codestates.member.controller.MemberController;
import com.codestates.member.mapper.MemberMapper;
import com.codestates.member.service.MemberService;
import com.google.gson.Gson;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.data.jpa.mapping.JpaMetamodelMappingContext;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;

@WebMvcTest(MemberController.class)   // (1)
@MockBean(JpaMetamodelMappingContext.class)   // (2)
@AutoConfigureRestDocs    // (3)
public class MemberControllerRestDocsTest {
    @Autowired
    private MockMvc mockMvc;  // (4)

    @MockBean
      // (5) 테스트 대상 Controller 클래스가 의존하는 객체를 Mock Bean 객체로 주입 받기

    @Test
    public void postMemberTest() throws Exception {
        // given
        // (6) 테스트 데이터 

        // (7) Mock 객체를 이용한 Stubbing

        // when
        ResultActions actions =
                mockMvc.perform(
                     // (8) request 전송
                );

        // then
        actions
                .andExpect(// (9) response에 대한 기대 값 검증)
                .andDo(document(
                            // (10) API 문서 스펙 정보 추가
                 ));
    }
}
```

1.  (1)에서 `@SpringBootTest` 애너테이션을 사용하지 않고, `@WebMvcTest` 애너테이션을 사용한다.  
    `@WebMvcTest` 애너테이션은 `Controller`를 테스트 하기 위한 전용 애너테이션이며, `@WebMvcTest` 애너테이션의 괄호 안에는 테스트 대상 `Controller` 클래스를 지정한다.
2.  (2)는 JPA에서 사용하는 `Bean` 들을 `Mock` 객체로 주입해주는 설정이다.  
    Spring Boot 기반의 테스트는 항상 최상위 패키지 경로에 있는 xxxxxxxApplication 클래스를 찾아서 실행한다.

```
@EnableJpaAuditing
@SpringBootApplication
public class Section3Week3RestDocsApplication {

    public static void main(String[] args) {
        SpringApplication.run(Section3Week3RestDocsApplication.class, args);
    }

}
```

`@EnableJpaAuditing` 을 xxxxxxApplication 클래스에 추가하게 되면 JPA와 관련된 Bean을 필요로 하기 때문에 `@WebMvcTest` 애너테이션을 사용해서 테스트를 진행 할 경우에는 (2)와 같이 `JpaMetamodelMappingContext`를 `Mock` 객체로 주입해 주어야 합니다.  
  

3.  (3)에서는 Spring Rest Docs에 대한 자동 구성을 위해 `@AutoConfigureRestDocs`를 추가해준자.
4.  (4)에서 `MockMvc` 객체를 주입 받는다.
5.  (5)에서는 `Controller` 클래스가 의존하는 객체(주로 서비스 클래스, Mapper)의 의존성을 제거하기 위해 `@MockBean` 애너테이션을 사용하여 `Mock` 객체를 주입 받는다.
6.  (6)에서는 HTTP Request에 필요한 request body나 query parameter, path variable 등의 데이터를 추가한다.
7.  (7)에서는 (5)에서 주입 받은 `Mock` 객체가 동작하도록 Mockito에서 지원하는 `given()` 등의 메서드로 Stubbing 해준다.
8.  (8)에서는 MockMvc의 `perform()` 메서드로 request를 전송한다.
9.  (9)에서는 response를 검증한다.
10.  (10)에서 테스트 수행 이후, API 문서를 자동 생성하기 위한 해당 `Controller` 핸들러 메서드의 API 스펙 정보를 `document(...)`에 추가해준다.  
    `document(…)` 메서드는 API 문서를 생성 하기 위해 Spring Rest Docs에서 지원하는 메서드이며, `.andDo(…)` 메서드는 `andExpect()`처럼 어떤 검증 작업을 하는 것이 아니라 일반적인 동작을 정의하고자 할 때 사용된다.

#### `@SpringBootTest` vs `@WebMvcTest`

`@SpringBootTest` 애너테이션은 `@AutoConfigureMockMvc` 과 함께 사용되어 `Controller`를 테스트 할 수 있는데, 프로젝트에서 사용하는 전체 `Bean`을 `ApplicationContext`에 등록하여 사용한다.  
  

이는 테스트 환경을 구성하는 것은 편리하지만 실행 속도가 상대적으로 느리다.  
  

`@WebMvcTest` 애너테이션은 `Controller` 테스트에 필요한 `Bean`만 `ApplicationContext`에 등록하기 때문에 실행 속도는 상대적으로 빠르다.  
  

하지만 `Controller`에서 의존하고 있는 객체가 있다면 해당 객체에 대해서 `Mock` 객체를 사용하여 의존성을 일일이 제거해 주어야 한다.  
  

**결과적으로 `@SpringBootTest` 는 데이터베이스까지 요청 프로세스가 이어지는 통합 테스트에 주로 사용되고, `@WebMvcTest` 는 Controller를 위한 슬라이스 테스트에 주로 사용된다.**

### API 문서 생성을 위한 API 스펙 정보 추가

#### Controller의 postMember() 핸들러 메서드에 대한 API 스펙 정보 추가 예시

```
import com.google.gson.Gson;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.data.jpa.mapping.JpaMetamodelMappingContext;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.restdocs.payload.JsonFieldType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import java.util.List;

import static com.codestates.util.ApiDocumentUtils.getRequestPreProcessor;
import static com.codestates.util.ApiDocumentUtils.getResponsePreProcessor;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.startsWith;
import static org.mockito.BDDMockito.given;
import static org.springframework.restdocs.headers.HeaderDocumentation.headerWithName;
import static org.springframework.restdocs.headers.HeaderDocumentation.responseHeaders;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.restdocs.mockmvc.RestDocumentationRequestBuilders.patch;
import static org.springframework.restdocs.mockmvc.RestDocumentationRequestBuilders.post;
import static org.springframework.restdocs.payload.PayloadDocumentation.*;
import static org.springframework.restdocs.request.RequestDocumentation.parameterWithName;
import static org.springframework.restdocs.request.RequestDocumentation.pathParameters;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(MemberController.class)
@MockBean(JpaMetamodelMappingContext.class)
@AutoConfigureRestDocs
public class MemberControllerRestDocsTest {
    @Autowired
    private MockMvc mockMvc;

    // (1)
    @MockBean
    private MemberService memberService;

    // (2)
    @MockBean
    private MemberMapper mapper;

    @Autowired
    private Gson gson;

    @Test
    public void postMemberTest() throws Exception {
        // (3) given
        MemberDto.Post post = new MemberDto.Post("mason@gmail.com", "Mason", "010-1234-5678");
        String content = gson.toJson(post);

        // (4)
        given(mapper.memberPostToMember(Mockito.any(MemberDto.Post.class))).willReturn(new Member());

        // (5)
        Member mockResultMember = new Member();
        mockResultMember.setMemberId(1L);
        given(memberService.createMember(Mockito.any(Member.class))).willReturn(mockResultMember);

        // (6) when
        ResultActions actions =
                mockMvc.perform(
                        post("/members")
                                .accept(MediaType.APPLICATION_JSON)
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(content)
                );

        // then
        actions
                .andExpect(status().isCreated())
                .andExpect(header().string("Location", is(startsWith("/members/"))))
                .andDo(document(       // (7) 
                        "post-member",     // (7-1)
                        getRequestPreProcessor(),      // (7-2)
                        getResponsePreProcessor(),     // (7-3)
                        requestFields(             // (7-4)
                                List.of(
                                        fieldWithPath("email").type(JsonFieldType.STRING).description("이메일"), // (7-5)
                                        fieldWithPath("name").type(JsonFieldType.STRING).description("이름"),
                                        fieldWithPath("phone").type(JsonFieldType.STRING).description("휴대폰 번호")
                                )
                        ),
                        responseHeaders(        // (7-6)
                                headerWithName(HttpHeaders.LOCATION).description("Location header. 등록된 리소스의 URI")
                        )
                ));
    }
}
```

1.  `Controller` 클래스의 코드를 확인해 보면 `Service` 클래스와 `Mapper` 를 핸들러 메서드 안에서 사용하고 있다.  
      
    즉, 테스트 케이스가 `Controller`의 `postMember()` 핸들러 메서드에 요청을 전송하면 `Mapper` 를 이용해 `MemberDto.Post` 객체와 `Member` 객체 간의 실제 매핑 작업을 진행한다.  
      
    또한 `Service` 객체를 통해 `createMember()` 메서드를 호출 함으로써 실제 비즈니스 로직을 수행하고 데이터 액세스 계층의 코드까지 호출할 것이다.  
      
    우리에게 필요한 핵심 관심사는 `Controller`가 요청을 잘 전달 받고, 응답을 잘 전송하며 요청과 응답이 정상적으로 수행되면 API 문서 스펙 정보를 잘 읽어 들여서 적절한 문서를 잘 생성하느냐 하는 것이다.  
      
    따라서 `Controller`가 `Service` 와 `Mapper`의 메서드를 호출하지 않도록 관계를 단절 시킬 필요가 있다.  
      
    `Controller`가 의존하는 객체와의 관계를 단절하기 위해 (1)과 (2)에서 `Service`와 `Mapper`의 `Mock Bean`을 주입 받는다.  
      
    두 `Mock` 객체는 테스트 케이스에서 가짜 메서드를 호출하는데 사용된다(Stubbing).
2.  (3)은 `postMember()` 핸들러 메서드에 전송하는 request body이다.
3.  (4), (5)에서는 `Controller`의 `postMember()`에서 의존하는 객체의 메서드 호출을 (1)과 (2)에서 주입 받은 `Mock` 객체를 사용해서 Stubbing하고 있다.
4.  (6)은 `MockMvc`의 `perform()` 메서드로 POST 요청을 전송하고 있다.
5.  (7)의 `document(…)` 메서드는 API 스펙 정보를 전달 받아서 실질적인 문서화 작업을 수행하는 `RestDocumentationResultHandler` 클래스에서 가장 핵심 기능을 하는 메서드이다.
6.  (7-1)에서`document()` 메서드의 첫 번째 파라미터는 API 문서 스니핏의 식별자 역할을 하며, (7-1)에서 “`post-member`”로 지정했기 때문에 문서 스니핏은 `post-member` 디렉토리 하위에 생성된다.
7.  (7-2)와 (7-3)은 문서 스니핏을 생성하기 전에 request와 response에 해당하는 문서 영역을 전처리하는 역할을 하는데 아래 코드와 같이 공통화 한 후, 모든 테스트 케이스에서 재사용 할 수 있도록 할 수 있다.

```
import org.springframework.restdocs.operation.preprocess.OperationRequestPreprocessor;
import org.springframework.restdocs.operation.preprocess.OperationResponsePreprocessor;

import static org.springframework.restdocs.operation.preprocess.Preprocessors.*;

public interface ApiDocumentUtils {
   static OperationRequestPreprocessor getRequestPreProcessor() {
       return preprocessRequest(prettyPrint());
    }

    static OperationResponsePreprocessor getResponsePreProcessor() {
        return preprocessResponse(prettyPrint());
    }
}
```

-   `preprocessRequest(prettyPrint())` 는 문서에 표시되는 JSON 포맷의 request body를 예쁘게 표현해 준다.
-   `preprocessResponse(prettyPrint())` 는 문서에 표시되는 JSON 포맷의 response body를 예쁘게 표현해 준다.

8.  (7-4)의 `requestFields(…)`는 문서로 표현될 request body를 의미하며, 파라미터로 전달되는 `List<FieldDescriptor>` 의 원소인 `FieldDescriptor` 객체가 request body에 포함된 데이터를 표현한다.
9.  (7-5)는 request body를 JSON 포맷으로 표현 했을 때, 하나의 프로퍼티를 의미하는 `FieldDescriptor` 이며, `type(JsonFieldType.STRING)`은 JSON 프로퍼티의 값이 문자열 임을 의미한다.
10.  (7-6)의 `responseHeaders(…)`는 문서로 표현될 response header를 의미하며, 파라미터로 전달되는 `HeaderDescriptor` 객체가 response header를 표현합니다.
11.  **정보를 선택적으로 수정하기 위해선 API 스펙 정보에서 필수가 아닌 선택 정보로 설정해야 한다. 이는 (7-5)의 `description(...)` 뒤에 `optional()`을 추가한다.**  
    **반대로 ignored()를 추가하면 API 스펙 정보에서 제외된다.**