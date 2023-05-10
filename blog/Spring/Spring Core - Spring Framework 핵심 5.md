태그: #Spring
연결문서: [Spring MVC  아키텍처](Spring%20MVC%20%20아키텍처.md)

# Spring Framework 핵심

## AOP(Aspect Oriented Programming)

### AOP란?

AOP는 Spring 프레임워크의 핵심 프로그래밍 모델 중 하나인 관점 지향 프로그래밍을 뜻한다.  
  

AOP는 애스펙트를 사용하여 다양한 기능들을 분리한다.  
학심 기능과 부가 기능을 분리하고 분리된 부가 기능을 어디에 적용할 지 선택하는 기능들을 만들었다.

-   애스펙트(Aspect)  
    부가 기능과 해당 부가 기능을 어디에 적용할 지 정의한 것으로, 분리한 부가 기능과 그 기능들을 어디에 적용할 지 선택하는 기능을 합해서 하나의 모듈로 만든 것이다.

애스펙트는 관점이라는 뜻을 의미한다.  
애플리케이션을 바라보는 관점을 하나 하나의 기능(횡단 관심사, cross-cutting concerns) 관점으로 보는 것이다.  
  

**AOP는 기존에 사용하던 OOP를 대체하기 위한 것이 아닌 횡단 관심사를 깔끔하게 처리하기 위해 OOP의 부족한 부분을 보조하는 목적으로 개발되었다.**

---

### AOP가 필요한 이유

소프트웨어 개발에서 변경 지점은 하나가 될 수 있도록 잘 모듈화 되어야 한다.  
부가 기능처럼 특정 로직을 애플리케이션 전반에 적용하는 문제는 일반적인 OOP 방식으로는 해결이 어렵기 때문에 핵심 기능과 부가 기능을 분리하는 AOP 방식이 필요하다.

#### 필수 개념

-   핵심 기능(Core Concerns) : 업무 로직을 포함하는 기능
-   부가 기능(Cross-Cutting Concerns) : 핵심 기능을 도와주는 부가적인 시능
    -   로깅, 보안, 트랜잭션 등이 있다.
-   Aspect : 부가 기능을 정의한 코드인 어드바이스(Advice)와 어드바이스를 어디에 적용할 지 결정하는 포인트컷(PointCut)을 합친 개념이다.(Advice + PointCut = Aspect)

#### 객체 지향 프로그래밍(Object Oriented Programming:OOP)

지금까지 많은 프로젝트가 OOP 패러다임을 지향하며 프로그래밍을 하고 있다.  
정의된 기능들을 재사용하기 위해 동작보다는 객체를 중심으로 프로그래밍하는 OOP가 등장했다.

-   OOP의 핵심은 공통된 목적은 띈 데이터와 동작을 묶어 하나의 객체로 정의하는 것이다.
-   객체를 적극적으로 활용함으로써 기능을 재사용할 수 있는 것이 큰 장점이다.
-   객체를 잘 활용하기 위해선 관심사 분리(Separation of Concerns:SoC)의 디자인 원칙을 준수해야 한다.

#### 객체 지향 프로그래밍의 주요 개념

-   Spring MVC 구조의 @Controller, @Service, @Repository와 같이 관심사 별로 계층을 나눠 객체를 관리한다.
-   **관심사의 분리는 모듈화의 핵심이다.**
-   문제점
    -   특정 관심사 업무 코드에 트랜잭션, 보안, 로깅 등의 코드가 존재하게 된다.
    -   트랜잭션, 보안, 로깅 코드는 업무와는 관련이 없지만 애플리케이션에 필수적인 부가 기능이다.
    -   트랜잭션, 보안, 로깅은 불특정 다수의 클래스에서 존재하게 된다.
    -   관심사 관점에서는 트랜잭션, 보안, 로깅 코드들을 횡단 관심사(부가 기능)라고 한다.
    -   업무 관련 코드는 핵심 관심사(핵심 기능)라고 한다.
    -   **비즈니스 클래스에 횡단 관심사와 핵심 관심사가 공존하게 된다.**
        -   메서드 복잡도가 증가하여 비즈니스 코드 파악이 어렵게 된다.
        -   부가 기능의 불특정 다수 메서드가 반복적으로 구현되어 횡단 관심사의 모듈화가 어렵다.

#### AOP의 등장

OOP만 사용해선 횡단 관심사 코드를 깔끔하게 분리하고, 비즈니스 코드에 적용하기 어려웠다. 이런 OOP의 관심사 분리에 대한 한계적인 부분을 해결하고자 AOP가 등장하게 된다.  
  

**AOP(Aspect-Oriented Programming)**는 기존과 다른 프로그램 구조 사고 방식을 제공함으로써 객체 지향 프로그래밍의 부족한 부분을 보완한다.  
  

OOP의 모듈화의 핵심 단위는 클래스이고, AOP의 모듈화의 핵심 단위는 관점이다.  
Aspect는 여러 유형과 객체 간에 발생하는 문제(Ex.트랜잭션)의 모듈화를 가능하게 한다.  
  

애플리케이션 로직은 크게 **핵심 기능과 부가 기능**으로 나눌 수 있다.

-   핵심 기능(Core Concerns)
    -   객체가 제공하는 고유의 기능(업무 로직 등을 포함)이다.
-   부가 기능(Cross-Cutting Concerns)
    -   핵심 기능을 보조하기 위해 제공되는 기능이다.
    -   로그 추적 로직, 보안, 트랜잭션 기능 등이 있다.
    -   단독으로 사용되지 않고 핵심 기능과 함께 사용된다.

부가 기능은 보통 여러 클래스에 걸쳐서 함께 사용된다.

-   부가 기능을 여러 곳에 적용하려면 중복 코드가 발생하게 된다.
-   부가 기능에 수정이 필요하게 되면 사용되는 모든 클래스를 하나씩 수정해야 한다.

---

### AOP 용어

#### 애스펙트(Aspect)

-   여러 객체에 공통으로 적용되는 기능을 말한다.
-   어드바이스 + 포인트컷을 모듈화하여 애플리케이션에 포함되는 횡단 기능이다.
-   여러 어드바이스와 포인트컷이 함께 존재한다.

#### 조인 포인트(Join Point)

-   클래스 초기화, 객체 인스턴스화, 메서드 호출, 필드 접근, 예외 발생과 같은 애플리케이션 실행 흐름에서의 특정 포인트를 의미한다.
-   애플리케이션에 새로운 동작을 추가하기 위해 조인포인트에 **관심 코드**를 추가할 수 있다.
-   횡단 관심은 조인포인트 전/후에 AOP에 의해 자동으로 추가된다.
-   추상적인 개념이고 AOP를 적용할 수 있는 모든 지점이라고 생각하면 된다.
-   Spring AOP는 프록시 방식을 사용하므로 조인 포인트는 항상 메서드 실행 지점으로 제한된다.
-   어드바이스 적용이 필요한 곳은 애플리케이션 내에 메서드를 갖는다.

#### 어드바이스(Advice)

-   조인포인트에서 수행되는 코드를 의미한다.
-   Aspect를 언제 핵심 코드에 적용할 지를 정의한다.
-   시스템 전체 애스펙트에 API 호출을 제공한다.
-   메서드를 호출하기 전에 각 상세 정보와 모든 메서드를 로그로 남기기 위해 메서드 시작 전의 포인트인 조인포인트 S를 선택한다.
-   부가 기능에 해당된다.

#### 포인트컷(PointCut)

-   조인 포인트 중에서 어드바이스가 적용될 위치를 선별하는 기능이다.
-   AspectJ 표현식을 사용해서 지정한다.
-   프록시를 사용하는 Spring AOP는 메서드 실행 지점만 포인트컷으로 선별 가능하다.

#### 위빙(Weaving)

-   포인트컷으로 결정한 타겟의 조인 포인트에 어드바이스를 적용하는 것이다.
    -   어드바이스를 핵심 코드에 적용하는 것을 의미한다.
-   핵심 기능 코드에 영향을 주지 않고 부가 기능을 추가할 수 있다.
-   AOP 적용을 위해 애스펙트 객체에 연결한 상태이다.
    -   컴파일 타임(AspectJ compiler)
    -   로드 타임
    -   런타임, Spring AOP는 런타임, 프록시 방식이다.

#### AOP 프록시(Proxy)

-   AOP 기능을 구현하기 위해 만든 프록시 객체이다.
-   스프링에서 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시이다.

#### 타겟(Target)

-   핵심 기능을 담고 있는 모듈로 타겟은 부가 기능을 부여할 대상이 된다.
-   어드바이스를 받는 개체이고 포인트컷으로 결정된다.

#### 어드바이저(Advisor)

-   하나의 어드바이스와 하나의 포인트컷으로 구성된다.
-   Spring AOP에서만 사용되는 특별한 용어이다.

---

### 타입별 Advice

#### 어드바이스(Advice)

-   Aspect를 언제 핵심 코드에 적용할 지를 정의한다.
-   부가 기능에 해당된다.
-   특정 조인 포인트에서 애스펙트에 의해 취해지는 조치이다.

#### Advice 순서

어드바이스는 기본적으로 순서를 보장하지 않는다.

-   순서를 지정하고 싶으면 @Aspect 적용 단위 org.springframework.core.annotation.@Order 애너테이션을 적용해야 한다.
    -   **어드바이스 단위가 아니라 클래스 단위로 적용할 수 있다.**
    -   하나의 애스펙트에 여러 어드바이스가 존재하면 순서를 보장 받을 수 없다.
-   **애스펙트를 별도의 클래스로 분리해야 한다.**

#### Advice 종류

-   **Before**
    -   조인 포인트 실행 이전에 실행한다.
    -   타겟 메서드가 실행되기 전에 처리해야할 필요가 있는 부가 기능을 메서드 호출 전에 실행한다.
    -   Before Advice를 구현한 메서드는 일반적으로 리턴 타입이 void이다.
        -   리턴 값을 갖더라도 실제 Advice 적용 과정에 아무 영향이 없다.
    -   메서드에서 예외를 발생시킬 경우 대상 객체의 메서드가 호출되지 않게 된다.
    -   작업 흐름을 변경할 수 없다.
    -   메서드 종료 시 자동으로 다음 타겟이 호출된다.(예외가 발생하면 다음 코드는 호출되지 않는다.)

```
@Before("hello.aop.order.aop.Pointcuts.orderAndService()")
public void doBefore(JoinPoint joinPoint) {
    log.info("[before] {}", joinPoint.getSignature());
}
```

-   After returning
    -   조인 포인트가 정상 완료 후 실행한다.
    -   메서드가 예외 없이 실행된 이후에 공통 기능을 실행한다.
    -   메서드 실행이 정상적으로 반환될 때 실행된다.
    -   returning 속성에 사용된 이름은 어드바이스 메서드의 매개변수 이름과 일치해야 한다.
    -   returning 절에 지정된 타입의 값을 반환하는 메서드만 대상을 실행한다.

```
@AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")
public void doReturn(JoinPoint joinPoint, Object result) {
    log.info("[return] {} return={}", joinPoint.getSignature(), result);
}
```

-   After throwing
    -   메서드가 예외를 던지는 경우에 실행한다.
    -   메서드를 실행하는 도중 예외가 발생한 경우 공통 기능을 실행한다.
    -   메서드 실행이 예외를 던져서 종료될 때 실행한다.
    -   throwing 속성에 사용된 이름은 어드바이스 메서드의 매개변수 이름과 일치해야 한다.
    -   throwing 절에 지정된 타입과 맞는 예외를 대상으로 실행한다.

```
@AfterThrowing(value = "hello.aop.order.aop.Pointcuts.orderAndService()", throwing = "ex")
public void doThrowing(JoinPoint joinPoint, Exception ex) {
    log.info("[ex] {} message={}", joinPoint.getSignature(), ex.getMessage());
}
```

-   After(finally)
    -   조인 포인트의 동작(정상 또는 예외)과는 상관없이 실행한다.
        -   예외 동작의 finally를 생각하면 된다.
    -   메서드 실행 후 공통 기능을 실행한다.
    -   일반적으로 리소스를 해제하는데 사용한다.
-   Around
    -   메서드 호출 전후에 수행하며 가장 강력한 어드바이스이다.
        -   조인 포인트 실행 여부 선택, 반환 값 변환, 예외 변환 등이 가능하다.
        -   조인 포인트 실행 여부 선택 - joinPoint.proceed()
        -   전달 값 변환 - joinPoint.proceed(args\[\])
        -   반환 값 변환
        -   예외 변환
            -   try ~ catch ~ finally가 들어가는 구문 처리가 가능하다.
    -   메서드 실행 전/후 ,예외 발생 시점에 공통 기능을 실행한다.
    -   어드바이스의 첫 번째 파라미터는 ProceedingJoinPoint를 사용해야 한다.
    -   proceed()를 통해 대상을 실행한다.
    -   proceed()를 여러 번 실행할 수 있다.

---

### PointCut 표현식

#### 포인트컷과 표현식 & 지시자

포인트컷은 관심 조인 포인트를 결정하므로 어드바이스가 실행되는 시기를 제어할 수 있다.  
AspectJ는 포인트컷을 편리하게 표현하기 위한 특별한 표현식을 제공한다.

#### 포인트컷 지시자

포인트컷 표현식은 execution 같은 포인트컷 지시자(PointCut Designator:PCD)로 시작한다.

-   포인트컷 지시자 종류

| 종류 | 설명 |
| :-: | :-: |
| execution | 메서드 실행 조인트 포인트를 매칭한다.   Spring AOP에서 가장 많이 사용하며, 기능도 복잡하다. |
| within | 특정 타입 내의 조인 포인트를 매칭한다. |
| args | 인자가 주어진 타입의 인스턴스인 조인 포인트 |
| this | 스프링 빈 객체(Spring AOP 프록시)를 대상으로 하는 조인 포인트 |
| target | Target 객체(Spring AOP 프록시가 가리키는 실제 대상)를 대상으로 하는 조인 포인트 |
| @target | 실행 객체의 클래스에 주어진 타입의 애너테이션이 있는 조인 포인트 |
| @within | 주어진 애너테이션이 있는 타입 내 조인 포인트 |
| @annotation | 메서드가 주어진 애너테이션을 가지고 있는 조인 포인트를 매칭 |
| @args | 전달된 실제 인수의 런타임 타입이 주어진 타입의 애너테이션을 갖는 조인 포인트 |
| bean | 스프링 전용 포인트컷 지시자이며, 빈의 이름으로 포인트컷을 지정한다. |

#### Poringcut 표현식 결합

포인트컷 표현식은 &&, ||, !를 사용하여 결합할 수 있다.  
이름으로 pointcut 표현식을 참조할 수도 있다.

```
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {} // (1)

@Pointcut("within(com.xyz.myapp.trading..*)")
private void inTrading() {} // (2)

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {} // (3)
```

1.  anyPublicOperation은 메서드 실행 조인 포인트가 공용 메서드의 실행을 나타내는 경우 일치한다.
2.  in Trading 메서드 실행이 거래 모듈에 있는 경우에 일치한다.
3.  tradingOperation은 메서드 실행이 거래 모듈의 공개 메서드를 나타내는 경우 일치한다.

#### 일반적인 pointcut 표현식

-   모든 공개 메서드 실행
    -   `execution(public * *(..))`

-   set 다음 이름으로 시작하는 모든 메서드 실행
    -   `execution(* set*(..))`

-   AccountService 인터페이스에 의해 정의된 모든 메소드의 실행
    -   `execution(* com.xyz.service.AccountService.*(..))`

-   service 패키지에 정의된 메서드 실행
    -   `execution(* com.xyz.service.*.*(..))`

-   서비스 패키지 또는 해당 하위 패키지 중 하나에 정의된 메서드 실행
    -   `execution(* com.xyz.service..*.*(..))`

-   서비스 패키지 내의 모든 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `within(com.xyz.service.*)`

-   서비스 패키지 또는 하위 패키지 중 하나 내의 모든 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `within(com.xyz.service..*)`

-   AccountService 프록시가 인터페이스를 구현하는 모든 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `this(com.xyz.service.AccountService)`

-   AccountService 대상 객체가 인터페이스를 구현하는 모든 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `target(com.xyz.service.AccountService)`

-   단일 매개변수를 사용하고 런타임에 전달된 인수가 Serializable과 같은 모든 조인 포인트 (Spring AOP에서만 메소드 실행)
    -   `args(java.io.Serializable)`

-   대상 객체에 @Transactional 애너테이션이 있는 모든 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `@target(org.springframework.transaction.annotation.Transactional)`

-   실행 메서드에 @Transactional 애너테이션이 있는 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `@annotation(org.springframework.transaction.annotation.Transactional)`

-   단일 매개 변수를 사용하고 전달된 인수의 런타임 유형이 @Classified 애너테이션을 갖는 조인 포인트(Spring AOP에서만 메서드 실행)
    -   `@args(com.xyz.security.Classified)`

-   tradeService 라는 이름을 가진 스프링 빈의 모든 조인 포인트 (Spring AOP에서만 메서드 실행)
    -   `bean(tradeService)`

-   와일드 표현식 \*Service 라는 이름을 가진 스프링 빈의 모든 조인 포인트
    -   `bean(*Service)`

---

### JoinPoint

#### AOP 적용 위치

-   AOP는 메서드 실행 위치 뿐만 아니라 다양한 위치에 적용할 수 있다.
    -   적용 가능 지점(조인 포인트) : 생성자, 필드 값 접근, static 메서드 접근, 메서드 실행
-   AOP를 수행하는 메서드는 JoinPoint 인스턴스를 인자로 받게 된다.
-   JoinPoint 인스턴스에서 조인 포인트 지점의 정보를 얻어내야 한다.

#### JoinPoint

조인포인트는 추상적인 개념이고 AOP를 적용할 수 있는 지점을 의미한다.

-   어드바이스가 적용될 수 있는 위치, 메서드 실행, 생성자 호출, 필드 값 접근, static 메서드 접근 같은 프로그램 실행 중 지점을 나타낸다.
-   AspectJ를 사용해서 컴파일 시점과 클래스 로딩 시점에 적용하는 AOP는 바이트코드를 실제 조작하기 때문에 해당 기능을 모든 지점에 다 적용할 수 있다.
-   프록시 방식을 사용하는 스프링 AOP는 메서드 실행 지점에만 AOP를 적용할 수 있다.
-   프록시는 메서드 오버라이딩 개념으로 동작한다.
-   생성자나 static 메서드, 필드 값 접근에는 프록시 개념이 적용될 수 없다.
-   프록시를 사용하는 Spring AOP의 조인 포인트는 메서드 실행으로 제한된다.
-   프록시 방식을 사용하는 Spring AOP는 스프링 컨테이너가 관리할 수 있는 스프링 빈에만 AOP를 적용할 수 있다.
-   JoinPoint 메서드는 어드바이스의 종류에 따라 사용 방법이 다소 다르지만 기본적으로 어드바이스 메서드에 매개변수로 선언만 하면 된다.

#### JoinPoint 인터페이스의 주요 기능

-   JoinPoint.getArgs() : JoinPoint에 전달된 인자를 배열로 반환한다.
-   JoinPoint.getThis() : AOP 프록시 객체를 반환한다.
-   JoinPoint.getTarget() : AOP가 적용된 대상 객체를 반환한다.
    -   클라이언트가 호출한 비즈니스 메서드를 포함하는 비즈니스 객체를 반환한다.
-   JoinPoint.getSignature() : 조언되는 메서드에 대한 설명을 반환한다.
    -   클라이언트가 호출한 메소드의 시그니처(리턴타입, 이름, 매개변수) 정보가 저장된 Signature 객체를 반환한다.
    -   Signature
        -   객체가 선언하는 모든 메서드에서 메서드의 이름, 매개변수를 담고 있는 객체들을 시그니처라고 한다.
    -   Signature가 제공하는 메서드
        -   String getName() : 클라이언트가 호출한 메소드의 이름을 반환한다.
            -   String toLongString() : 클라리언트가 호출한 메소드의 리턴타입, 이름, 매개변수를 패키지 경로까지 포함해서 반환한다.
        -   String toShortString() : 클라이언트가 호출한 메소드 시그니처를 축약한 문자열로 반환한다.
-   JoinPoint.toString() : 조언되는 방법에 대한 유용한 설명을 인쇄한다.

#### ProceedingJoinPoint 인터페이스의 주요 기능

-   proceed() : 다음 어드바이스나 타켓을 호출한다.

---

### 애너테이션(Annotation)을 이용한 AOP

#### Spring에서의 AOP

AOP는 Spring IoC를 보완하여 매우 강력한 미들웨어 솔루션을 제공한다.

-   @AspectJ 애너테이션 스타일
-   스키마 기반 접근

#### @AspectJ 지원

@AspectJ는 애너테이션이 있는 일반 Java 클래스로 관점을 선언하는 스타일을 말한다.

-   @AspectJ 스타일을 AspcetJ 5 릴리스의 일부로 AspectJ 프로젝트에 의해 도입되었다.
-   스프링은 pointcut 구문 분석 및 일치를 위해 AspectJ가 제공하는 라이브러리를 사용하여 AspectJ 5와 동일한 애너테이션을 해석한다.
-   **AOP 런타임은 여전히 순수한 Spring AOP**이며, AspectJ 컴파일러나 위버에 의존하지 않는다.

-   Spring 설정에서 @AspectJ aspect를 사용하기 위해서는 @AspectJ aspect에 기반한 Spring AOP 설정과 이러한 aspect에 의해 조언되는 자동 프록시 빈에 대한 Spring 지원을 활성화해야 한다.
-   @AspectJ 지원은 XML 또는 Java 스타일 설정으로 활성화할 수 있다.
-   Java 설정으로 @AspectJ 지원 활성화 방법
    -   @Configuration으로 @AspectJ 지원을 활성화하려면 @EnableAspectJAutoProxy 애너테이션을 추가한다.

```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

-   XML 설정으로 @AspectJ 지원 활성화 방법
    -   XML 기반 구성으로 @AspectJ 지원을 활성화하려면 aop:aspectj-autoproxy 요소를 사용한다.

```
<aop:aspectj-autoproxy/>
```

#### Aspect 선언

@AspectJ 지원이 활성화되면 @AspectJ 관점(@Aspect 애너테이션이 있음)이 있는 클래스로 애플리케이션 컨텍스트에 정의된 모든 빈이 Spring에서 자동으로 감지되고 Spring AOP를 구성하는 데 사용된다.

```
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

#### 포인트컷 선언

-   포인트컷은 관심 조인 포인트를 결정하므로 **어드바이스가 실행되는 시기를 제어**할 수 있다.
-   Spring AOP는 Spring Bean에 대한 메서드 실행 조인 포인트만 지원하므로 Pointcut은 Spring Bean의 메서드 실행과 일치하는 것으로 생각할 수 있다.
-   Pointcut 선언은 이름과 매개변수를 포함하는 서명과 우리가 관심 있는 메서드 실행을 정확히 결정하는 Pointcut 표현식으로 **두 부분으로 구성**된다.
-   Pointcut 표현식은 @Pointcut 애너테이션을 사용하여 표시된다.

#### 어드바이스 선언

-   어드바이스는 포인트컷 표현식과 연관되며 **포인트컷과 일치하는 메서드 실행 전 또는 메서드 실행 이후에 실행**된다.
-   Pointcut 표현식은 명명된 Pointcut에 대한 단순 참조이거나 제자리에 선언된 Pointcut 표현식일 수 있다.