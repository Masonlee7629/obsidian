태그: #Spring
연결문서: [Spring MVC - 서비스 계층 2](Spring%20MVC%20-%20서비스%20계층%202.md)

# Service

서비스 계층은 웹 애플리케이션의 비즈니스 요구 사항을 처리하는 핵심 계층이다.  
  

애플리케이션의 비즈니스 요구 사항이 얼마나 복잡하냐에 따라서 비즈니스 로직의 복잡도 역시 달라질 수 있다.  
  

Spring은 개발자가 핵심 비즈니스 로직 개발에 집중할 수 있도록 비즈니스 로직 이외의 작업들을 대신해준다.

## DI를 통한 서비스 계층과 API 계층 연동

API 계층과 서비스 계층을 연동한다는 의미는 API 계층에서 구현한 Controller 클래스가 서비스 계층의 Service 클래스와 메서드 호출을 통해 상호 작용한다는 것을 의미한다.  
  

> **Service의 의미**  
> 애플리케이션에서 Service의 의미는 도메인 업무 영역을 구현하는 비즈니스 로직과 관련이 있다.
> 
> 애플리케이션의 비즈니스 로직을 처리하기 위한 서비스 계층은 대부분 도메인 모델을 포함하고 있다.
> 
> 도메인 모델을 다시 빈약한 **도메인 모델(anemic domain model)**과 **풍부한 도메인 모델(rich domain model)**로 구분할 수 있는데, 이러한 **도메인 모델은 DDD(도메인 주도 설계, Domain Driven Design)와 관련히 깊다.**

### 비즈니스 로직을 처리하는 Service 클래스 작성

API 계층에서 클라이언트의 요구 사항을 만족하는 Controller 클래스를 구현해 두었다면 이 Controller를 기반으로 해당 Controller와 연동하는 Service 클래스의 큰 틀을 만들 수 있다.

-   Service 클래스
    -   도메인 업무 영역을 구현하는 비즈니스 로직을 처리하는 클래스
    -   Controller가 전달 받은 요청을 실제로 처리하는 역할
    -   Controller의 핸들러 메서드와 1 대 1로 매칭
-   도메인 엔티티(Entity) 클래스
    -   API 계층에서 전달 받은 요청 데이터를 기반으로, 서비스 계층에서 비즈니스 로직을 처리하기 위해 필요한 데이터를 전달 받고 비즈니스 로직을 처리한 후 결과 값을 다시 API 계층으로 리턴해주는 역할
    -   서비스 계층에서 데이터 액세스 계층과 연동하며 비즈니스 로직을 처리하기 위해 필요한 데이터를 담는 역할

#### 구현되지 않은 Service 클래스 예시

```
import java.util.List;

public class MemberService {
    public Member createMember(Member member) {
        return null;
    }

    public Member updateMember(Member member) {
        return null;
    }

    public Member findMember(long memberId) {
        return null;
    }

    public List<Member> findMembers() {
        return null;
    }

    public void deleteMember(long memberId) {

    }
}
```

#### 도메인 엔티티 클래스 구현 예시

API 계층의 DTO 클래스에서 사용한 멤버 변수가 모두 포함된다.

-   `@Getter`, `@Setter`
    -   Lombok 라이브러리에서 제공하는 애너테이션
    -   각 멤버 변수에 해당하는 getter/setter 메서드를 자동으로 생성해주는 유틸리티성 라이브러리
-   `@AllArgsConstructor`
    -   엔티티 클래스에 추가된 모든 멤버 변수를 파라미터로 갖는 생성자를 자동으로 생성하는 애너테이션
-   `@NoArgsConstructor`
    -   파라미터가 없는 기본 생성자를 자동으로 생성하는 애너테이션

```
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Member {
  private long memberId;
  private String email;
  private String korName;
  private String phoneNumber;
}
```

#### 구현된 Service 클래스 예시

실제 구현 시 데이터 액세스 계층과 연동하여 DB에 데이터를 저장 및 조회 등이 가능해야 한다.

```
public class MemberService {
  public Member createMember(Member member) {
    Member createdMember = member;
    return createdMember;
  }

  public Member updateMember(Member member) {
    Member updatedMember = member;
    return updatedMember;
  }

  public Member findMember(long memberId) {
    Member member = new Member(memberId, "mason@gmail.com", "메이슨", "010-1111-1111");
    return member;
  }

  public List<Member> findMembers() {
    List<Member> members = List.of(
      new Member(1, "mason@gmail.com", "메이슨", "010-1111-1111"),
      new Member(2, "serena@gmail.com", "세레나", "010-2222-2222")
    );
    return members;
  }

  public void deleteMember(long memberId) {
  }
}
```

### DI를 적용한 Service 계층과 API 계층 연동

DI를 적용하면 클래스 간의 결합을 느슨한 결합(loose Coupling)으로 쉽게 만들 수 있다.  
  

스프링에서 지원하는 DI 기능을 사용하지 않으면 Controller와 Service가 강하게 결합되어 있는 상태가 된다.

-   **느슨한 결합(Loose Coupling)**
    -   클래스의 자료 구조, 메서드를 추상화할 수 있는 인터페이스 클래스를 사용해 의존성을 최소화
-   **강한 결합(Tight Coupling)**
    -   클래스와 객체가 서로 의존적인 상태로 객체가 변경될 시 클래스가 전체적으로 수정되어야 할 수 있음

```
// Controller 클래스에 DI 적용
@RestController
@RequestMapping("/v1/members")
@Validated
public class MemberController {
  private final MemberService memberService;

    // DI를 적용하여 MemberController 생성자 파라미터로 MemberService의 객체를 주입
  public MemberController(MemberService memberService) {
    this.memberService = memberService;
  }

    ...
    ... // MemberContoller 핸들러 메서드 생략
}
```

```
// Service 클래스에 @Service 애너테이션 추가
// DI를 통해 객체를 주입 받기 위해서는
// 주입을 받는 클래스와 주입 대상 클래스 모두 Spring Bean으로 등록되어야 함
@Service
public class MemberService {
  // 생성자가 하나 이상일 경우
  // DI를 적용하기 위한 생성자에 반드시 @Autowired 애너테이션 적용
  ...
    ... // MemberService 핸들러 메서드 생략
}
```