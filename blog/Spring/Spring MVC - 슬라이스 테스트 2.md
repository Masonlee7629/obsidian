태그: #Spring
연결문서: [Spring MVC - Mockito](Spring%20MVC%20-%20Mockito.md)

# 슬라이스 테스트(Slice Test)

## 데이터 액세스 계층 테스트

현재 데이터 액세스 계층에서 사용하고 있는 기술은 Spring Data JPA이며 Spring에서는 JPA에 대한 테스트를 쉽게 진행할 수 있는 몇 가지 방법들을 제공한다.

### 데이터 액세스 계층을 테스트 하기 위한 한 가지 규칙

-   **DB의 상태를 테스트 케이스 실행 이전으로 되돌려서 깨끗하게 만든다.**

### Repository 테스트 예시

#### saveMember() 테스트 예시

```
import com.codestates.member.entity.Member;
import com.codestates.member.repository.MemberRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest   // (1)
public class MemberRepositoryTest {
    @Autowired
    private MemberRepository memberRepository;   // (2)

    @Test
    public void saveMemberTest() {
        // given  (3)
        Member member = new Member();
        member.setEmail("mason@gmail.com");
        member.setName("Mason");
        member.setPhone("010-1111-2222");

        // when  (4)
        Member savedMember = memberRepository.save(member);

        // then  (5)
        assertNotNull(savedMember); // (5-1)
        assertTrue(member.getEmail().equals(savedMember.getEmail()));
        assertTrue(member.getName().equals(savedMember.getName()));
        assertTrue(member.getPhone().equals(savedMember.getPhone()));
    }
}
```

1.  (1)의 `@DataJpaTest` 는 Spring에서 데이터 액세스 계층을 테스트하기 위한 가장 핵심적인 방법이다.  
      
    `@DataJpaTest` 애너테이션을 테스트 클래스에 추가함으로써, `MemberRepository`의 기능을 정상적으로 사용하기 위한 Configuration을 Spring이 자동으로 해준다.  
      
    `@DataJpaTest` 애너테이션은 `@Transactional` 애너테이션을 포함하고 있기 때문에 **하나의 테스트 케이스 실행이 종료되는 시점에 데이터베이스에 저장된 데이터는 rollback 처리된다.**
2.  (2)에서 테스트 대상 클래스인 `MemberRepository`를 DI 받는다.
3.  (3)에서 테스트 할 회원 정보 데이터를 준비한다.
4.  (4)에서 회원 정보를 저장한다.
5.  (5)에서 회원 정보가 저장되었는지 검증(Assertion)한다.
    -   (5-1)과 같이 회원 정보를 정상적으로 저장한 뒤, 리턴 값으로 반환된 `Member` 객체가 `null`이 아닌지를 검증한다.
    -   `assertTrue()` 메서드로 리턴 값으로 반환된 `Member` 객체의 필드들이 테스트 데이터와 일치하는지 검증한다.

#### findMember() 테스트 예시

```
import com.codestates.member.entity.Member;
import com.codestates.member.repository.MemberRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
public class MemberRepositoryTest {
    ...
    ...
    @Test
    public void findByEmailTest() {
        // given (1)
        Member member = new Member();
        member.setEmail("hgd@gmail.com");
        member.setName("홍길동");
        member.setPhone("010-1111-2222");

        // when 
        memberRepository.save(member);  // (2)
        Optional<Member> findMember = memberRepository.findByEmail(member.getEmail()); // (3)

                // then (4)
        assertTrue(findMember.isPresent()); // (4-1)
        assertTrue(findMember.get().getEmail().equals(member.getEmail())); // (4-2)
    }
}
```

1.  (1)에서 테스트 할 회원 정보 데이터를 준비한다.
2.  (2)에서 회원 정보를 저장한다.
3.  (3)에서 저장된 회원 정보 중 이메일에 해당하는 회원 정보의 조회를 테스트하기 위해 `findByEmail()`로 확인한다.
4.  (4)에서 회원 정보의 조회가 정상적으로 이루어지는지 검증(Assertion)한다.
    -   (4-1)에서 조회된 회원 정보가 `null`이 아닌지를 검증한다.
    -   (4-2)에서 조회한 회원의 이메일 주소와 테스트 데이터의 이메일이 일치하는지 검증한다.

