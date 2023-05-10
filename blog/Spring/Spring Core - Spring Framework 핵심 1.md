태그: #Spring
연결문서: [Spring Core - Spring Framework 핵심 2](Spring%20Core%20-%20Spring%20Framework%20핵심%202.md)

# Spring Framework 핵심

## DI(Dependency Injection)

### DI란?

DI(의존성 주입)는 IoC라는 원칙을 구현하기 위해서 사용되는 방법 중 하나이다.  
  

일반적으로는 생성자를 통한 의존관계 주입을 사용한다.  
그러나 이런 방법을 사용하면 객체를 생성할 때마다 직접 주입할 객체를 작성해야하는 불편함이 생긴다.  
  

이를 해결하기 위한 일차적인 방법으로 **의존성 주입을 관리하는 설정 파일을 만들어서 관심사 분리**를 통해 해결할 수 있다.  
  

이는 프레임워크에서 사용하는 방식이 아닌, 단순 컨테이너를 활용해 객체의 의존관계를 주입하는 방식이다.

```
public class DependencyConfig {

  public MemberService memberService() {
    return new MemberService(memberRepository());
  }

  public MemberRepository memberRepository() {
    return new MemberRepository();
  }

  public CoffeeService coffeeService() {
    return new CoffeeService(coffeeRepository());
  }

  public CoffeeRepository coffeeRepository() {
    return new CoffeeRepository();
  }
}
```

### 스프링 컨테이너(Spring Container)

**스프링 컨테이너는 스프링 프레임워크의 핵심 컴포넌트**이다.  
스프링 컨테이너는 내부에 존재하는 애플리케이션 빈의 생명주기를 관리한다.

-   Bean 생성, 관리, 제거 등의 역할을 담당한다.

#### 스프링 컨테이너란?

ApplicationContext를 스프링 컨테이너라 하고 인터페이스로 구현되어 있다.(다형성 적용)

-   스프링 컨테이너는 XML, 애너테이션 기반의 자바 설정 클래스로 만들 수 있다.
-   예전에는 개발자가 xml을 통해 모두 설정해줬지만, 이러한 복잡한 부분들을 Spring Boot를 사용하면서 거의 사용하지 않는다.
-   빈의 인스턴스화, 구성, 전체 생명 주기 및 제거까지 처리한다.
    -   컨테이너는 개발자가 정의한 Bean을 객체로 만들어 관리하고 개발자가 필요로 할 때 한다.
-   스프링 컨테이너를 통해 원하는 만큼 많은 객체를 가질 수 있다.
-   의존성 주입을 통해 애플리케이션의 컴포넌트를 관리한다.
    -   스프링 컨테이너는 서로 다른 빈을 연결해 애플리케이션의 빈을 연결하는 역할을 한다.
    -   개발자는 모듈 간에 의존 및 결합으로 인해 발생하는 문제로부터 자유로울 수 있다.
    -   메서드가 언제, 어디서 호출되어야 하는지, 메서드를 호출하기 위해 필요한 매개 변수를 준비해서 전달하지 않는다.

#### 스프링 컨테이너 사용 이유

객체를 사용하기 위해서 new 생성자를 사용해야 했으며, 애플리케이션에서 이런 객체가 무수히 많이 존재하고 서로 참조하게 되어 있었다.(서로 참조가 심할수록 의존성이 높다고 표현하며, 낮은 결합도와 높은 캡슐화가 객체지향 프로그래밍의 핵심 중 하나인데 높은 의존성은 이를 지키지 못하는 방법이 될 수 밖에 없다.)  
여기서 객체 간의 의존성을 낮추기 위해 Spring 컨테이너가 사용된다.

스프링 컨테이너를 사용하면 구현 클래스에 있는 의존성을 제거하고 인터페이스에만 의존하도록 설계 가능하다.

#### 스프링 컨테이너 생성 과정

1.  스프링 컨테이너 생성
    
    ```
    new AnnotationConfigApplicationContext(구성정보.class)
    ```
    
2.  스프링 빈 등록
    -   스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈 등록
    -   @Bean 이 스프링 빈
    -   빈 이름은 메서드 이름으로 지정
    -   @Bean(name="")으로 빈 이름 지정 가능 - 설정 클래스 AppConfig
3.  스프링 빈 의존관계 주입
    -   파라미터로 넘어온 설정 클래스 정보를 참고해 의존관계를 주입
    -   생성자를 호출하면서 의존관계 주입이 한 번에 가능

스프링 컨테이너를 만드는 다양한 방법은 **ApplicationContext 인터페이스의 구현체이다.**

-   DependencyConfig.class 등의 구성 정보를 지정하여 스프링 컨테이너를 생성해야 한다.
-   DependencyConfig에 있는 구성 정보를 통해 스프링 컨테이너는 필요한 객체들을 생성한다.
-   애플리케이션 클래스는 구성 메타데이터와 결합되어 ApplicationContext 생성 및 초기화된 후 완전히 구성되고 실행 가능한 시스템 또한 애플리케이션을 갖게 된다.

스프링 빈 조회에서 상속관계가 있을 시 부모타입으로 조회하면 자식 타입도 함께 조회한다.

-   모든 자바 객체의 최고 부모인 object타입으로 조회하면 모든 스프링 빈을 조회한다.

#### 스프링 컨테이너의 종류

파라미터로 넘어온 설정 클래스 정보를 참고해서 빈의 생성, 관계 설정 등의 제어작업을 총괄하는 컨테이너이다.

##### BeanFactory

-   스프링 컨테이너의 최상위 인터페이스이다.
-   BeanFactory는 빈을 등록, 생성, 조회하고 돌려주는 등 빈을 관리하는 역할을 한다.
-   getBean() 메서드를 통해 빈을 인스턴스화할 수 있다.
-   @Bean이 붙은 메서드의 명을 스프링 빈의 이름으로 사용해 빈 등록을 한다.

##### ApplicationContext

-   BeanFactory의 기능을 상속받아 제공한다.
-   빈을 관리하고 검색하는 기능을 BeanFactory가 제공하고, 그 외에 부가기능을 제공한다.
-   부가 기능
    -   MessageSource : 메세지 다국화를 위한 인터페이스
    -   EnvironmentCapable: 개발, 운영 등 환경변수 등으로 나눠 처리하고, 애플리케이션 구동 시 필요한 정보들을 관리하기 위한 인터페이스
    -   ApplicationEventPublisher: 이벤트 관련 기능을 제공하는 인터페이스
    -   ResourceLoader: 파일, 클래스 패스, 외부 등 리소스를 편리하게 조회

#### 컨테이너 인스턴스화

Application 생성자에 제공된 위치 경로 또는 경로는 컨테이너가 로컬 파일 시스템, Java CLASSPATH 등과 같은 다양한 외부 리소스로부터 구성 메타데이터를 로드할 수 있도록 하는 리소스 문자열이다.

```
// Annotation
ApplicationContext context = new AnnotationConfigApplicationContext(DependencyConfig.class);
```