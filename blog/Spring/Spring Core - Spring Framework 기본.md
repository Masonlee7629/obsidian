태그: #Spring
연결문서: [Spring Core - Spring Framework 핵심 1](Spring%20Core%20-%20Spring%20Framework%20핵심%201.md)

# Spring Framework 기본

## Spring Framework 소개

### Framework

> "소프트웨어의 구체적인 부분에 해당하는 설꼐와 구현을 재사용이 가능하게끔 일련의 협업화된 형태로 클래스들을 제공하는 것" -Ralph Johnson-

Frame은 어떤 대상의 큰 틀이나 외형적인 구조를 의미하며, 프로그래밍 세계에서의 Frame 역시 이와 비슷한 의미를 가지고 있다.  
단어의 대로 소프트웨어 관점에서의 Framework는 우리가 어떤 애플리케이션을 만들기 위한 틀 혹은 구조를 제공한다고 생가하면 된다.

#### Framework의 장점

-   효율적으로 코드를 작성할 수 있다.  
    개발하고자 하는 애플리케이션을 전부 개발하는 것이 아니라 서로 다른 애플리케이션 간의 통신이나, 데이터를 데이터 저장소에 저장하는 등의 다양한 기능들 역시 Framework가 라이브러리 형태로 제공함으로써 **개발자가 애플리케이션의 핵심 로직을 개발하는 것에 집중할 수 있도록** 해준다.
-   정해진 규약이 있어 애플리케이션을 효율적으로 관리할 수 있다.  
    Framework의 규약에 맞게 코드를 작성하기 떄문에 유지보수가 필요한 경우 더 빠르고 쉽게 문제점을 파악하여 수정할 수 있다. 동시에 내가 작업했던 코드를 다른 사람이 수정할 경우에도 이미 Framework의 규약에 맞게 작성된 코드이기 때문에, 빠르게 코드를 파악하고 수정하기 용이하다. 이는 유지보수 이외에도 비슷한 기능을 개발할 때 코드의 재사용이 용이하고 기능의 확장 또한 쉽게 가능하다.

#### Framework의 단점

-   사용하고자 하는 Framework에 대한 학습이 필요하다.  
    Framework에서 정하는 규약들을 학습할 시간이 필요하다는 의미이다. Spring의 경우 Java언어에 대한 이해가 필요하지만 추가로 Spring이라는 Framework에 대한 학습이 필요하다.
-   자유롭고 유연한 개발이 어렵다.  
    사용하는 Framework의 규약을 벗어나기 어렵다. 이미 만들어진 애플리케이션에서 Framework를 변경하거나, 유연한 개발을 위해 Framework를 사용하지 않게 변경할 경우 많은 시간과 노력이 필요하다.

---

### Framework와 Library의 차이

Library는 애플리케이션을 개발하는 데 사용되는 일련의 데이터 및 프로그래밍 코드이다. 소프트웨어 관점에서 애플리케이션을 개발할 때 필요한 기능을 미리 구현해놓은 집합체라고 생각할 수 있다.  
  

Framework는 전체적인 구조 또는 뼈대를 의미하고, Library는 다양한 기능을 제공하는 부품을 의미한다.  
  

따라서 한번 정해진 Framework를 교체하는 일은 어렵지만, Library는 쉽게 교체가 가능하며 필요한 Library들을 선택적으로 사용할 수 있다.  
  

이는 **애플리케이션에 대한 제어권이 차이가 있다**고 표현할 수 있다.  
  

개발자가 짜놓은 코드 내에서 필요한 기능이 있으면 해당 Library를 호출해서 사용할 수 있으며, 이는 **애플리케이션 흐름의 주도권이 개발자에게 있는 것**이다.  
  

반면, Framework는 개발자가 코드를 작성하면, 작성한 코드를 사용하여 Framework에서 애플리케이션의 흐름을 만들어내고, 이는 **애플리케이션 흐름의 주도권이 Framework에 있는 것**이다.

---

### Spring Framework

Spring Framework는 Java 플랫폼을 위한 오픈 소스 애플리케이션 프레임워크이다.  
  

웹 애플리케이션 개발을 위한 Framework에는 Spring 뿐만 아니라, Django, Express, Flask, Lalavel 등 다양한 Framework를 통해 개발이 가능하며 각각 Framework마다 사용하는 언어도 다르고 개발 방법도 조금씩 달라진다.

#### Spring Framework의 장점

1.  POJO(Plain Old Java Object)기반의 구성
2.  DI(Dependency Injection) 지원
3.  AOP(Aspect Oriented Programming, 관점지향 프로그래밍) 지원
4.  Java 언어를 사용함으로써 얻는 장점
    -   정적 타입 언어로써 변수의 타입, 메서드의 입력과 출력이 어떤 타입을 가져야 하는지를 강제함
        -   협업 시 코드의 수정, 보완이 용이하며 웹 서버를 구축할 때 런타임 에러를 사전에 방지 가능

---

## Spring Framework의 특징

### POJO(Plain Old Java Object)

[##_Image|kage@byC8pY/btsajeow9ti/Y9yCAA5XFH00cWrt2EFuQK/img.png|CDM|1.3|{"originWidth":594,"originHeight":476,"style":"alignCenter","width":null}_##]

위 그림에서 POJO는 Spring에서 사용하는 핵심 개념들에 둘러 싸인 모습으로, IoC/DI, AOP, PSA를 통해서 POJO를 달성할 수 있다는 것을 의미한다.  
  

POJO는 순수한 Java 객체를 의미한다.  
  

프로그래밍 관점에서 POJO는 Java로 생성하는 순수한 객체로 프로그래밍 코드를 작성하는 것을 의미하며, POJO 프로그래밍으로 작성된 코드라고 불리기 위해서는 크게 두 가지 정도의 기본적인 규칙을 지켜야 한다.

-   Java나 Java의 스펙(사양)에 정의된 것 이외에는 다른 기술이나 규약에 얽매이지 않아야 한다.
-   특정 환경에 종속적이지 않아야 한다.

#### POJO 프로그래밍이 필요한 이유

-   특정 환경이나 기술에 종속적이지 않으면 재사용이 가능하고, 확장 가능한 유연한 코드를 작성할 수 있다.
-   저수준 레벨의 기술과 환경에 종속적인 코드를 애플리케이션 코드에서 제거함으로써 코드가 깔끔해진다.
-   코드가 깔끔해지기 때문에 디버깅하기 상대적으로 쉽다.
-   특정 기술이나 환경에 종속적이지 않기 때문에 테스트가 단순해진다.
-   **객체지향적인 설계를 제한없이 적용할 수 있다.**

#### POJO와 Spring의 관계

Spring은 POJO 프로그래밍을 지향하는 Framework이다.  
  

Spring은 최대한 다른 환경이나 기술에 종속적이지 않도록 하기 위한 POJO 프로그래밍 코드를 작성하기 위해 **IoC/DI, AOP, PSA**라는 기술을 지원하고 있다.

---

### IoC(Inversion of Control)

Framework와 Library의 차이점에서 애플리케이션 흐름의 주도권을 언급했었다.  
  

IoC(제어의 역전)는 **애플리케이션 흐름의 주도권이 뒤바뀐 것이며, 주도권을 Spring이 가지는 것**을 의미한다.  
  

일반적으로 Java 콘솔 애플리케이션을 실행하려면 main() 메서드가 있어야 한다. 그리고 main() 메서드 안에서 다른 객체의 메서드를 호출하는 등의 과정을 거쳐 프로세스가 진행되며, 이것은 개발자가 작성한 코드를 순차적으로 실행하는 애플리케이션의 일반적인 제어 흐름이다.  
  

[##_Image|kage@dgpiyc/btsar5qfAry/ZxZGQrVCGUal3kqcA2v7yK/img.png|CDM|1.3|{"originWidth":830,"originHeight":478,"style":"alignCenter","width":null}_##]

위 그림은 서블릿 기반의 애플리케이션을 웹에서 실행하기 위한 서블릿 컨테이너의 모습이다.  
  

Java 콘솔 애플리케이션의 경우 main() 메서드가 종료되면 애플리케이션의 실행이 종료된다.  
  

하지만 웹에서 동작하는 애플리케이션의 경우 클라이언트가 외부에서 접속해 사용하는 서비스이기 때문에 main() 메서드가 종료되지 않아야 할 것이다.  
  

그러나 서블릿 컨테이너에는 서블릿 사양(Specification)에 맞게 작성된 서블릿 클래스만 존재하고, 별도의 main() 메서드가 존재하지 않는다.  
  

이는 서블릿 컨테이너는 클라이언트의 요청이 들어올 때마다 서블릿 컨테이너 내의 컨테이너 로직(service() 메서드)이 서블릿을 직접 실행시켜 주기 때문이다.  
  

이 경우, 서블릿 컨테이너가 서블릿을 제어하고 있기 때문에 애플리케이션의 주도권은 서블릿 컨테이너에 있고, 서블릿과 웹 애플리케이션 간에 **IoC의 개념이 적용**되어 있는 것이다.

Spring에는 **DI를 통해 IoC의 개념이 적용**된다.

---

### DI(Dependency Injection)

DI는 IoC의 개념을 구체화시킨 것으로 볼 수 있으며 의존성 주입이라고도 부른다.  
  

**'의존성 주입'**이란 생성자를 통해 어떤 클래스의 객체를 전달받는 것을 의미한다.  
생성자 파라미터로 객체를 전달하는 것을 외부에서 객츠를 주입한다고 표현하는 것이다.  
  

즉, 클래스의 생성자로 객체를 전달 받는 코드가 있다면, 객체를 외부에서 주입 받고 있는 것으로 의존성 주입이 이루어지고 있다고 할 수 있다.

#### DI의 필요성

의존성 주입을 사용할 때는 현재 클래스 내부에서 외부 클래스의 객체를 생성하기 위한 **new 키워드의 사용 여부**를 결정해야한다.  
  

new 키워드를 사용해서 객체를 생성하게 되면 참조 할 클래스가 바뀌게 될 경우, 이 클래스를 사용하는 모든 클래스들을 수정해야 한다.  
이처럼 new 키워드를 사용해서 의존 객체를 생성할 때, 클래스들 간에 **강하게 결합(Tight Coupling)**되어 있다고 한다.  
  

결론적으로 의존성 주입을 하더라도 의존성 주입의 혜택을 보기 위해서는 클래스들 간에 **느슨한 결합(Loose Coupling)**이 필요하다.  
  

Java에서 클래스들 간의 관계를 느슨하게 만드는 대표적인 방법은 **인터페이스(Interface)를 사용**하는 것이다.  
  

**Spring에서는 애플리케이션 코드에서 이루어지는 의존성 주입을 Spring이 대신 해준다.**

---

### AOP(Aspect Oriented Programming)

AOP(관심 지향 프로그래밍)는 애플레케이션의 핵심 업무 로직에서 로깅이나 보안, 트랜잭션 같은 공통 기능 로직들을 분리하는 것이라고 할 수 있다.  
  

AOP에서 의미하는 Aspect는 애플리케이션의 공통 관심사를 의미한다.  
  

애플리케이션의 **공통 관심 사항(Cross-cutting concern)**는 비스니스 로직을 제외한 애플리케이션 전반에 걸쳐서 사용되는 공통 기능들을 의미하며, 로깅, 보안, 트랜잭션, 모니터링, 트레이싱 등의 기능이 있다.  
  

반대로 **핵심 관심 사항(Core concern)**은 애플리케이션의 주목적을 달성하기 위한 핵심 로직에 대한 관심사, 비즈니스 로직을 말한다.

#### AOP가 필요한 이휴

-   코드의 간결성 유지
-   객체 지향 설계 원칙에 맞는 코드 구현
-   코드의 재사용

---

### PSA(Portable Service Abstraction)

PSA는 **애플리케이션에서 특정 서비스를 이용할 떄, 서비스의 기능을 접근하는 방식 자체를 일관되게 유지하면서 기술 자체를 유연하게 사용할 수 있도록 하는 것**을 말한다.  
  

추상화된 상위 클래스를 일관되게 바라보며 하위 클래스의 기능을 사용하는 것이 PSA의 기본 개념이다.

#### PSA가 필요한 이유

어떤 서비스를 이용하기 위한 접근 방식을 일관된 방식으로 유지함으로써 애플리케이션에서 사용하는 기술이 변경되더라도 최소한의 변경만으로 변경된 요구사항을 반영하기 위함이다.  
  

Spring에서 PSA가 적용된 분야로는 트랜잭션 서비스, 메일 서비스, Spring Data 서비스 등이 있다.

---

## Spring Framework 모듈 구성

### 아키텍처(Architecture)

컴퓨터 시스템에서 아키텍처는 어떠한 시스템을 구축하는데 있어 해당 시스템의 비즈니스적 요구 사항을 만족하는 전체 시스템 구조를 정의하는 것이며, 이해 당사자들이 전체 시스템 구조를 이해하는데 무리가 없도록 일반적으로 이미지나 도형 등을 많이 사용한다.  
  

아주 거대하고 방대한 시스템이라면 어쩔 수 없이 어느 정도의 복잡함이 아키텍처 내에 표현될 수 밖에 없겠지만 기본적으로 **최대한 심플함을 유지하기 위해 노력**하는 것이 좋다.

#### 시스템 아키텍처

시스템 아키텍처는 하드웨어와 소프트웨어를 모두 포함하는 어떤 시스템의 전체적인 구성을 큰 그림으로 표현한 것이다.  
  

시스템 아키텍처를 통해 기본적으로 해당 시스템이 어떤 하드웨어로 구성되고, 어떤 소프트웨어를 사용하는지를 대략적으로 알 수 있다.  
  

또한 해당 시스템 구성 요소들 간의 상호작용이 어떻게 이루어지는지 등 시스템이 정상적으로 동작하기위한 동작 원리 등이 시스템 아키텍처 안에 표현이 되면 이해 당사자들이 해당 아키텍처를 이해하는데 도움이 된다.

#### 소프트웨어 아키텍처

소프트웨어는 하드웨어를 제외한 컴퓨터 내의 모든 프로그램을 포괄하는 의미를 가지고 있으며 이러한 소프트웨어의 구성을 큰그림으로 표현한 것이 소프트웨어 아키텍처이다.  
  

소프트웨어 아키텍처의 사례로 대표적인 것이 Java 플랫폼 아키텍처이다.  
  

[##_Image|kage@4cDte/btsar5DLfv4/4VjKrSKmrbWfZKV0qHdkek/img.gif|CDM|1.3|{"originWidth":646,"originHeight":396,"style":"alignCenter","width":null}_##]

#### 애플리케이션 아키텍처

애플리케이션은 소프트웨어 종류의 하나로써 좁게는 데스크탑이나 스마트폰에서 사용하는 응용 프로그램을 말하며, 넓게는 클라이언트의 요청을 처리하는 서버 애플리케이션을 의미한다.  
  

웹 상에서 동작하는 것을 웹 애플리케이션을 위한 아키텍처라고 하며, 이는 계층형 애플리케이션 아키텍처(N-티어)와 연관이 있다.  
  

[##_Image|kage@ZBJAg/btsaEh4B0H8/IizLrkWnAKOtC2V8c7QsKk/img.png|CDM|1.3|{"originWidth":841,"originHeight":666,"style":"alignCenter","width":null}_##]

-   API 계층(API Layer)  
    API 계층은 클라이언트의 요청을 받아들이는 계층이다. 일반적으로 표현 계층(Presentation Layer)라고도 불리지만 REST API를 제공하는 애플리케이션의 경우 API 계층이라고 표현한다.
-   서비스 계층(Service Layer)  
    서비스 계층은 API 계층에서 전달 받은 요청을 업무 도메인의 요구 사항에 맞게 비즈니스적으로 처리하는 계층이다. 애플리케이션의 핵심 로직은 서비스 계층에 포함되어 있다고 해도 과언이 아닐만큼 애플리케이션에 있어 핵심이 되는 계층이다.
    -   소프트웨어 공학의 도메인(Domain) : 소프트웨어가 풀고자하는 현실 세상의 문제 해결 대상 분야 또는 해당 비즈니스 영역
-   데이터 액세스 계층(Data Access Layer)  
    데이터 액세스 계층은 비즈니스 계층에서 처리된 데이터를 데이터베이스 등의 데이터 저장소에 저장하기 위한 계층이다.

---

### 아키텍처로 보는 Spring Framwork 모듈(Module) 구성

[##_Image|kage@EAFF3/btsaERx4RgO/Rj8qYZ2lNEe8pSLsADmR40/img.png|CDM|1.3|{"originWidth":720,"originHeight":540,"style":"alignCenter","width":null}_##]

  
  

위 그림은 Spring Framework에서 지원하는 모듈들을 아키텍처로 표현한 그림이다.  
  

Spring Framework에서는 약 20여개의 모듈을 통해 다양한 기능들을 제공한다.

-   모듈(Module) : Java에서는 일반적으로, 지원되는 여러가지 기능들을 목적에 맞게 그룹화하여 묶어 놓은 것을 모듈이라고 부른다. 모듈은 Java의 패키지 단위로 묶여 있으며, 이 패키지 안에는 관련 기능을 제공하기 위한 클래스들이 포함되어 있다. 또한, 일반적으로 모듈은 재사용이 가능하도록 라이브러리 형태로 제공되는 경우가 많다.

---

## Spring Boot

Spring Framework은 엔터프라이즈 애플리케이션을 개발하기 위한 핵심 기능을 제공하는 Spring Project 중 하나이다.  
  

그리고 Spring Boot은 Spring Framework의 편리함에도 불구하고 Spring 설정의 복잡함으로 인해 Spring 기반 애플리케이션 개발을 시작하기도 전에 어려움을 겪는 문제점을 해결하기 위해 생겨난 Spring Project 중 하나이다.

### Spring Boot를 사용해야 하는 이유

-   XML 기반의 복잡한 설계 방식 지양
-   의존 라이브러리의 자동 관리
-   애플리케이션 설정의 자동 구성
-   프로덕션급 애플리케이션의 손쉬운 빌드
-   내장된 WAS를 통한 손쉬운 배포
    -   WAS(Web Application Server) : 구현된 코드를 빌드해서 나온 결과물을 웹 애플리케이션으로 실행되게 해주는 서버