태그: #Spring
연결문서: [Spring Core - Spring Framework 핵심 4](Spring%20Core%20-%20Spring%20Framework%20핵심%204.md)

# Spring Framework 핵심

## DI(Dependency Injection)

### Java 기반 컨테이너 설정

자바 기반 설정의 가장 중요 애너테이션 두 가지는 다음과 같다.

-   @Configuration
-   @Bean
-   메서드가 Spring 컨테이너에서 관리할 새 객체를 인스턴스화, 구성 및 초기화한다는 것을 나타내는 데 사용된다.

```
// DependencyConfig 클래스
컨텍스트를 인스턴스화할 때
@Configuration
public class DependencyConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

#### AnnotationConfigApplicationContext를 사용하여 스프링 컨테이너를 인스턴스화 하는 방법

애너테이션을 이용해 Config 클래스를 설정하는 방법이다.

-   Spring 3.0에 도입된 AnnotationConfigApplicationContext이다.
-   ApplicationContext 구현은 아래와 같은 애너테이션이 달린 클래스로 파라미터를 전달 받는다.
    -   @Configuration 클래스
    -   @Component 클래스
    -   JSR-330 메타데이터
-   @Configuration 클래스가 입력으로 제공되면 @Configuration 클래스 자체가 Bean 정의로 등록되고 클래스 내에서 선언된 모든 @Bean 메서드도 Bean 정의로 등록된다.
-   @Component 클래스와 JSR-330 클래스가 제공되면 빈 정의로 등록되며 필요한 경우 해당 클래스 내에서 @Autowired 또는 @Inject와 같은 DI 메타데이터가 사용되는 것으로 가정한다.
-   예제

```
//@Configuration 클래스를 입력으로 사용 (DependencyConfig.class)
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(DependencyConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

```
// @Component 또는 JSR-330 주석이 달린 클래스는 다음과 같이 생성자에 입력으로 사용
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

#### @Bean 애너테이션을 사용하기

@Bean은 메서드-레벨 애너테이션이며, <Bean/>에서 제공하는 일부 속성을 지원한다.

-   init-method
-   destroy-method
-   autowiring
-   @Bean 애너테이션은 @Configuration-annoted 또는 @Component-annoted 클래스에서 사용할 수 있다.

**빈 선언**  
@Bean 애너테이션을 메서드에 추가해서 Bean으로 정의(선언)할 수 있다.

-   애너테이션 방식의 configuration

```
@Configuration
public class DependencyConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

-   XML 방식의 configuration

```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

-   빈 정의가 있는 인터페이스를 구현하여 bean configuration을 설정할 수 있다.

```
public interface BaseConfig {

    @Bean
    default TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}

@Configuration
public class DependencyConfig implements BaseConfig {

}
```

**빈 의존성**  
@Bean 애너테이션이 추가된(@Bean-annotated) 메서드는 빈을 구축하는데 필요한 의존성을 나타내는데 매개 변수를 사용할 수 있다.

```
@Configuration
public class DependencyConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

#### @Configuration 애너테이션을 사용하기

-   @Configuration는 해당 객체가 bean definitions의 소스임을 나타내는 애너테이션이다.
-   @Configuration는 @Bean-annoted 메서드를 통해 bean을 선언한다.
-   @Configuration 클래스의 @Bean 메서드에 대한 호출을 사용하여 bean 사이의 의존성을 정의할 수 있다.

**Bean 사이에 의존성 주입**  
빈이 서로 의존성을 가질 때, 의존성을 표현하는 것은 다른 bean 메서드를 호출하는 것처럼 간단하다.

```
// beanOne은 생성자 주입을 통해 beanTwo에 대한 참조를 받는다
@Configuration
public class DependencyConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

**Java를 기반으로 설정되어 있는 환경에서 내부적으로 작동하는 방식에 대한 정보**

```
@Configuration
public class DependencyConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

-   clientDao() 메서드는 clientService1()과 clientService2() 메서드에서 1번씩 호출되었다.
-   이 메서드는 ClientDaoImpl의 새 인스턴스를 만들고 이를 반환하므로 일반적으로 두 개의 인스턴스(각 서비스마다 하나씩)가 있어야 한다.
    -   문제점 - Spring에 인스턴스화된 빈은 기본적으로 싱글톤 범위를 갖게 된다.
-   하위 클래스의 하위 메서드는 상위 메서드를 호출하고 **새 인스턴스를 만들기 전에 먼저 컨테이너에 캐시된(범위 지정) bean이 있는지 확인**한다.
-   확인 사항
    -   모든 @Configuration 클래스는 시작 시 CGLIB를 사용하여 하위클래스로 분류된다.
    -   스프링에서 CGLIB라는 바이트코드 조작 라이브러리를 사용하는 것이다.
    -   AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록하는 것이다.(DependencyConfig -> DependencyConfig@CGLIB)
    -   같은 메서드를 호출할 때 이미 스프링 컨테이너에 등록되어 있으면 기존에 만들어진 걸 반환한다.
    -   스프링 컨테이너에 등록되어 있지 않을 때에만 필요한 Bean을 생성하고 스프링 컨테이너에 등록한다.

#### Java 코드에서 애너테이션을 사용해 Spring 컨테이너를 구성하는 방법

Spring의 자바 기반 구성 기능 특징인 애너테이션을 사용해 구성을 복잡성을 줄일 수 있다.

**@Import 애너테이션**

-   XML 파일 내에서 요소가 사용되는 것처럼 구성을 모듈화하는데 사용된다.
-   다른 구성 클래스에서 @Bean definitions를 가져올 수 있다.

```
@Configuration
public class DependencyConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(DependencyConfigA.class)
public class DependencyConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

-   컨텍스트를 인스턴스화할 때 DependencyConfigA.class와 DependencyConfigB.class 모두 지정하는 대신 DependencyConfigB만 제공하면 된다.

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(DependencyConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

-   ctx에 Import(DependencyConfigA.class) 받은 DependencyConfigB.class 사용으로 인해 ctx.getBean(A.class)가 가능해진다.
-   컨테이너 인스턴스화를 단순화할 수 있다.
    -   많은 @Configuration 클래스를 기억할 필요 없이 하나의 클래스만 처리하면 된다.

**추가한 @Bean 애너테이션에서 의존성 주입**

**문제점**

-   실제로 사용될 때 빈은 1개의 구성 파일만 @Import받지 않고 여러 구성 클래스 간에 걸쳐 서로 의존성을 갖는다.
-   **XML을 사용할 때는 컴파일러가 관여하지 않아** 컨테이너 초기화 중에 ref=”some Bean”을 선언하여 스프링으로 해결할 수 있어 **문제가 되지 않는다.**
-   @Configuration 클래스를 사용할 때, 자바 컴파일러는 구성 모델에 제약을 두며, 다른 빈에 대한 참조는 유효한 자바 구문이어야 한다.

```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

**해결 방법**

-   @Bean 메서드는 빈 의존성을 설명하는 임의 개수 파라미터를 가질 수 있다.
-   @Autowired 및 @Value 주입 및 다른 bean과 동일한 기능을 사용할 수 있다.
-   @Configuration의 생성자 주입은 스프링 프레임워크 4.3에서만 지원된다.
-   대상 빈이 하나의 생성자만 정의하는 경우 @Autowired 지정할 필요가 없다.

```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```