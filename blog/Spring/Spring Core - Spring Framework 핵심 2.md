태그: #Spring
연결문서: [Spring Core - Spring Framework 핵심 3](Spring%20Core%20-%20Spring%20Framework%20핵심%203.md)

# Spring Framework 핵심

## DI(Dependency Injection)

### 빈(Bean)

빈은 스프링 컨테이너에 의해 관리되는 재사용 소프트웨어 컴포넌트이다.  
이는 스프링 컨테이너가 관리하는 자바 객체를 의미하며, 하나 이상의 빈을 관리한다

-   빈(Bean)은 인스턴스화된 객체를 의미한다.
-   스프링 컨테이너에 등록된 객체를 스프링 빈이라고 한다.
-   @Bean이 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다.
-   빈은 클래스의 등록정보, 게터/세터 메서드를 포함한다.
-   빈은 컨테이너에 사용되는 설정 메타데이터로 생성된다.
-   설정 메타데이터
    -   XML 또는 자바 애너테이션, 자바 코드로 표현한다.
    -   컨테이너의 명령과 인스턴스화, 설정, 조립할 객체를 정의한다.

#### 컨테이너의 의미

소프트웨어 개발 용어의 관점에서 컨테이너란 내부에 또 다른 컴포넌트를 가지고 있는 어떤 컴포넌트를 의미한다.

-   컨테이너는 먼저 객체를 생성하고 객체를 서로 연결한다.
-   객체를 설정하는 단계를 지나 마지막으로 생명주기 전반을 관리한다.
-   컨테이너는 객체의 의존성을 확인해 생성한 뒤 적절한 객체에 의존성을 주입한다.
-   스프링은 스프링 컨테이너를 통해 객체를 관리한다.
-   스프링 컨테이너에서 관리되는 객체를 빈(Bean)이라고 한다.

#### BEAN 접근 방법

ApplicationContext를 사용하여 bean 정의를 읽고 액세스할 수 있다.

```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("memberRepository", memberRepository.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

-   getBean을 사용하여 bean의 인스턴스를 가져올 수 있다.
-   ApplicationContext 인터페이스는 bean을 가져오는 몇 가지 방법들이 있다.
-   실제적으로 응용 프로그램 코드에서는 getBean() 메서드로 호출하여 사용하면 안된다.

#### BeanDefinition

스프링은 다양한 설정 형식을 BeanDefinition이라는 추상화 덕분에 지원할 수 있다.

-   Bean은 BeanDefinition(빈 설정 메타정보)으로 정의되고 BeanDefinition에 따라서 활용하는 방법이 달라지게 된다.
-   BeanDefinition(빈 설정 메타정보)
    -   속성에 따라 컨테이너가 Bean을 어떻게 생성하고 관리할 지 결정한다.
    -   @Bean or <bean> 당 각 1개씩 메타 정보가 생성된다.
    -   Spring이 설정 메타정보를 BeanDefinition 인터페이스를 통해 관리하기 때문에 컨테이너 설정을 XML, Java로 할 수 있다.

### 빈 스코프(Bean Scope)

**bean definition이 레시피라는 개념은 중요하다.**  
bean definition을 만들 때 해당 bean definition에 의해 정의된 클래스의 실제 인스턴스를 만들기 위한 레시피를 만든다. (bean definition = recipe)

-   빈이 존재할 수 있는 범위를 의미한다.
-   특정 bean 정의에서 생성된 개체에 연결할 다양한 의존성 및 구성 값뿐만 아니라 특정 bean 정의에서 생성된 개체의 범위도 제어할 수 있다.
-   Spring Framework는 6개의 범위를 지원하며, 그 중 4개는 ApplicationContext를 사용하는 경우에만 사용할 수 있다.
-   bean은 여러 범위 중 하나에 배치되도록 정의할 수 있다.
-   구성을 통해 생성하는 개체의 범위를 선택할 수 있기 때문에 강력하고 유연하다.
-   사용자 정의 범위를 생성할 수도 있다.

|   Scope   |   Description   |
| :-: | :-: |
|   sington   |   (Default) 각 Spring 컨테이너에 대한 단일 객체 인스턴스에 대한 단일 bean definition의 범위를 지정   |
|   prototype   |   스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프   |
|   request   |   웹 요청이 들오오고 나갈 때까지 유지되는 스코프   |
|   session   |   웹 세션이 생성되고 종료될 때까지 유지되는 스코프   |
|   application   |   웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프   |
|   websocket   |   단일 bean definition 범위를 WebSocket의 라이프사이클까지 확장   Spring ApplicationContext의 컨텍스트에서만 유효   |

#### 싱글톤(singleton) 스코프

싱글톤 스코프는 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.

-   스프링 컨테이너의 시작과 함께 생성되어서 스프링 컨테이너가 종료될 때까지 유지된다.
-   싱글톤 빈의 하나의 공유 인스턴스만 관리하게 된다.
    -   private 생성자를 사용해 외부에서 임의로 new를 사용하지 못하도록 막아야 한다.
-   해당 bean definition와 일치하는 ID 또는 ID를 가진 빈에 대한 모든 요청은 스프링 컨테이너에서 해당 특정 빈 인스턴스를 반환한다.
-   스프링 컨테이너 종료 시 소멸 메서드가 자동으로 실행된다.

#### 싱글톤 내용 정리

-   싱글톤은 해당 빈의 인스턴스를 오직 하나만 생성해서 사용하는 것을 의미한다.
-   단일 인스턴스는 싱글톤 빈의 캐시에 저장된다.
-   이름이 정해진 빈에 대한 모든 요청과 참조는 캐시된 개체를 반환한다.
    -   싱글톤 스코프의 스프링 빈은 여러 번 호출해도 모두 같은 인스턴스 참조 주소값을 가진다.

#### 싱글톤 패턴의 문제점

-   싱글톤 패턴을 구현하는 코드 자체가 많다.
-   의존 관계상 클라이언트가 구체 클래스에 의존한다.
-   지정해서 가져오기 때문에 테스트하기 어렵다.
-   private 생성자를 사용하여 자식 클래스를 만들기 어렵기 때문에 유연성이 떨어진다.
-   속성 공유
    -   멀티쓰레드 환경에서 싱글톤 객체의 속성은 여러 쓰레드에 의해 바뀔 수 있다.
    -   A 쓰레드에선 속성 값을 x로 바꾸고 출력하는 과정에서 B 쓰레드가 속성 값을 y로 바꾸면 A 쓰레드에선 예상하지 못한 값이 나올 수 있다.(1개의 인스턴스에서 속성 값을 공유하기 때문)
    -   가급적 읽기만 가능해야 한다.
-   Application 초기 구동 시 인스턴스 생성
    -   싱글톤 빈은 기본적으로 애플리케이션 구동 시 생성되므로 싱글톤 빈이 많을수록 구동 시간이 증가할 수 있다.

**이런 싱글톤 패턴 문제는 싱글톤 컨테이너가 해결해준다.**

-   스프링 컨테이너는 싱글톤 컨테이너 역할을 한다.
-   싱글톤 객체로 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
-   스프링 컨테이너의 위 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하며 객체를 싱글톤으로 유지할 수 있다.

```
@Configuration
public class DependencyConfig {
  @Bean
  public MemberService memberService() {
    return new MemberService(memberRepository());
  }
  @Bean
  public MemberRepository memberRepository() {
    return new MemberRepository();
  }
  @Bean
  public CoffeeService coffeeService() {
    return new CoffeeService(coffeeRepository());
  }
  @Bean
  public CoffeeRepository coffeeRepository() {
    return new CoffeeRepository();
  }
}
```

-   @Configuration과 @Bean 애너테이션을 추가한다.
-   @Bean을 통해 스프링 컨테이너에 등록된다.

#### 싱글톤 방식의 주의점

-   싱글톤 방식은 여러 클라이언트가 하나의 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 무상태로 설계해야 한다.
    -   특정 클라이언트가 값을 변경할 수 있으면 안된다.
    -   읽기만 가능해야 한다.
    -   스프링 빈의 공유 값을 설정하면 장애가 발생할 수 밖에 없다.