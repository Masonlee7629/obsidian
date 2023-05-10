태그: #Spring 
연결문서: [Spring MVC - Spring Rest Docs 1](Spring%20MVC%20-%20Spring%20Rest%20Docs%201.md)

# API 문서화

## API 문서화란?

**API 문서화란 클라이언트가 REST API 백엔드 애플리케이션에 요청을 전송하기 위해서 알아야 되는 요청 정보(요청 URL(또는 URI), request body, query parameter 등)를 문서로 잘 정리하는 것을 의미한다.**

### API 문서화가 필요한 이유

우리가 개발한 REST API 기반의 백엔드 애플리케이션을 클라이언트 쪽에서 사용하려면 API 사용을 위한 어떤 정보가 필요하기 때문이다.

-   API 사용을 위한 어떤 정보가 담겨 있는 문서를 API 문서 또는 API 스펙(사양)이라고 한다.

API 문서는 개발자가 요청 URL(또는 URI) 등의 API 정보를 직접 수기로 작성할 수도 있고, 애플리케이션 빌드를 통해 API 문서를 자동 생성할 수도 있다.

### API 문서 생성의 자동화가 필요한 이유

한번 작성된 API 문서에 기능이 추가되거나 수정되면 API 문서 역시 수정되어야 하는데 사람이 **직접 수기로 작성해야할 경우 비효율적**이며, API 문서에 추가된 기능을 빠트리거나, **클라이언트에게 제공된 API 정보와 수기로 작성한 API 문서의 정보가 다를 수도 있다.**  
  

실제로 클라이언트에게 제공한 API 문서와 백엔드 애플리케이션의 API 스펙 정보가 달라서 프론트엔드 쪽에서 API 문서를 기반으로 백엔드 애플리케이션 쪽에 요청을 전송했을 때, 에러가 발생하는 경우가 빈번하다.  
  

따라서 API 문서 자동화를 통해 작업 시간을 단축하고, 애플리케이션의 완성도를 높여줄 필요가 있다.

### Spring Rest Docs vs Swagger

#### Swagger의 API 문서화 방식

-   애너테이션 기반의 API 문서화 방식이다.
-   애플리케이션 코드에 문서화를 위한 애너테이션들이 포함된다.
-   가독성 및 유지 보수성이 떨어진다.
-   API 문서와 API 코드 간의 정보 불일치 문제가 발생할 수 있다.
-   API 툴로써의 기능을 활용할 수 있다.

```
@ApiOperation(value = "회원 정보 API", tags = {"Member Controller"}) // (1)
@RestController
@RequestMapping("/swagger/members")
@Validated
@Slf4j
public class MemberControllerSwaggerExample {
    private final MemberService memberService;
    private final MemberMapper mapper;

    public MemberControllerSwaggerExample(MemberService memberService, MemberMapper mapper) {
        this.memberService = memberService;
        this.mapper = mapper;
    }

    // (2)
    @ApiOperation(value = "회원 정보 등록", notes = "회원 정보를 등록합니다.")

    // (3)
    @ApiResponses(value = {
            @ApiResponse(code = 201, message = "회원 등록 완료"),
            @ApiResponse(code = 404, message = "Member not found")
    })
    @PostMapping
    public ResponseEntity postMember(@Valid @RequestBody MemberDto.Post memberDto) {
        Member member = mapper.memberPostToMember(memberDto);
        member.setStamp(new Stamp()); // homework solution 추가

        Member createdMember = memberService.createMember(member);

        return new ResponseEntity<>(
                new SingleResponseDto<>(mapper.memberToMemberResponse(createdMember)),
                HttpStatus.CREATED);
    }
    ...
    ...
    // (4)
    @ApiOperation(value = "회원 정보 조회", notes = "회원 식별자(memberId)에 해당하는 회원을 조회합니다.")
    @GetMapping("/{member-id}")
    public ResponseEntity getMember(
            @ApiParam(name = "member-id", value = "회원 식별자", example = "1")  // (5)
            @PathVariable("member-id") @Positive long memberId) {
        Member member = memberService.findMember(memberId);
        return new ResponseEntity<>(
                new SingleResponseDto<>(mapper.memberToMemberResponse(member))
                                    , HttpStatus.OK);
    }
    ...
    ...
}
```

위 코드는 MemberController의 API 정보를 문서화 하기 위해서 Swagger를 적용한 코드의 일부이다.  
  

Swagger를 사용하면 (1) ~ (5)와 같이 API 문서를 만들기 위한 많은 애너테이션들이 애플리케이션 코드에 추가되어야 한다.  
  

애플리케이션 기능을 구현하기 위한 코드에 API 문서를 생성하기 위한 애너테이션이 추가되는 것은 코드의 간결함을 추구하는 개발자라면 무언가 불편한 구조로 보일 가능성이 높다.  
  

Controller 뿐만 아니라 Request Body나 Response Body 같은 DTO 클래스에도 Swagger의 애너테이션을 추가해 주어야 한다.  
  

이런 단점이 있더라도 **Postman처럼 API 요청 툴로써의 기능을 사용할 수 있다는 것이 Swagger의 대표적인 장점이다.**

#### Spring Rest Docs의 API 문서화 방식

-   테스트 코드 기반의 API 문서화 방식이다.
-   애플리케이션 코드에 문서화를 위한 정보들이 포함되지 않는다.
-   테스트 케이스의 실행이 "`passed`"여야 API 문서가 생성된다.
-   테스트 케이스를 반드시 작성해야 된다.
-   API 툴로써의 기능을 제공하지 않는다.

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

    @MockBean
    private MemberService memberService;

    @MockBean
    private MemberMapper mapper;

    @Autowired
    private Gson gson;

    @Test
    public void postMemberTest() throws Exception {
        // given
        MemberDto.Post post = new MemberDto.Post("mason@gmail.com",
                "Mason",
                "010-1234-5678");
        String content = gson.toJson(post);

        // willReturn()이 최소 null은 아니어야 한다.
        given(mapper.memberPostToMember(Mockito.any(MemberDto.Post.class)))
                .willReturn(new Member());

        Member mockResultMember = new Member();
        mockResultMember.setMemberId(1L);
        given(memberService.createMember(Mockito.any(Member.class))).willReturn(mockResultMember);

        // when
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
                .andDo(document("post-member",    // =========== (1) API 문서화 관련 코드 시작 ========
                        getRequestPreProcessor(),
                        getResponsePreProcessor(),
                        requestFields(
                                List.of(
                                        fieldWithPath("email").type(JsonFieldType.STRING).description("이메일"),
                                        fieldWithPath("name").type(JsonFieldType.STRING).description("이름"),
                                        fieldWithPath("phone").type(JsonFieldType.STRING).description("휴대폰 번호")
                                )
                        ),
                        responseHeaders(
                                headerWithName(HttpHeaders.LOCATION).description("Location header. 등록된 리소스의 URI")
                        )
                ));   // =========== (2) API 문서화 관련 코드 끝========
    }
}
```

Spring Rest Docs와 Swagger의 가장 큰 차이점은 Spring Rest Docs의 경우 애플리케이션 기능 구현과 관련된 코드에는 API 문서 생성을 위한 애너테이션 같은 어떠한 정보도 추가되지 않는다는 것이다.  
  

Spring Rest Docs를 사용한 API 문서화의 대표적인 장점은 테스트 케이스에서 전송하는 API 문서 정보와 Controller에서 구현한 Request Body, Response Body, Query Parmeter 등의 정보가 하나라도 일치하지 않으면 테스트 케이스의 실행 결과가 “`failed`” 되면서 API 문서가 정상적으로 생성이 되지 않는다는 것이다.  
  

즉, 테스트 케이스의 실행 결과를 "`passed`"로 만들지 않으면 API 문서 생성이 완료되지 않는다.  
  

따라서 애플리케이션에 정의되어 있는 API 스펙 정보와 API 문서 정보의 불일치로 인해 발생하는 문제를 방지할 수 있다.  
  

Spring Rest Docs의 대표적인 단점이라면 테스트 케이스를 일일이 작성해야 되고, Controller에 대한 모든 테스트 케이스를 “`passed`”로 만들어야 한다는 점이다.