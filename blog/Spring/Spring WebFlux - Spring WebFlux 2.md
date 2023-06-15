태그: #Spring 
연결문서: [Cloud - 운영 환경 구성 1](Cloud%20-%20운영%20환경%20구성%201.md)

# Spring WebFlux

## 리액티브한 샘플 애플리케이션 구현

### 프로젝트 설정
- Spring WebFlux를 프로젝트에서 사용하기 위한 설정은 Spring MVC 학습에서 설정했던 방식과 크게 다르지 않지만 약간의 차이점이 존재함
<br>

#### build.gradle 설정

```java
...

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-webflux'     // (1)
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	implementation 'org.springframework.boot:spring-boot-starter-data-r2dbc'   // (2)
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'io.projectreactor:reactor-test'
	implementation 'org.mapstruct:mapstruct:1.5.1.Final'
	annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.1.Final'
	runtimeOnly 'io.r2dbc:r2dbc-h2'  // (3)
}

...
```

- Spring WebFlux를 프로젝트에서 사용하기 위해서는 build.gradle에 의존성을 추가하면 됨
- (1)에서는 우리가 기존에 사용하던 ‘spring-boot-starter-web’에서 ‘spring-boot-starter-webflux’로 변경함
- 리액티브 스택에서는 JPA 대신에 R2DBC를 사용하며, 따라서 (2)에서 기존의 'spring-boot-starter-data-jpa’에서 ‘spring-boot-starter-data-r2dbc'로 변경됨
- 인메모리 DB인 H2 역시 Non-Blocking을 지원하는 드라이버를 사용할 수 있도록 (3)과 같이 ‘com.h2database:h2’에서 ‘io.r2dbc:r2dbc-h2’로 변경됨

<br>

#### application.yml 설정

```java
spring:
  sql:
    init:
      schema-locations: classpath*:db/h2/schema.sql   // (1)
      data-locations: classpath*:db/h2/data.sql       // (2)
logging:
  level:
    org:
      springframework:
        r2dbc: DEBUG          // (3)
```

- Spring Data 패밀리 프로젝트에서는 (1), (2)와 같이 직접 테이블 스키마를 정의하고, 샘플 데이터를 정의해서 애플리케이션 실행 시, SQL 스크립트를 실행할 수 있음
    - Spring Data JDBC와 마찬가지로 Spring Data R2DBC 역시 Spring Data JPA의 Auto DDL 같은 기능을 제공하지 않기 때문에 (1)과 같이 직접 SQL 스크립트 설정을 추가해야 함
    - 회원 목록 조회를 위해 애플리케이션 실행 시, 20 row의 샘플 데이터가 MEMBER 테이블에 저장되도록 data.sql에 스크립트가 추가되어 있음
- Spring Data R2DBC 기술을 이용해 데이터베이스와 상호작용하는 동작을 로그로 출력하고자 r2dbc 로그 레벨을 ‘DEBUG’로 설정함

<br>

#### DB Schema 설정

```SQL
CREATE TABLE IF NOT EXISTS MEMBER (
    MEMBER_ID bigint NOT NULL AUTO_INCREMENT,
    EMAIL varchar(100) NOT NULL UNIQUE,
    NAME varchar(100) NOT NULL,
    PHONE varchar(100) NOT NULL,
    MEMBER_STATUS varchar(20) NOT NULL,
    CREATED_AT datetime NOT NULL,
    LAST_MODIFIED_AT datetime NOT NULL,
    PRIMARY KEY (MEMBER_ID)
);

CREATE TABLE IF NOT EXISTS STAMP (
    STAMP_ID bigint NOT NULL AUTO_INCREMENT,
    STAMP_COUNT bigint NOT NULL,
    MEMBER_ID bigint NOT NULL,
    CREATED_AT datetime NOT NULL,
    LAST_MODIFIED_AT datetime NOT NULL,
    PRIMARY KEY (STAMP_ID),
    FOREIGN KEY (MEMBER_ID) REFERENCES MEMBER(MEMBER_ID)
);

...
```

- ‘src/main/resources/db/h2’에 있는 scheme.sql 파일의 테이블 생성 스크립트
- schema.sql 파일은 테이블 schema 설정에 사용됨

<br>

### 애플리케이션 공통 설정

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.r2dbc.config.EnableR2dbcAuditing;
import org.springframework.data.r2dbc.repository.config.EnableR2dbcRepositories;

@EnableR2dbcRepositories    // (1)
@EnableR2dbcAuditing        // (2)
@SpringBootApplication
public class WebFluxCoffeeOrderSampleApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebFluxCoffeeOrderSampleApplication.class, args);
	}

}
```

- R2DBC의 Reposiroty를 사용하기 위해서는 main() 메서드가 포함된 애플리케이션 클래스(WebFluxCoffeeOrderSampleApplication)에 (1)과 같이 @EnableR2dbcRepositories 애너테이션을 추가해 주어야 함
- 데이터베이스에 엔티티가 저장 및 수정될 때, 생성 날짜와 수정 날짜를 자동으로 저장할 수 있도록 Auditing 기능을 사용하기 위해 (2)와 같이 @EnableR2dbcAuditing 애너테이션을 추가함

<br>

### Controller 구현

#### Spring WebFlux가 적용된 MemberController

```java
import com.member.dto.MemberDto;
import com.member.mapper.MemberMapper;
import com.member.service.MemberService;
import com.utils.UriCreator;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Mono;

import javax.validation.Valid;
import javax.validation.constraints.Positive;
import java.net.URI;
import java.util.List;

@Validated
@RestController
@RequestMapping("/members")
public class MemberController {
    private final MemberService memberService;
    private final MemberMapper mapper;

    public MemberController(MemberService memberService, MemberMapper mapper) {
        this.memberService = memberService;
        this.mapper = mapper;
    }

    @PostMapping
    public Mono<ResponseEntity> postMember(@Valid @RequestBody Mono<MemberDto.Post> requestBody) {  // (1)
        return requestBody
                .flatMap(post -> memberService.createMember(mapper.memberPostToMember(post)))  // (2)
                .map(createdMember -> {
                    URI location = UriCreator.createUri(MEMBER_DEFAULT_URL, createdMember.getMemberId());
                    return ResponseEntity.created(location).build();
                });
    }

    @PatchMapping("/{member-id}")
    public ResponseEntity patchMember(@PathVariable("member-id") @Positive long memberId,
                                      @Valid @RequestBody Mono<MemberDto.Patch> requestBody) {    // (3)
        Mono<MemberDto.Response> response =
                requestBody
                        .flatMap(patch -> {            // (4)
                            patch.setMemberId(memberId);         
                            return memberService.updateMember(mapper.memberPatchToMember(patch));
                        })
                        .map(member -> mapper.memberToMemberResponse(member));

        return new ResponseEntity<>(response, HttpStatus.OK);
    }

    @GetMapping("/{member-id}")
    public ResponseEntity getMember(@PathVariable("member-id") @Positive long memberId) {
        Mono<MemberDto.Response> response =
                memberService.findMember(memberId)   // (5)
                        .map(member -> mapper.memberToMemberResponse(member));
        return new ResponseEntity(response, HttpStatus.OK);
    }

    @GetMapping
    public ResponseEntity getMembers(@RequestParam("page") @Positive int page,
                                     @RequestParam("size") @Positive int size) {
        Mono<List<MemberDto.Response>> response =
                memberService.findMembers(PageRequest.of(page - 1, size, Sort.by("memberId").descending()))  // (6)
                        .map(pageMember -> mapper.membersToMemberResponses(pageMember.getContent()));

        return new ResponseEntity<>(response, HttpStatus.OK);
    }

    @DeleteMapping("/{member-id}")
    public ResponseEntity deleteMember(@PathVariable("member-id") long memberId) {
        Mono<Void> result = memberService.deleteMember(memberId);    // (7)
        return new ResponseEntity(result, HttpStatus.NO_CONTENT);
    }
}
```

- Mono라는 Reactor의 타입이 코드 곳곳에 추가되었음
    - Mono라는 Reactor의 타입이 추가되는 순간, Reactor의 동작 원리와 Reactor에서 지원하는 다양한 Operator의 사용법을 알아야 함
- (1)을 보면 postMember() 핸들러 메서드의 파라미터인 MemberDto.Post 객체가 Mono로 래핑 되어 있음
    - Spring WebFlux에서는 request body로 단순히 MemberDto.Post 객체를 전달받을 수도 있지만 (1)과 같이 Mono&#60;MemberDto.Post&#62;와 같이 전달받을 수도 있음
    - 전달받은 객체에 Blocking 요소가 포함되지 않도록 request body를 전달받는 순간부터 Non-Blocking으로 동작하도록 Operator 체인을 바로 연결해서 다음 처리를 시작할 수 있음
- (1)에서 전달받은 request body가 Mono&#60;MemberDto.Post&#62;이기 때문에 (2)와 같이 바로 다음 처리를 Non-Blocking으로 처리할 수 있도록 Operator 체인을 연결할 수 있음
    - (2)에서는 MemberService 클래스의 createMember() 메서드를 호출해서 회원 정보를 저장하는 처리를 바로 이어서 수행하고 있음
    - 모든 처리가 Mono Sequence 내에서 처리되기 때문에 Non-Blocking으로 처리됨
- (3)에서도 역시 Mono&#60;MemberDto.Patch&#62;로 request body를 전달 받음
- (4)에서 Mono Sequence 내부에서 MemberService 클래스의 updateMember() 메서드를 호출해서 회원 정보를 저장하는 처리를 바로 이어서 수행함
- (5)에서는 MemberService 클래스의 findeMember() 메서드를 호출해서 회원 정보를 조회함
    - 한 가지 기억해야 될 내용은 MemebrService 클래스의 메서드를 호출해서 Mono Sequecne를 추가적으로 연결할 수 있다는 의미는 MemberService 클래스의 메서드 역시 리턴 타입이 Mono라는 것
    - 기존에 알고 있던 MemberService 클래스의 메서드는 대부분 Member 객체를 리턴했지만 Spring WebFlux 기반의 MemberService 클래스는 Mono와 같이 Mono로 래핑 한 값을 리턴함
- (6)에서는 페이지네이션을 위해 PageRequest 객체를 MemberController 쪽에서 직접 만들어서 MemberService 쪽으로 전달하고 있는 것이 Spring MVC 방식과 다름
- (7)에서는 회원 정보를 삭제하기 위해 MemberService의 deleteMember() 메서드를 호출함
    - 리턴되는 데이터가 없는 경우, Spring MVC 방식에서는 메서드이 리턴 타입이 void이지만 Spring WebFlux에서는 Mono&#60;Void&#62;가 됨

<br>

### Entity 클래스 정의

#### Member 엔티티

```java
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.relational.core.mapping.Column;
import org.springframework.data.relational.core.mapping.Table;
import java.time.LocalDateTime;

@NoArgsConstructor
@Getter
@Setter
@Table    // (1)
public class Member {
    @Id   // (2)
    private Long memberId;

    private String email;

    private String name;

    private String phone;

    private MemberStatus memberStatus = MemberStatus.MEMBER_ACTIVE;

    @CreatedDate   // (3)
    private LocalDateTime createdAt;

    @LastModifiedDate   // (4)
    @Column("last_modified_at")
    private LocalDateTime modifiedAt;

    public Member(String email) {
        this.email = email;
    }

    public Member(String email, String name, String phone) {
        this.email = email;
        this.name = name;
        this.phone = phone;
    }
    public enum MemberStatus {
        MEMBER_ACTIVE("활동중"),
        MEMBER_SLEEP("휴면 상태"),
        MEMBER_QUIT("탈퇴 상태");

        @Getter
        private String status;

        MemberStatus(String status) {
           this.status = status;
        }
    }
}
```

- R2DBC는 Spring Data JDBC나 Spring Data JPA처럼 애너테이션이나 컬렉션 등을 이용한 연관 관계 매핑은 지원하지 않으므로, 엔티티 클래스의 코드는 Spring Data 패밀리 프로젝트에서 사용되는 최소한의 설정만 포함됨
- (1)에서 @Table 애너테이션을 명시적으로 추가했지만 생략해도 무방함
- (2)에서 memberId 필드에 @Id 애너테이션을 추가함
    - Spring Data 패밀리 프로젝트의 기술들은 식별자에 해당되는 필드에 @Id 애너테이션을 필수로 추가해야 함
- (3)과 (4)에서는 @CreatedDate, @LastModifiedDate 애너테이션을 추가해서 데이터가 저장 또는 업데이트될 때 별도의 날짜/시간 정보를 추가하지 않아도 Spring Data 패밀리에서 지원하는 Auditing 기능을 통해 자동으로 날짜/시간 정보가 테이블에 저장되도록 함

<br>

#### Stamp

```java
import lombok.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.relational.core.mapping.Column;
import org.springframework.data.relational.core.mapping.Table;

import java.time.LocalDateTime;

@NoArgsConstructor
@Getter
@Setter
@Table
public class Stamp {
    @Id
    private long stampId;
    private int stampCount;
    private long memberId;    // (1)

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column("last_modified_at")
    private LocalDateTime modifiedAt;

    public Stamp(long memberId) {
        this.memberId = memberId;
    }
}
```

- R2DBC가 별도의 연관 관계 매핑 기능을 지원하지 않기 때문에 테이블 간의 관계로 따지자면 Stamp 클래스가 Member와 1대1 관계이므로 (1)과 같이 MEMBER 테이블의 식별자에 해당하는 memberId 필드가 외래키 역할을 하도록 추가함
- Spring R2DBC에서 엔티티 간의 연관 관계 매핑을 지원하지 않는 이유는 연관 관계 매핑이 적용되는 순간 내부적으로 Blocking 요소가 포함될 가능성이 있기 때문
- Spring WebFlux의 기술을 효과적으로 잘 활용하기 위해서는 구현 코드 또는 사용하는 써드 파티 라이브러리 등에 Blocking 요소가 포함이 되는지 여부를 잘 판단하는 것이 중요함

<br>

### 서비스 클래스 구현
- Spring WebFlux에서는 모든 데이터의 이동이 Mono 또는 Flux 안에서 이루어짐
<br>

```java
import com.exception.BusinessLogicException;
import com.exception.ExceptionCode;
import com.member.entity.Member;
import com.member.repository.MemberRepository;
import com.stamp.Stamp;
import com.utils.CustomBeanUtils;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.r2dbc.core.R2dbcEntityTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import reactor.core.publisher.Mono;

import static org.springframework.data.relational.core.query.Criteria.where;
import static org.springframework.data.relational.core.query.Query.query;

@Transactional
@Service
public class MemberService {
    private final MemberRepository memberRepository;  // (1)
    private final CustomBeanUtils<Member> beanUtils;
    private final R2dbcEntityTemplate template;       // (2)
    public MemberService(MemberRepository memberRepository, CustomBeanUtils<Member> beanUtils, R2dbcEntityTemplate template) {
        this.memberRepository = memberRepository;
        this.beanUtils = beanUtils;
        this.template = template;
    }

    public Mono<Member> createMember(Member member) {
        return verifyExistEmail(member.getEmail())      // (3)
                .then(memberRepository.save(member))    // (4)
                .map(resultMember -> {
                    // Stamp 저장
                    template.insert(new Stamp(resultMember.getMemberId())).subscribe();  // (5)

                    return resultMember;
                });

    }

    public Mono<Member> updateMember(Member member) {
        return findVerifiedMember(member.getMemberId())    // (6)
                .map(findMember -> beanUtils.copyNonNullProperties(member, findMember))  // (7)
                .flatMap(updatingMember -> memberRepository.save(updatingMember));    // (8)
    }

    @Transactional(readOnly = true)
    public Mono<Member> findMember(long memberId) {
        return findVerifiedMember(memberId);
    }

    @Transactional(readOnly = true)
    public Mono<Page<Member>> findMembers(PageRequest pageRequest) {
        return memberRepository.findAllBy(pageRequest)  // (9)
                .collectList()     // (10)
                .zipWith(memberRepository.count())   // (11)
                .map(tuple -> new PageImpl<>(tuple.getT1(), pageRequest, tuple.getT2()));  // (12)
    }

    public Mono<Void> deleteMember(long memberId) {
        return findVerifiedMember(memberId)
                .flatMap(member -> template.delete(query(where("MEMBER_ID").is(memberId)), Stamp.class))  // (13)
                .then(memberRepository.deleteById(memberId));              // (14)
    }

    private Mono<Void> verifyExistEmail(String email) {
        return memberRepository.findByEmail(email)
                .flatMap(findMember -> {
                    if (findMember != null) {
                        return Mono.error(new BusinessLogicException(ExceptionCode.MEMBER_EXISTS)); // (15)
                    }
                    return Mono.empty();    // (16)
                });
    }

    private Mono<Member> findVerifiedMember(long memberId) {
        return memberRepository
                .findById(memberId)
                .switchIfEmpty(Mono.error(new BusinessLogicException(ExceptionCode.MEMBER_NOT_FOUND))); // (17)
    }
}
```

- Spring Data R2DBC에서 데이터베이스에 접근하는 방법은 두 가지
    - (1)은 우리가 사용해봤던 익숙한 방식인 Repository
    - (2)는 Spring Data R2DBC에서 지원하는 가독성 좋은 SQL 쿼리 빌드 메서드를 이용하는 방식
- (3)에서 회원 정보를 등록하기 전에 이미 존재하는 이메일인지의 여부를 검증함
    - (15)를 보면 email을 조건으로 조회한 후, 존재하는 이메일이면 Exception을 throw하고 있음
        - Spring MVC에서는 throw 키워드를 사용해서 Exception을 throw하지만 Spring WebFlux에서는 error() Operator를 사용해서 Exception을 throw할 수 있음
    - (16)은 존재하는 이메일이 아닌 경우의 처리
        - Spring MVC의 경우 별도의 코드가 필요 없지만 Spring WebFlux의 경우 Mono 안에서 모든 처리가 이루어져야 하므로, Mono.empty()를 리턴해 주어야 다음 동작을 진행할 수 있음
- (3)에서 존재하는 이메일이 아닐 경우 다음 동작을 진행함
    - (4)의 then() Operator는 이 전에 동작하고 있던 Sequence를 종료하고 새로운 Sequence를 시작하게 해주는 Operator
    - (4)에서는 회원 정보를 저장 한 뒤에 다음 동작을 수행하도록 리턴 값으로 Mono&#60;Member&#62;를 리턴하며, 여기서의 Mono&#60;Member&#62;는 데이터베이스에 저장된 데이터
- Spring MVC 기반 코드에서는 JPA를 CASCADE 기능을 이용해서 회원 정보를 저장하면 스탬프 정보까지 자동으로 테이블에 저장을 해주지만 Spring Data R2DBC의 경우, 직접 테이블에 저장하는 코드가 필요함
    - (5)에서 R2dbcEntityTemplate의 insert() 메서드를 이용해서 스탬프 정보를 테이블에 저장함
    - 중요한 포인트는 insert() 메서드를 호출하고, subscribe()를 호출해야 된다는 것
    - map() Operator에서 리턴하는 값은 Controller 쪽으로 전달하는 회원 정보
    - 스탬프 정보는 회원 정보를 저장하는 Operator 체인 내부에 별도로 존재하는 Inner Sequence이기 때문에 subscribe()를 호출해야지만 테이블에 데이터를 저장하는 동작을 수행함
    - 리액티브 프로그래밍의 특징 중 하나가 바로 subscribe() 메서드를 호출하지 않으면 아무 동작을 수행하지 않는다는 것
- (6)에서는 회원 정보 수정 전에 존재하는 회원인지 여부를 확인함
    - (17)에서 switchIfEmpty() Operator를 사용하여 회원이 존재하지 않는 다면 Exception을 throw함
    - switchIfEmpty() Operator는 emit되는 데이터가 없다면 switchIfEmpty() Operator의 파라미터로 전달되는 Publisher가 대체 동작을 수행할 수 있게 해주는 Operator
- (7)은 회원 정보 중에서 request body에 포함된 정보만 테이블에 업데이트되도록 해주는 유틸리티 클래스
    - beanUtils.copyNonNullProperties(member, findMember))에서 첫 번째 파라미터는 request body에 포함된 데이터이며, 두 번째 파라미터는 테이블에서 조회한 회원의 기존 데이터
    - 첫 번째 파라미터(member)에서 null이 아닌 필드의 값만 두 번째 파라미터(findMember)의 동일한 필드에 덮어 씌우기 때문에 실제 테이블에 저장 전, 간편하게 회원 정보 필드를 업데이트할 수 있음
- (7)에서 업데이트된 Member 객체를 (8)에서 테이블에 저장함
- (9)에서는 회원 정보 목록에 페이지네이션 처리함
    - Spring MVC 기반 코드에서는 PageRequest 객체를 MemberService 클래스에서 생성했지만 여기서는 PageRequest 객체가 Sequence 내부에서 재사용되어야하기 때문에 Controller 쪽에서 미리 생성한 PageRequest 객체를 findMembers() 메서드의 파라미터로 전달함
- 파라미터로 전달받은 PageRequest 객체와 (10)에서 collectList() Operator를 통해 변환되는 List 객체, (11)의 테이블에서 조회되는 전체 데이터의 건 수는 모두 (12)에서 PageImpl 객체를 생성하기 위해 필요한 정보
    - zipWith() Operator는 zipWith()를 호출하는 Mono와 zipWith()의 파라미터로 주어지는 Mono에서 emit하는 두 개의 데이터를 Tuple2 객체로 결합한 후, Downstream에 전달하는 Operator
    - collectList() Operator는 List를 emit하고, memberRepository.count()는 Long 타입의 전체 건 수를 emit함
    - map() Operator가 전달받는 Tuple2 객체인 tuple.getT1()의 값은 List이고, tuple.getT2()의 값은 전체 건 수(Long)
- (13)에서는 R2dbcEntityTemplate의 delete() 메서드와 SQL 쿼리 빌드 메서드 체인을 통해 스탬프 정보를 삭제함
    - MEMBER 테이블과 STAMP 테이블은 외래키로 관계를 맺고 있기 때문에 MEMBER 테이블의 식별자를 외래키로 가지는 STAMP 테이블의 스탬프 정보를 먼저 삭제해 주어야 함
- (13)에서 스탬프 정보를 삭제했으니 (14)에서는 회원 정보를 삭제함
    - 여기서는 MemberRepository의 deleteById() 메서드를 이용해서 회원 정보를 삭제함

<br>

### Repository 구현

```java
import com.member.entity.Member;
import org.springframework.data.domain.Pageable;
import org.springframework.data.r2dbc.repository.R2dbcRepository;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public interface MemberRepository extends R2dbcRepository<Member, Long> {  // (1)
    Mono<Member> findByEmail(String email);    // (2)
    Flux<Member> findAllBy(Pageable pageable); // (3)
}
```

- (1)의 R2dbcRepository 인터페이스는 Spring Data R2DBC에서 사용하는 Repository
    - R2dbcRepository는 기본적인 CRUD 기능과 페이지네이션, 정렬 기능을 모두 포함하고 있음
- (2)와 (3)을 보면 Spring Data R2DBC에서 조회되는 데이터는 모두 Mono 또는 Flux임을 알 수 있음
    - 이를 통해 Controller부터 Repository까지 완전한 Non-Blocking 동작을 수행할 수 있게 됨