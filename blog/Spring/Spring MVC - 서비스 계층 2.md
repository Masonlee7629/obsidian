태그: #Spring
연결문서: [Spring MVC - 예외 처리 1](Spring%20MVC%20-%20예외%20처리%201.md)

# Service

## 매퍼(Mapper)를 이용한 DTO클래스와 엔티티(Entity) 클래스 매핑

### 매퍼(Mapper)

매퍼는 DTO 클래스와 엔티티 클래스를 서로 변환해주는 클래스이다.

-   매퍼를 이용해 계층 간의 역할 분리
    -   DTO 클래스는 API 계층에서만 데이터를 처리하는 역할을 하고, 엔티티 클래스는 서비스 계층에서만 데이터를 처리하는 역할을 함으로써 계층 간의 분리가 이루어짐

#### 매퍼를 이용한 매핑 순서

1.  Mapper 클래스 구현
2.  Controller의 핸들러 메서드에 매퍼 클래스 적용

### Mapper 클래스 구현

Mapper 클래스의 클래스 레벨에 `@Component` 애너테이션을 추가한다.  
  

이는 Mapper 클래스를 Spring Bean으로 등록하고, 등록된 Bean은 Controller에서 사용한다.

```
@Component
public class MemberMapper {

  // MemberPostDto 를 Member 엔티티로 변환
  public Member memberPostDtoToMember(MemberPostDto memberPostDto) {
    return new Member(
      0L,
      memberPostDto.getEmail(),
      memberPostDto.getName(),
      memberPostDto.getPhone()
    );
  }

  // MemberPatchDto 를 Member 엔티티로 변환
  public Member memberPatchDtoToMember(MemberPatchDto memberPatchDto) {
    return new Member(
      memberPatchDto.getMemberId(),
      // 일반적으로 이메일은 고유한 데이터로 변경되지 않아
      // null로 Patch가 불가능하도록 표기한 것
      null,
      memberPatchDto.getName(),
      memberPatchDto.getPhone()
    );
  }

  // Member 엔티티를 MemberResponseDto 로 변환
  // MemberResponseDto 는 ResponseBody의 역할을 해주는 DTO 클래스
  public MemberResponseDto memberToMemberResponseDto(Member member) {
    return new MemberResponseDto(
      member.getMemberId(),
      member.getEmail(),
      member.getName(),
      member.getPhone()
    );
  }
}
```

### Controller의 핸들러 메서드에 매퍼(Mapper) 클래스 적용

DI로 주입하여 Spring Bean에 등록된 Mapper 객체를 Controller에서 사용한다.

-   Service 계층과 연동이 이루어지는 곳에서 DTO 클래스를 엔티티 클래스로 변환하여 전달
-   클라이언트에게 전달할 ResponseBody는 API 계층의 영역으로 DTO 클래스로 전달
    -   DTO 클래스가 엔티티 클래스로 변환되었다면 다시 DTO 클래스로 재변환

```
@RestController
@RequestMapping("/v1/members/")
@Validated
public class MemberController {
  private final MemberService memberService;
  private final MemberMapper mapper;

  public Controller(MemberService memberService, MemberMapper mapper) {
    this.memberService = memberService;
    this.mapper = mapper;
  }

  @PostMapping
  public ResponseEntity postMember(@Valid @RequestBody MemberPostDto memberDto) {
    // Mapper를 이용하여 MemberPostDto를 Member로 변환
    Member member = mapper.memberPostDtoToMember(memberDto);

    Member response = memberService.createMember(member);

    // Mapper를 이용하여 Member를 MemberResponseDto로 변환
    return new ResponseEntity<>(mapper.memberToMemberResponseDto(response), HttpStatus.CREATED);
  }

  @PatchMapping("/{member-id}")
  public ResponseEntity patchMember(@PathVariable("member-id") @Positive long memberId,
                                    @Valid @RequestBody MemberPatchDto memberPatchDto) {
    memberPatchDto.setMemberId(memberId);

    // Mapper를 이용하여 MemberPatchDto를 Member로 변환
    Member response = 
      memberService.updateMember(mapper.memberPatchDtoToMember(memberPatchDto));

    // Mapper를 이용하여 Member를 MemberResponseDto로 변환
    return new ResponseEntity<>(mapper.memberToMemberResponseDto(response), HttpStatus.OK);
  }

  @GetMapping("/{member-id}")
  public ResponseEntity getMember(@PathVariable("member-id") @Positive long memberId) {
    Member response = memberService.findMember(memberId);

    // Mapper를 이용하여 Member를 MemberResponseDto로 변환
    return new ResponseEntity<>(mapper.memberToMemberResponseDto(response), HttpStatus.OK);
  }

  @GetMapping
  public ResponseEntity getMembers() {
    List<Member> members = memberService.findMembers();

    // // Mapper를 이용하여 List<Member>를 MemberResponseDto로 변환
    List<MemberResponseDto> response = members.stream()
                        .map(member -> mapper.memberToMemberResponseDto(member))
                        .collect(Collectors.toList());

    return new ResponseEntity<>(response, HttpStatus.OK);
  }

  @DeleteMapping("/{member-id}")
  public ResponseEntity deleteMember(@PathVariable("member-id") @Positive long memberId) {
    System.out.println("# delete member");
    memberService.deleteMember(memberId);

    return new ResponseEntity(HttpStatus.NO_CONTENT);
  }
}
```

### MapStruct를 이용한 Mapper 자동 생성

MapStruct는 Mapper 구현 클래스를 자동으로 생성해주는 코드 자동 생성기이다.  
  

도메인 업무 기능이 증가함에 따라 매퍼 클래스를 생성하는 것은 비효율적이기 때문에 MapStruct를 이용해 자동으로 구현함으로 생산성을 향상시켜줄 수 있다.

#### MapStruct 의존 라이브러리 설정

build.gradle 파일에 의존 라이브러리 추가 및 Gradle 프로젝트 Reload해야 한다.

```
dependencies {
    ...
    ...
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
}
```

#### MapStruct 기반의 Mapper 인터페이스 정의

`@Mapper`를 추가하여 해당 인터페이스가 MapStruct의 매퍼 인터페이스임을 정의해야 한다.

-   애트리뷰트로 componentModel = “spring” 을 지정하면 Spring Bean으로 등록
-   해당 인터페이스를 통해 MapStruct가 Mapper 구현 클래스를 자동 생성

```
@Mapper(componentModel = "spring")
public interface MemberMapper {
  Member memberPostDtoToMember(MemberPostDto memberPostDto);
  Member memberPatchDtoToMember(MemberPatchDto memberPatchDto);
  MemberResponseDto memberToMemberResponseDto(Member member);
}
```

#### 자동 생성된 Mapper 인터페이스 구현 클래스 확인

Mapper 인터페이스의 구현 클래스는 Gradle의 build task를 실행하면 자동으로 생성된다.

-   Project 탭 > 프로젝트명 > build 디렉토리 내 Mapper 인터페이스가 위치한 패키지 내에 생성

```
@Component
public class MemberMapperImpl implements MemberMapper {
  public MemberMapperImpl() {
  }

  public Member memberPostDtoToMember(MemberPostDto memberPostDto) {
    if (memberPostDto == null) {
      return null;
    } else {
      Member member = new Member();
      member.setEmail(memberPostDto.getEmail());
      member.setName(memberPostDto.getName());
      member.setPhone(memberPostDto.getPhone());
      return member;
    }
  }

  public Member memberPatchDtoToMember(MemberPatchDto memberPatchDto) {
    if (memberPatchDto == null) {
      return null;
    } else {
      Member member = new Member();
      member.setMemberId(memberPatchDto.getMemberId());
      member.setName(memberPatchDto.getName());
      member.setPhone(memberPatchDto.getPhone());
      return member;
    }
  }

  public MemberResponseDto memberToMemberResponseDto(Member member) {
    if (member == null) {
      return null;
    } else {
      long memberId = 0L;
      String email = null;
      String name = null;
      String phone = null;
      memberId = member.getMemberId();
      email = member.getEmail();
      name = member.getName();
      phone = member.getPhone();
      MemberResponseDto memberResponseDto = 
                    new MemberResponseDto(memberId, email, name, phone);
      return memberResponseDto;
    }
  }
}
```

---

### Controller의 핸들러 메서드에서 MapStruct 적용

MapStruct 인터페이스의 위치만 import문으로 전달한다.

```
//import com.codestates.member.mapper.MemberMapper;
import com.codestates.member.mapstruct.mapper.MemberMapper; // 패키지 변경

@RestController
@RequestMapping("/v1/members/")
@Validated
public class MemberController {
  ...
  ... // 변경 사항 없음으로 생략
}
```

### MapStruct vs ModelMapper

Java에서 Object를 Mapping하는 라이브러리는 생각보다 많이 존재한다. 그 중에서 가장 많이 사용되는 Mapping 라이브러리에는 **MapStruct와 ModelMapper**가 있다.  
  

ModelMapper가 여전히 많이 사용되고 있지만 ModelMapper는 Runtime 시 Java의 리플렉션 API를 이용해서 매핑을 진행하기 때문에 컴파일 타임에 이미 Mapper가 모두 생성되는 MapStruct보다 성능면에서 월등히 떨어진다.  
  

따라서 ModelMapper의 대안으로 MapStruct가 많이 사용되고 있는 추세이다.

### DTO 클래스와 엔티티 클래스의 역할 분리가 필요한 이유

#### 계층별 관심사의 분리

-   DTO 클래스
    -   API 계층에서 Request Body를 전달 받고, Response Body를 전송하는 것이 목적
-   Entity 클래스
    -   Service 계층에서 Data Access 계층과 연동하여 비즈니스 로직의 결과로 생성된 데이터를 다루는 것이 목적
-   객체 지향 프로그래밍
    -   **하나의 클래스나 메서드 내에서 여러 기능을 구현하는 것은 객체 지향 코드 관점에서 리팩토리 대상**

#### 코드 구성의 단순화

DTO 클래스에서 사용하는 유효성 검사 애너테이션이 Entity 클래스에서 사용되면 JPA에서 사용하는 애너테이션과 뒤섞인 상태로 유지보수하기 어려운 코드가 된다.  
  

따라서 역할 분리를 통해 코드를 단순화할 필요가 있다.

-   Entity 클래스
    -   데이터 액세스 기술인 JPA에서 사용하는 애너테이션이 적용
-   DTO 클래스
    -   유효성 검사 애너테이션을 사용

#### REST API 스펙의 독립성 확보

DTO 클래스를 사용하여 클라이언트에게 원하는 정보만 선별하여 제공 가능하다.

-   Data Access 계층에서 전달 받은 데이터로 이루어진 Entity 클래스 그대로 전달하면 원치 않는 데이터까지 클라이언트에게 전송될 가능성이 존재
    -   Ex) 회원의 로그인 패스워드