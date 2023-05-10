태그: #Spring
연결문서: [Spring MVC - 트랜잭션 3](Spring%20MVC%20-%20트랜잭션%203.md)

# 트랜잭션

## 선언형 방식의 트랜잭션 적용

Spring에서 선언형 방식으로 트랜잭션을 적용하는 방법은 크게 두 가지이다.

-   애너테이션 방식의 트랜잭션 적용
-   AOP 방식의 트랜잭션 적용

### Spring Boot에서의 트랜잭션 설정

```
@Configuration
@EnableTransactionManagement
public class JpaConfig{
    ...
    ...
    // (1)
    @Bean
    public DataSource dataSource() {
        final DriverManagerDataSource dataSource = new DriverManagerDataSource();

                ...
                ...

        return dataSource;
    }

    @Bean
    public PlatformTransactionManager transactionManager(){
        JpaTransactionManager transactionManager
                = new JpaTransactionManager();    // (2)
        transactionManager.setEntityManagerFactory(
                entityManagerFactoryBean().getObject() );
        return transactionManager;
    }
}
```

트랜잭션은 기본적으로 데이터베이스와의 인터랙션과 관련이 있기 때문에 (1)과 같이 데이터베이스 커넥션 정보를 포함하고 있는 Datasource가 필요하다.  
  

Spring에서 트랜잭션은 기본적으로 `PlatformTransactionManager`에 의해 관리되며, `PlatformTransactionManager` 인터페이스를 구현해서 해당 데이터 액세스 기술에 맞게 유연하게 트랜잭션을 적용 할 수 있도록 추상화 되어 있다.  
  

현재 사용하는 데이터 액세스 기술이 JPA이기 때문에 (2)와 같이 `PlatformTransactionManager`의 구현 클래스인 `JpaTransactionManager`를 사용한다.

---

### 애너테이션 방식의 트랜잭션 적용

Spring에서 트랜잭션을 적용하는 가장 간단한 방법은 `@Transactional`이라는 애너테이션을 트랜잭션이 필요한 영역에 추가해 주는 것이다.

#### 클래스 레벨에 `@Transactional` 적용

```
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional   // (1)
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());

        return memberRepository.save(member);
    }

        ...
        ...
}
```

(1)과 같이 `@Transactional` 애너테이션을 클래스 레벨에 추가하면 기본적으로 해당 클래스에서 MemberRepository의 기능을 이용하는 모든 메서드에 트랜잭션이 적용된다.

#### JPA 로그 레벨 설정

```
spring:
  h2:
    console:
      enabled: true
      path: /h2
  datasource:
    url: jdbc:h2:mem:test
  jpa:
    hibernate:
...
...

logging:         # (1)
  level:
    org:
      springframework:
        orm:
          jpa: DEBUG
```

애플리케이션을 실행시키기 전에 트랜잭션이 어떻게 적용되는지 로그로 확인할 수 있도록 JPA의 로그 레벨을 `application.yml`에 추가한다.  
  

로그 레벨을 ‘`DEBUG`’ 레벨로 설정하면 JPA 내부에서 ‘`DEBUG`’ 로그 레벨을 지정한 부분의 로그를 확인할 수 있다.

#### 트랜잭션 적용 유무 확인

```
// (1) 트랜잭션 생성
Creating new transaction with name [com.codestates.member.service.MemberService.
createMember]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
...
...
Initiating transaction commit
...
...
// (2) 커밋
Committing JPA transaction on EntityManager [SessionImpl(1508768151<open>)]

// (3) 트랜잭션 종료
Not closing pre-bound JPA EntityManager **after transaction** 

// (4) EntityManager 종료
Closing JPA EntityManager in OpenEntityManagerInViewInterceptor
```

-   (1)에서 `MemberService`의 `createMember()` 메서드가 호출되며 새로운 트랜잭션이 생성
-   (2)를 통해 트랜잭션에서 commit이 일어나는 것을 확인
-   (3)에서 트랜잭션이 종료됨을 확인
-   (4)에서 JPA의 `EntityManager`를 종료함을 확인

#### rollback 동작 유무 확인

```
@Service
@Transactional
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());
        Member resultMember = memberRepository.save(member);

        if (true) {    // (1)
            throw new RuntimeException("Rollback test");
        }
        return resultMember;
    }

        ...
        ...
}
```

`createMember()` 메서드에서 회원 정보를 저장하고 메서드가 종료되기 전에 (1)과 같이 강제로 `RuntimeException` 이 발생하도록 수정한 뒤, 회원 정보를 저장한다.

```
Initiating transaction rollback  // (1)

// (2)
Rolling back JPA transaction on EntityManager [SessionImpl(1632113004<open>)]

Not closing pre-bound JPA EntityManager after transaction
...
...
java.lang.RuntimeException: Rollback test
...
...
Closing JPA EntityManager in OpenEntityManagerInViewInterceptor
```

로그를 확인해보면 rollback이 정상적으로 동작하는 것을 확인할 수 있다.  
이는 H2 웹 콘솔을 통해서 MEMBER 테이블을 조회해도 저장된 회원 정보가 없는 것을 확인할 수 있다.  
  

체크 예외의 경우, `@Transactional` 애너테이션만 추가해서는 rollback이 되지 않는다.  
  

따라서, 캐치(catch)한 후에 해당 예외를 복구할지 회피할지 등의 적절한 예외 전략을 고민해 볼 필요가 있다.  
  

만약 별도의 예외 전략을 짤 필요가 없다면 `@Transactional(rollbackFor = {SQLException.class, DataFormatException.class})`와 같이 해당 체크 예외를 직접 지정해주거나 언체크 예외(unchecked exception)로 감싸서 rollback이 동작하도록 할 수 있다.

#### 메서드 레벨에 `@Transactional` 적용

```
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional  // (1)
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    // (2)
    @Transactional(readOnly = true)
    public Member findMember(long memberId) {
        return findVerifiedMember(memberId);
    }
        ...
        ...
}
```

클래스 레벨의 `@Transactional` 애너테이션 이외에 (2)와 같이 메서드 레벨에 `@Transactional(readOnly = true)`를 추가했다.  
  

이런 경우, 해당 메서드는 **읽기 전용 트랜잭션**이 적용된다.  
  

`@Transactional(readOnly = true)`로 설정하면 JPA 내부적으로 영속성 컨텍스트를 flush하지 않는다. 그리고 읽기 전용 트랜잭션일 경우, 변경 감지를 위한 스냅샷 생성도 하지 않는다.  
  

이처럼 flush 처리를 하지 않고, 스냅샷을 생성하지 않으므로 불필요한 추가 동작을 줄일 수 있다.  
  

따라서, **조회 메서드**에서 `@Transactional(readOnly = true)` 로 설정하면 JPA가 자체적으로 성능 최적화 과정을 거치도록 할 수 있다.

#### 클래스 레벨과 메서드 레벨의 트랜잭션 적용 순서

-   클래스 레벨에만 `@Transactional`이 적용된 경우
    -   클래스 레벨의 `@Transactional` 애너테이션이 메서드에 일괄 적용
-   클래스 레벨과 메서드 레벨에 함께 적용된 경우
    -   메서드 레벨의 `@Transactional` 애너테이션이 적용
    -   만약 메서드 레벨에 `@Transactional` 애너테이션이 적용되지 않았을 경우, 클래스 레벨의 `@Transactional` 애너테이션이 적용

#### 트랜잭션 격리 레벨(Isolation Level)

ACID 원칙에서 트랜잭션은 다른 트랜잭션에 영향을 주지 않고, 독립적으로 실행되어야 하는 격리성이 보장되어야 하는데 Spring은 이런 격리성을 조정할 수 있는 옵션을 `@Transactional` 애너테이션의 `isolation` 애트리뷰트를 통해 제공하고 있다.  
  

트랜잭션의 격리 레벨을 일반적으로 데이터베이스나 데이터 소스에 설정된 격리 레벨을 따르는 것이 권장된다.

-   `Isolation.DEFAULT`
    -   데이터베이스에서 제공하는 기본 값
-   `Isolation.READ_UNCOMMITTED`
    -   다른 트랜잭션에서 커밋하지 않은 데이터를 읽는 것을 허용
-   `Isolation.READ_COMMITTED`
    -   다른 트랜잭션에 의해 커밋된 데이터를 읽는 것을 허용
-   `Isolation.REPEATABLE_READ`
    -   트랜잭션 내에서 한 번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회되도록 함
-   `Isolation.SERIALIZABLE`
    -   동일한 데이터에 대해서 동시에 두 개 이상의 트랜잭션이 수행되지 못하도록 함

---

### AOP 방식의 트랜잭션 적용

AOP를 이용해 트랜잭션을 적용하면 `@Transactional` 애너테이션 조차도 비즈니스 로직에 적용하지 않고, 트랜잭션을 적용할 수 있다.

#### AOP 방식으로 트랜잭션 적용 순서

```
import org.springframework.aop.Advisor;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionManager;
import org.springframework.transaction.interceptor.*;
import java.util.HashMap;
import java.util.Map;

// (1)
@Configuration
public class TxConfig {
    private final TransactionManager transactionManager;

        // (2)
    public TxConfig(TransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }

    @Bean
    public TransactionInterceptor txAdvice() {
        NameMatchTransactionAttributeSource txAttributeSource =
                                    new NameMatchTransactionAttributeSource();

                // (3)
        RuleBasedTransactionAttribute txAttribute =
                                        new RuleBasedTransactionAttribute();
        txAttribute.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);

                // (4)
        RuleBasedTransactionAttribute txFindAttribute =
                                        new RuleBasedTransactionAttribute();
        txFindAttribute.setPropagationBehavior(
                                        TransactionDefinition.PROPAGATION_REQUIRED);
        txFindAttribute.setReadOnly(true);

                // (5)
        Map<String, TransactionAttribute> txMethods = new HashMap<>();
        txMethods.put("find*", txFindAttribute);
        txMethods.put("*", txAttribute);

                // (6)
        txAttributeSource.setNameMap(txMethods);

                // (7)
        return new TransactionInterceptor(transactionManager, txAttributeSource);
    }

    @Bean
    public Advisor txAdvisor() {
                // (8)
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.codestates.coffee.service." +
                "CoffeeService.*(..))");

        return new DefaultPointcutAdvisor(pointcut, txAdvice());  // (9)
    }
}
```

1.  AOP 방식으로 트랜잭션을 적용하기 위한 `Configuration` 클래스 정의
2.  `TransactionManager` DI
3.  트랜잭션 어드바이스용 `TransactionInterceptor` 빈 등록  
    Spring에서는 `TransactionInterceptor`를 이용해서 대상 클래스 또는 인터페이스에 트랜잭션 경계를 설정하고 트랜잭션을 적용할 수 있다.
    -   트랜잭션 애트리뷰트 지정
        -   트랜잭션 애트리뷰트는 `메서드 이름 패턴`에 따라 구분해서 적용 가능하기 때문에 (3), (4)와 같이 트랜잭션 애트리뷰트를 설정할 수 있음
        -   (3)은 조회 메서드를 제외한 공통 트랜잭션 애트리뷰트이며, (4)는 조회 메서드에 적용하기 위한 트랜잭션 애트리뷰트
    -   트랜잭션을 적용할 메서드에 트랜잭션 애트리뷰트 매핑
        -   설정한 트랜잭션 애트리뷰트는 (5)와 같이 Map에 추가하는데, Map은 key를 `메서드 이름 패턴`으로 지정해서 각각의 트랜잭션 애트리뷰트를 추가
        -   트랜잭션 애트리뷰트를 추가한 Map 객체를 (6)과 같이 `txAttributeSource.setNameMap(txMethods)`으로 전달
    -   `TransactionInterceptor` 객체 생성
        -   (7)과 같이 `TransactionInterceptor` 의 생성자 파라미터로 `transactionManager`와 `txAttributeSource`를 전달
4.  Advisor 빈 등록
    -   포인트컷 지정
        -   트랜잭션 어드바이스인 `TransactionInterceptor`를 타겟 클래스에 적용하기 위해 포인트컷을 지정
        -   (8)과 같이 `AspectJExpressionPointcut` 객체를 생성한 후, 포인트컷 표현식을 이용해 타겟 클래스로 지정
    -   Advisor 객체 생성
        -   (9)와 같이 `DefaultPointcutAdvisor` 의 생성자 파라미터로 포인트컷과 어드바이스를 전달