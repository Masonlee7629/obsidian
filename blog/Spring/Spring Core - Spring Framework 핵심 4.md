태그: #Spring
연결문서: [Spring Core - Spring Framework 핵심 5](Spring%20Core%20-%20Spring%20Framework%20핵심%205.md)

# Spring Framework 핵심

## DI(Dependency Injection)

### Component Scan

스프링은 설정 정보 없이 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.

-   지금까지는 스프링 빈을 등록할 때 자바 코드의 @Bean or XML의 등의 설정 정보에 등록할 스프링 빈들을 직접 작성했다.
-   이렇게 수작업으로 등록하게 되면 설정 정보가 커지고, 누락하는 등 다양한 문제가 발생할 수 있다.
-   @ComponentScan은 @Component가 붙은 모든 클래스를 스프링 빈으로 등록해주기 때문에 설정 정보에 붙여주면 된다.
    -   의존관계도 자동으로 주입하는 @Autowired 기능도 제공한다.
    -   기존에 작성하던 DependencyConfig와의 비교한다면 @Bean으로 등록한 클래스를 볼 수 없다.
    -   **컴포넌트 스캔을 사용하면 @Configuration이 붙은 설정 정보도 자동으로 등록된다.**
        -   @Configuration이 붙은 설정 정보가 자동 등록되는 이유는 @Configuration 코드에 @Component 애너테이션이 붙어있기 때문이다.
            -   기존에 작성한 AppConfig이 있다면 정상적인 작동이 되지 않는다.
            -   새 프로젝트로 진행할 경우 문제가 되지 않는다.
        -   DependencyConfig 등 @Configuration 설정이 된 파일이 있을 시 @ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class)) 코드 추가

#### 예제 코드

```
// MemberService.java
@Component
public class MemberService {

  private final MemberRepository memberRepository;

  @Autowired
  public MemberService(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }


  public void createMember(Member member) {
    memberRepository.postMember(member);
  }

  public Member getMember(Long memberId) {
    return memberRepository.getMember(memberId);
  }

  public void deleteMember(Long memberId) {
    memberRepository.deleteMember(memberId);
  }
}
```

```
// CoffeeService.java
@Component
public class CoffeeService {

  private static CoffeeRepository coffeeRepository;

  @Autowired
  public CoffeeService(CoffeeRepository coffeeRepository) {
    this.coffeeRepository = coffeeRepository;
  }

  public void createCoffee(Coffee coffee) {
    coffeeRepository.postCoffee(coffee);
  }

  public Coffee editCoffee(Long coffeeId, String korName, int price) {
    return coffeeRepository.patchCoffee(coffeeId, korName, price);
  }

  public Coffee getCoffee(Long coffeeId) {
    return coffeeRepository.getCoffee(coffeeId);
  }
  public void deleteCoffee(Long coffeeId) {
    coffeeRepository.deleteCoffee(coffeeId);
  }
}
```

```
// MemberRepository.java
@Component
public class MemberRepository {

  private static Map<Long, Member> members = new HashMap<>();

  public void postMember(Member member) {
    members.put(member.getMemberId(), member);
  }

  public Member getMember(Long memberId) {
    return members.get(memberId);
  }

  public void deleteMember(Long memberId) {
    members.remove(memberId);
  }

}
```

```
// CoffeeRepository.java
@Component
public class CoffeeRepository {

  private static Map<Long, Coffee> drinks = new HashMap<>();

  public void postCoffee(Coffee coffee) {
    drinks.put(coffee.getCoffeeId(), coffee);
  }

  public Coffee patchCoffee(Long coffeeId, String korName, int price) {
    Coffee drink = drinks.get(coffeeId);
    drink.setKorName(korName);
    drink.setPrice(price);

    return drinks.put(coffeeId, drink);
  }

  public Coffee getCoffee(Long coffeeId) {
    return drinks.get(coffeeId);
  }

  public void deleteCoffee(Long coffeeId) {
    drinks.remove(coffeeId);
  }

}
```

-   코드들의 특징을 보면 @Component와 @Autowired를 볼 수 있다.
    -   @ComponentScan - @ComponentScan이 등록된 곳에서 @Component를 가져오기 위해 사용된다.
    -   @Autowired - 생성자 의존성 주입에 필요한 설정 정보 대신 의존관계 자동 주입을 해준다.

#### basePackages

탐색할 패키지의 시작 위치를 지정하고, 해당 패키지부터 하위 패키지까지 모두 탐색한다.

-   @ComponentScan()의 매개변수로 basePackages = “”를 줄 수 있다.
-   지정하지 않으면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.
    -   설정 정보 클래스의 위치를 프로젝트 최상단에 두고 패키지 위치는 지정하지 않는 방법이 가장 편할 수 있다.
-   스프링 부트를 사용하면 @SpringBootApplication 를 프로젝트 시작 루트 위치에 두는 것이 좋다.
    -   @SpringBootApplication에 @ComponentScan이 들어있다.

#### 컴포넌트 스캔 기본 대상

-   @Component : 컴포넌트 스캔에서 사용된다.
-   @Controller & @RestController : 스프링 MVC 및 REST 전용 컨트롤러에서 사용된다.
-   @Service : 스프링 비즈니스 로직에서 사용된다.
    -   특별한 처리를 하지 않는다.
    -   개발자들이 핵심 비즈니스 로직이 여기에 있다는 비즈니스 계층 인식에 도움이 된다.
-   @Repository : 스프링 데이터 접근 계층에서 사용된다.
    -   스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
-   @Configuration : 스프링 설정 정보에서 사용된다.
    -   스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리한다.

**필터**

-   includeFilters : 컴포넌트 스캔 대상을 추가로 지정한다.
-   excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정한다.
-   **FilterType 옵션**
    -   ANNOTATION: 기본값, 애너테이션으로 인식해서 동작한다.
    -   ASSIGNABLE\_TYPE: 지정한 타입과 자식 타입을 인식해서 동작한다.
    -   ASPECTJ: AspectJ 패턴을 사용한다.
    -   REGEX: 정규 표현식을 나타낸다.
    -   CUSTOM: TypeFilter라는 인터페이스를 구현해서 처리한다.

---

### 다양한 의존 관계 주입 방법

스프링에서 DI할 수 있는 의존 관계 주입에는 네 가지 방법이 있다.

-   생성자 주입
-   수정자 주입(setter 주입)
-   필드 주입
-   일반 메서드 주입

#### 생성자 주입

생성자는 통하여 의존 관계를 주입 받는 방법이다.

생성자에 @Autowired를 하면 스프링 컨테이너에 @Component로 등록된 빈에서 생성자에 필요한 빈들을 주입한다.

-   특징
    -   생성자 호출 시점에 딱 한 번만 호출되는 것이 보장된다.
    -   **불변과 필수** 의존 관계에 사용된다.
    -   생성자가 한 개만 존재하는 경우에는 @Autowired를 생략해도 자동 주입된다.
    -   NullPointerException을 방지할 수 있다.
    -   주입받을 필드를 final로 선언 가능하다.
-   예제

```
@Component
public class CoffeeService {
  private final MemberRepository memberRepository;
  private final CoffeeRepository coffeeRepository;

  @Autowired
  public CoffeeService(MemberRepository memberRepository, CoffeeRepository coffeeRepository) {
    this.memberRepository = memberRepository;
    this.coffeeRepository = coffeeRepository;
  }
}
```

#### 수정자 주입(setter 주입)

setter라 불리는, 필드의 값을 변경하는 수정자 메서드를 통해서 의존 관계를 주입하는 방법이다.

-   특징
    -   **선택과 변경** 가능성이 있는 의존 관계에 사용된다.
    -   자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.
    -   생성자 주입과 차이점은 생성자 대신 set필드명 메서드를 생성하여 의존관계를 주입하게 된다.
        -   @Component가 실행하는 클래스를 스프링 빈으로 등록한다.
        -   스프링 빈으로 등록한 다음 의존 관계를 주입하게 되는데 @Autowired가 있는 것들을 자동으로 주입하게 된다.
-   예제

```
@Component
public class CoffeeService {
  private MemberRepository memberRepository;
  private CoffeeRepository coffeeRepository;

  @Autowired
  public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }

  @Autowired
  public void setCoffeeRepository(CoffeeRepository coffeeRepository) {
    this.coffeeRepository = coffeeRepository;
  }
}
```

-   **생성자가 1개 일 때 @Autowired가 없어도 작동이 되는 이유**  
    스프링이 해당 클래스 객체를 생성하여 빈에 넣어야하는데 생성할 때 생성자를 부를 수 밖에 없게 된다. 그렇기 때문에 빈을 등록하면서 의존 관계 주입도 같이 발생하게 된다.

#### 필드 주입

필드에 @Autowired를 붙여서 바로 주입하는 방법이다.

-   특징
    -   코드가 간결해서 예전에 많으 사용된 방식이지만, 외부에서 변경이 불가능하여 테스트하기 힘들다는 단점이 있다.
    -   DI 프레임워크가 없으면 아무것도 할 수 없다.
    -   실제 코드와 상관 없는 특정 테스트를 하고 싶을 때 사용할 수 있다.
    -   정상적으로 작동하게 하려면 결국 setter가 필요하게 돼서 수정자 주입을 사용하는게 더 편리해진다.
-   예제

```
@Component
public class CoffeeService {
  @Autowired
  private MemberRepository memberRepository;
  @Autowired
  private CoffeeRepository coffeeRepository;
}
```

#### 일반 메서드 주입

일반 메서드를 사용해 주입하는 방법이다.

-   특징
    -   한 번에 여러 필드를 주입 받을 수 있다.
    -   일반적으로 사용되지 않는다.

#### 옵션 처리

주입할 스프링 빈이 없을 때 동작해야하는 경우가 있다.

-   @Autowired만 사용하는 경우 required 옵션 기본값인 true가 사용되어 자동 주입 대상이 없으면 오류가 발생하는 경우가 있을 수 있다.
-   스프링 빈을 옵셔널하게 해둔 상태에서 등록이 되지 않고, 기본 로직으로 동작하게 하는 경우
-   자동 주입 대상 옵션 처리 방법
    -   @Autowired(required=false) : 자동 주입할 대상이 없으면 **수정자 메서드 자체가 호출되지 않게 된다.**
    -   org.springframework.lang.@Nullable : 자동 주입할 대상이 없으면 null이 입력된다.
    -   Optional<> : 자동 주입할 대상이 없으면 Optional.empty가 입력된다.
-   예제

```
public class AutowiredTest {
  public static void main(String[] args) {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
  }

  static class TestBean {

    @Autowired(required = false)
    public void setNoBean1(Member noBean1) {
      System.out.println("noBean1 = " + noBean1);
    }

    @Autowired
    public void setNoBean2(@Nullable Member noBean2) {
      System.out.println("noBean2 = " + noBean2);
    }

    @Autowired
    public void setNoBean3(Optional<Member> noBean3) {
      System.out.println("noBean3 = " + noBean3);
    }
  }
}

// 출력 결과
noBean2 = null
noBean3 = Optional.empty
```

#### 생성자 주입을 사용해야 하는 이유

과거에는 수정자, 필드 주입을 많이 사용했지만, 최근에는 대부분 생성자 주입 사용을 권장한다.

-   **불변**
    -   의존 관계 주입은 처음 애플리케이션이 실행될 때 대부분 정해지고 종료 전까지 변경되지 않고 변경되어서는 안된다.
    -   수정자 주입 같은 경우에는 이름 메서드를 public으로 열어두어 변경이 가능하기 때문에 적합하지 않다.
    -   누군가 실수로 변경할 수도 있고, 애초에 변경하면 안되는 메서드가 변경할 수 있게 설계하는 것은 좋은 방법이 아니다.
    -   생성자 주입은 객체를 생성할 때 최초로 한 번만 호출되고, 그 이후에는 다시는 호출되는 일이 없기 때문에 불변하게 설계할 수 있다.

-   **누락**
    -   호출했을 때는 NPE(Null PointException)이 발생하는데 의존 관계 주입이 누락되었기 때문에 발생한다.
    -   생성자 주입을 사용하면 주입 데이터 누락 시 **컴파일 오류**가 발생한다.

-   **final 키워드 사용 가능**
    -   생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다.
    -   생성자에서 값이 설정되지 않으면 컴파일 시점에서 오류를 확인할 수 있다.
    -   java: variable (데이터 이름) might not have been initialized
    -   생성자 주입을 제외한 나머지 주입 방식은 생성자 이후에 호출되는 형태이므로 final 키워드를 사용할 수 없다.

-   **순환 참조**
    -   순환 참조를 방지할 수 있다.
    -   개발하다보면 여러 컴포넌트 간에 의존성이 생긴다.(A -> B를 참조하고, B -> A를 참조)
    -   필드 주입과 수정자 주입은 빈이 생성된 후에 참조를 하기 때문에 애플리케이션이 어떠한 오류나 경고 없이 구동된다.
        -   실제 코드가 호출될 때까지 문제를 알 수 없다.
    -   생성자를 통해 주입하게 되면 BeanCurrentlyInCreationException이 발생하게 된다.