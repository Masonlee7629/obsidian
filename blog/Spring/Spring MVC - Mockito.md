태그: #Spring 
연결문서: [Spring MVC - TDD](Spring%20MVC%20-%20TDD.md)

# 테스팅(Testing)

## Mockito

### Mock이란?

**테스트 세계에서의 Mock은 바로 가짜 객체를 의미한다.**  
  

그리고 단위 테스트나 슬라이스 테스트 등에 Mock 객체를 사용하는 것을 **Mocking**이라고 한다.

### Mockito란?

Mock 객체로 Mocking을 할 수 있게 해주는 여러 오픈 소스 라이브러리가 있지만 그 중에서 가장 많이 사용되고, Spring Framework 자체적으로도 지원하고 있는 Mocking 라이브러리가 **Mockito**이다.  
  

Mockito의 Mocking 기능을 이용해서 테스트하고자 하는 대상에서 다른 영역(다른 계층 또는 외부 통신이 필요한 서비스 등)을 단절시켜 오로지 테스트 대상에만 집중할 수 있다.

### 슬라이스 테스트에 Mockito 적용

#### Controller의 `postMember()` 테스트에 Mockito 적용

```
import com.google.gson.Gson;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.transaction.annotation.Transactional;

import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.startsWith;
import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class MemberControllerMockTest {
    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private Gson gson;

    // (1)
    @MockBean
    private MemberService memberService;

    // (2)
    @Autowired
    private MemberMapper mapper;

    @Test
    void postMemberTest() throws Exception {
        // given
        MemberDto.Post post = new MemberDto.Post("mason@gmail.com",
                                                        "Mason",
                                                    "010-1234-5678");

        Member member = mapper.memberPostToMember(post);  // (3)
        member.setMemberId(1L);     // (4)

        // (5)
        given(memberService.createMember(Mockito.any(Member.class)))
                .willReturn(member);

        String content = gson.toJson(post);

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
                .andExpect(header().string("Location", is(startsWith("/members/"))));
    }
}
```

1.  `@MockBean` 애너테이션은 Application Context에 등록되어 있는 Bean에 대한 Mockito Mock 객체를 생성하고 주입해주는 역할을 한다  
      
    (1)과 같이 `@MockBean` 애너테이션을 필드에 추가하면 해당 필드의 Bean에 대한 Mock 객체를 생성한 후, 필드에 주입(DI)한다.  
      
    (1)에서는 `MemberService` 빈에 대한 `Mock` 객체를 생성해서 `memberService` 필드에 주입한다.
2.  (2)에서 `MemberMapper`를 DI 받는 이유는 `MockMemberService`의 `createMember()`에서 리턴하는 `Member` 객체를 생성하기 위함이다.
3.  (3)에서 `MemberMapper`를 이용해 `post(MemberDto.post 타입)` 변수를 `Member` 객체로 변환한다.
4.  실제 `createMember()`의 리턴 값(`Member` 객체)에는 `memberId`가 포함이 되는데 이 `memberId`는 response의 Location header에 포함이 되어야 하므로 (4)와 같이 `MockMemberService`의 `createMember()`에서도 `memberId`를 리턴해 줄 수 있도록 `memberId`를 추가해준다.
5.  (5)는 Mockito에서 지원하는 **Stubbing** 메서드이다.
    -   Stubbing은 테스트를 위해서 `Mock` 객체가 항상 일정한 동작을 하도록 지정하는 것이다.
    -   `given(memberService.createMember(Mockito.any(Member.class)))`
        -   `given()`은 `Mock` 객체가 특정 값을 리턴하는 동작을 지정하는데 사용하며, Mockito에서 지원하는 `when()`과 동일한 기능을 한다.
        -   위 코드에선 `Mock` 객체인 `memberService` 객체로 `createMember()` 메서드를 호출하도록 정의하고 있다.
        -   `createMember()`의 파라미터인 `Mockito.any(Member.class)` 는 Mockito에서 지원하는 변수 타입 중 하나이다.
            -   `MockMemberService`가 아닌 실제 `MemberService` 클래스에서 `createMember()`의 파라미터 타입은 `Member` 타입이므로 `Mockito.any()`에 `Member.class`를 타입으로 지정했다.
    -   `.willReturn(member)`
        -   `MockMemberService`의 `createMember()` 메서드가 리턴 할 Stub 데이터이다.

테스트를 실행하면 **MockMemberService**의 `createMember()` 메서드가 호출되므로, 데이터 액세스 계층의 로직은 실행되지 않는다.  
  

`MockMemberService` 클래스는 테스트하고자 하는 Controller의 테스트에 집중할 수 있도록 다른 계층과의 연동을 끊어주는 역할을 하는 것이다.

#### Service의 `createMember()` 테스트에 Mockito 적용

```
@Transactional
@Service
public class MemberService {
    private final MemberRepository memberRepository;
    private final ApplicationEventPublisher publisher;

    public MemberService(MemberRepository memberRepository,
                         ApplicationEventPublisher publisher) {
        this.memberRepository = memberRepository;
        this.publisher = publisher;

    }

    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());     // (1)
        Member savedMember = memberRepository.save(member);

        publisher.publishEvent(new MemberRegistrationApplicationEvent(this, savedMember));
        return savedMember;
    }

    ...
    ...

    private void verifyExistsEmail(String email) {
        Optional<Member> member = memberRepository.findByEmail(email);  // (2)

        // (3)
        if (member.isPresent())
            throw new BusinessLogicException(ExceptionCode.MEMBER_EXISTS);
    }
}
```

위 코드는 `MemberService` 클래스의 코드로, 테스트 하려는 부분은 `createMember()` 메서드의 `verifyExistsEmeil()` 메서드가 정상적인 동작을 수행하는지 테스트하는 것이다.  
  

`verifyExistsEmail()` 메서드의 내부를 보면 (2)와 같이 `verifyExistsEmail()` 메서드의 파라미터로 전달 받은 email을 조건으로 한 회원 정보가 있는지 `memberRepository.findByEmail(email)`을 통해 DB에서 조회하고 있다.  
  

그러나 `verifyExistsEmail()` 메서드가 DB에서 `Member` 객체를 잘 조회하는지 여부를 테스트 하려는게 아니라, 어디서 조회해 왔든 상관없이 조회된 `Member` 객체가 `null`이면 `BusinessLogicException` 을 잘 던지는지 여부만 테스트하면 된다.  
  

따라서 DB에서 회원 정보를 조회하는 (2)의 `memberRepository.findByEmail(email)` 은 Mocking의 대상이 된다.  
  

```
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.BDDMockito.given;

// (1)
@ExtendWith(MockitoExtension.class)
public class MemberServiceMockTest {
    @Mock   // (2)
    private MemberRepository memberRepository;

    @InjectMocks    // (3)
    private MemberService memberService;

    @Test
    public void createMemberTest() {
        // given
        Member member = new Member("hgd@gmail.com", "홍길동", "010-1111-1111");

        // (4)
        given(memberRepository.findByEmail(Mockito.anyString()))
                                                    .willReturn(Optional.of(member)); // (5)

                // when / then (6)
        assertThrows(BusinessLogicException.class, () -> memberService.createMember(member));
    }
}
```

위 코드는 `MemberService`의 `createMember()`를 테스트하는 `MemberServiceMockTest` 클래스이다.

1.  Spring을 사용하지 않고, JUnit에서 Mockito의 기능을 사용하기 위해서는 (1)과 같이 `@ExtendWith(MockitoExtension.class)`를 추가해야 한다.
2.  (2)와 같이 `@Mock` 애너테이션을 추가하면 해당 필드의 객체를 `Mock` 객체로 생성한다.
3.  (3)과 같이 `@InjectMocks` 애너테이션을 추가한 필드에 (2)에서 생성한 `Mock` 객체를 주입해준다. (3)의 **`memberService` 객체는 주입 받은 `memberRepository Mock` 객체를 포함하고 있다.**
4.  (4)에서는 (2)에서 생성한 `memberRepository Mock` 객체로 Stubbing을 한다.
    -   `memberRepository.findByEmail(Mockito.anyString())`의 리턴 값으로 (5)와 같이 `Optional.of(member)`를 지정했기 때문에 테스트 케이스를 실행하면 결과는 “`passed`”이다.
    -   `Optional.of(member)` 의 `member` 객체에 포함된 이메일 주소가 `memberService.createMember(member)` 에서 파라미터로 전달한 `member` 객체에 포함된 이메일 주소와 동일하기 때문에 검증 결과가 “`passed`”이다.