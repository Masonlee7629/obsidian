태그: #Spring
연결문서: [Spring MVC - Spring Data JPA 2](Spring%20MVC%20-%20Spring%20Data%20JPA%202.md)

# Spring Data JPA

## JPA(Java Persistence API)란?

JPA는 Java 진영에서 사용하는 ORM(Object-Relational Mapping) 기술의 표준 사양(또는 명세, Specification)이다.  
  

표준 사양은 Java의 인터페이스로 사양이 정의되어 있기 때문에 JPA라는 표준 사양을 구현한 구현체는 따로 있다는 것을 의미한다.  
  

JPA 표준 사양을 구현한 구현체로 Hibernate ORM, EclipseLink, DataNucleus 등이 있다.  
  

이 Hibernate ORM은 JPA에서 정의해둔 인터페이스를 구현한 구현체로써 JPA에서 지원하는 기능 이외에 Hibernate 자체적으로 사용할 수 있는 API 역시 지원하고 있다.

### 데이터 액세스 계층에서의 JPA 위치

[##_Image|kage@nNdZ3/btsaJTRe45s/yNVtPmTAekYXXKtwl8VJHk/img.png|CDM|1.3|{"originWidth":1920,"originHeight":1080,"style":"alignCenter","width":null}_##]

  
데이터 액세스 계층에서 JPA는 계층의 상단에 위치한다.  
  

데이터 저장, 조회 등의 작업은 JPA를 거쳐 JPA의 구현체인 Hibernate ORM을 통해서 이루어지며 Hibernate ORM은 내부적으로 JDBC API를 이용해서 데이터베이스에 접근하게 된다.

### JPA에서 P(Persistence)의 의미

Persistence는 영속성, 지속성이라는 뜻을 가지고 있다. 즉, 무언가를 금방 사라지지 않고 오래 지속되게 한다는 것이 Persistence의 목적이다.

#### 영속성 컨텍스트(Persistence Context)

ORM은 객체(Object)와 데이터베이스 테이블의 매핑을 통해 엔티티 클래스 객체 안에 포함된 정보를 테이블에 저장하는 기술이다.  
  

JPA에서는 테이블과 매핑되는 엔티티 객체 정보를 **영속성 컨텍스트(Persistence Context)라는 곳에 보관**해서 애플리케이션 내에서 오래 지속되도록 한다.  
  

이렇게 보관된 엔티티 정보는 데이터베이스 테이블에 데이터를 저장, 수정, 조회, 삭제하는데 사용된다.  
  

[##_Image|kage@cvdrol/btsayWHPNHX/TSkizEn5kOJgfmGbg0IZHk/img.png|CDM|1.3|{"originWidth":1280,"originHeight":1080,"style":"alignCenter","width":null}_##]

위 그림과 같이 영속성 컨텍스트에는 **1차 캐시**라는 영역과 **쓰기 지연 SQL 저장소**라는 영역이 있다.  
  

JPA API 중에서 엔티티 정보를 영속성 컨텍스트에 저장하는 API를 사용하면 영속성 컨텍스트의 1차 캐시에 엔티티 정보가 저장된다.

#### JPA API로 영속성 컨텍스트 이해하기

-   build.gradle 설정  
    spring-boot-starter-data-jpa를 추가하면 기본적으로 Spring Data JPA 기술을 포함해서 JPA API를 사용할 수 있다.

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa' // (1)
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

-   JPA 설정(application.yml)
    -   (1) : JPA에서 사용하는 엔티티 클래스를 정의하고 애플리케이션 실행 시, 이 엔티티와 매핑되는 테이블을 데이터베이스에 자동으로 생성해준다.
    -   (2) : JPA의 동작 과정을 이해하기 위해 JPA API를 통해서 실행되는 SQL 쿼리를 로그로 출력해준다.

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
      ddl-auto: create  # (1) 스키마 자동 생성
    show-sql: true      # (2) SQL 쿼리 출력
```

-   샘플 코드 실행을 위한 Configuration 클래스 생성
    -   (1) 특정 클래스에 `@Configuration` 애너테이션을 추가하면, Spring에서 Bean 검색 대상인 Configuration 클래스로 간주한다.
    -   (2)와 같이 `@Bean` 애너테이션이 추가된 메서드를 검색한 후, 해당 메서드에서 리턴하는 객체를 Spring Bean으로 추가해준다.
    -   (3)과 같이 CommandLineRunner 객체를 람다 표현식으로 정의해주면 애플리케이션 부트스트랩 과정이 완료된 후에 이 람다 표현식에 정의한 코드를 실행해준다.

```
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;

// (1)
@Configuration
public class JpaBasicConfig {
    private EntityManager em;
    private EntityTransaction tx;

        // (2)
    @Bean
    public CommandLineRunner testJpaBasicRunner(EntityManagerFactory emFactory) {
        this.em = emFactory.createEntityManager();
        this.tx = em.getTransaction();

        return args -> {
            (3)
        };
    }
}
```

-   영속성 컨텍스트에 엔티티 저장
    -   (1)과 (2)의 `@Entity`와 `@ID` 애너테이션을 추가하면 JPA에서 해당 클래스를 엔티티 클래스로 인식한다.
    -   (3)의 `@GeneratedValue` 애너테이션은 식별자를 생성해주는 전략을 지정할 때 사용한다.

```
import lombok.Getter;

import javax.persistence.*;

@Getter
@Setter
@NoArgsConstructor
@Entity  // (1)
public class Member {
    @Id  // (2)
    @GeneratedValue  // (3)
    private Long memberId;

    private String email;

    public Member(String email) {
        this.email = email;
    }
}
```

-   영속성 컨텍스트와 테이블에 엔티티 저장
    -   (1)에서는 `EntityManager`를 통해서 `Transaction` 객체를 얻는다. JPA에서는 이 `Transaction` 객체를 기준으로 데이터베이스의 테이블에 데이터를 저장한다.
    -   (2)와 같이 `Transaction`을 시작하기 위해서 `tx.begin()` 메서드를 먼저 호출해야한다.
    -   (3)에서 `member` 객체를 영속성 컨텍스트에 저장한다.
    -   (4)와 같이 `tx.commit()`을 호출하는 시점에 영속성 컨텍스트에 저장되어 있는 `member` 객체를 **데이터베이스의 테이블에 저장**한다.
    -   (5)에서 `em.find(Member.class, 1L)`을 호출하면 (3)에서 영속성 컨텍스트에 저장한 `member` 객체를 1차 캐시에서 조회한다.  
        1차 캐시에 member 객체 정보가 있기때문에 별도로 테이블에 SELECT 쿼리를 전송하지 않는다.
    -   (6)에서 `em.find(Member.class, 2L)`를 호출해서 식별자 값이 2L인 `member` 객체를 조회한다. 하지만 영속성 컨텍스트에는 식별자 값이 2L인 member 객체는 존재하지 않기 때문에 (7)의 결과는 true가 된다.  
        (6)에서는 영속성 컨텍스트에서 식별자 값이 2L인 member 객체가 존재하지 않기 때문에 테이블에 직접 SELECT 쿼리를 전송한다.

```
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;

@Configuration
public class JpaBasicConfig {
    private EntityManager em;
    private EntityTransaction tx;

    @Bean
    public CommandLineRunner testJpaBasicRunner(EntityManagerFactory emFactory) {
        this.em = emFactory.createEntityManager();

        // (1)
        this.tx = em.getTransaction();

        return args -> {
            example02();
        };
    }

    private void example02() {
        // (2)
        tx.begin();
        Member member = new Member("hgd@gmail.com");

        // (3)
        em.persist(member);

        // (4)
        tx.commit();

        // (5)
        Member resultMember1 = em.find(Member.class, 1L);

        System.out.println("Id: " + resultMember1.getMemberId() + ", email: " + resultMember1.getEmail());

        // (6)
        Member resultMember2 = em.find(Member.class, 2L);

        // (7)
        System.out.println(resultMember2 == null);

    }
}
```

[##_Image|kage@czHu0p/btsaPGcZ2l1/5YzjBPwgvCS9TJL5mJf9A1/img.png|CDM|1.3|{"originWidth":1280,"originHeight":1080,"style":"alignCenter","width":null}_##]

위 그림은 코드 실행 시 영속성 컨텍스트의 상태이다.  
  

`tx.commit()` 을 했기 때문에 `member`에 대한 INSERT 쿼리는 실행되어 쓰기 지연 SQL 저장소에서 사라진다.  
  

실행 결과를 보면 (1)에서 SELECT 쿼리가 실행된 것을 볼 수 있다.  
  

이 SELECT 쿼리를 통해 (6)에서 `em.find(Member.class, 2L)` 로 조회를 했는데 식별자 값이 2L에 해당하는 `member2` 객체가 영속성 컨텍스트의 1차 캐시에 없기 때문에 추가적으로 한번 더 조회한다.

-   em.persist()를 호출하면 영속성 컨텍스트의 1차 캐시에 엔티티 클래스의 객체가 저장되고, 쓰기 지연 SQL 저장소에 INSERT 쿼리가 등록된다.
-   tx.commit()을 하는 순간 쓰기 지연 SQL 저장소에 등록된 INSERT 쿼리가 실행되고, 실행된 INSERT 쿼리는 쓰기 지연 SQL 저장소에서 제거된다.
-   em.find()를 호출하면 먼저 1차 캐시에서 해당 객체가 있는지 조회하고, 없으면 테이블에 SELECT 쿼리를 전송해서 조회한다.

```
Hibernate: drop table if exists member CASCADE 
Hibernate: drop table if exists orders CASCADE 
Hibernate: drop sequence if exists hibernate_sequence
Hibernate: create sequence hibernate_sequence start with 1 increment by 1
Hibernate: create table member (member_id bigint not null, email varchar(255), 
                                                       primary key (member_id))
Hibernate: create table orders (order_id bigint not null, created_at timestamp, 
                                                      primary key (order_id))
Hibernate: call next value for hibernate_sequence
Hibernate: insert into member (email, member_id) values (?, ?)
Id: 1, email: hgd@gmail.com

// (1)
**Hibernate: select member0_.member_id as member_i1_0_0_, 
        member0_.email as email2_0_0_ from member member0_ where member0_.member_id=?**
true
```

-   쓰기 지연을 통한 영속성 컨텍스트와 테이블에 엔티티 일괄 저장

```
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;

@Configuration
public class JpaBasicConfig {
    private EntityManager em;
    private EntityTransaction tx;

    @Bean
    public CommandLineRunner testJpaBasicRunner(EntityManagerFactory emFactory) {
        this.em = emFactory.createEntityManager();
        this.tx = em.getTransaction();

        return args -> {
                example03();
        };
    }

    private void example03() {
        tx.begin();

        Member member1 = new Member("hgd1@gmail.com");
        Member member2 = new Member("hgd2@gmail.com");

        em.persist(member1);  // (1)
        em.persist(member2);  // (2)


        tx.commit(); // (3)
     }
}
```

[##_Image|kage@oBk8i/btsaqOKp4Ar/gKQFq2wflF9VW2bmjdhv6K/img.png|CDM|1.3|{"originWidth":1280,"originHeight":1080,"style":"alignCenter","width":null}_##]

위 그림은 `tx.commit()` 이 실행되기 직전의 영속성 컨텍스트 상태를 표현한 것이다.  
  

`tx.commit()`을 하기 전까지는 `em.persist()`를 통해 쓰기 지연 SQL 저장소에 등록된 INSERT 쿼리가 실행이 되지 않는다.  
  

따라서 테이블에 데이터가 저장되지 않는다.  
  

[##_Image|kage@bEQLlS/btsar6qeQOY/9kKkYNxapYKL0UUY4JkZA1/img.png|CDM|1.3|{"originWidth":1280,"originHeight":1080,"style":"alignCenter","width":null}_##]

위 그림은 `tx.commit()` 이 실행된 직후의 영속성 컨텍스트 상태를 표현한 것이다.  
  

`tx.commit()`이 실행된 이후에는 쓰기 지연 SQL 저장소에 등록된 INSERT 쿼리가 모두 실행되고 실행된 쿼리는 제거된다.  
  

따라서 테이블에 데이터가 저장된다.  
  

코드 실행 결과 (1)과 같이 쓰기 지연 SQL 저장소에 저장된 INSERT 쿼리가 실행된 것을 확인할 수 있다.

```
Hibernate: drop table if exists member CASCADE 
Hibernate: drop table if exists orders CASCADE 
Hibernate: drop sequence if exists hibernate_sequence
Hibernate: create sequence hibernate_sequence start with 1 increment by 1
Hibernate: create table member (member_id bigint not null, email varchar(255), primary key (member_id))
Hibernate: create table orders (order_id bigint not null, created_at timestamp, primary key (order_id))
Hibernate: call next value for hibernate_sequence
Hibernate: call next value for hibernate_sequence

// (1)
Hibernate: insert into member (email, member_id) values (?, ?)
Hibernate: insert into member (email, member_id) values (?, ?)
```

-   영속성 컨텍스트와 테이블에 엔티티 업데이트
    -   (1)에서 member 객체를 영속성 컨텍스트의 1차 캐시에 저장한다.
    -   (2)에서 `tx.commit()`을 호출해서 영속성 컨텍스트의 쓰지 지연 SQL 저장소에 등록된 INSERT 쿼리를 실행한다.
    -   (3)과 같이 (2)에서 테이블에 저장된 member 객체를 영속성 컨텍스트의 1차 캐시에서 조회한다.  
        이는 **테이블에서 조회하는 것이 아니다.**  
        영속성 컨텍스트의 1차 캐시에 이미 저장된 객체가 있기 때문에 영속성 컨텍스트에서 조회한다.
    -   (4)에서 setter 메서드로 이메일 정보를 변경한다.
    -   (5)에서 `tx.commit()`을 실행하면 쓰기 지연 SQL 저장소에 등록된 UPDATE 쿼리가 실행된다.
-   UPDATE 쿼리 실행 과정
    1.  영속성 컨텍스트에 엔티티가 저장될 경우에는 저장되는 시점의 상태를 그대로 가지고 있는 스냅샷을 생성한다.
    2.  해당 엔티티의 값을 setter 메서드로 변경한 후, `tx.commit()`을 하면 변경된 엔티티와 이 전에 이미 떠 놓은 스냅샷을 비교한 후, 변경된 값이 있으면 쓰기 지연 SQL 저장소에 UPDATE 쿼리를 등록하고 UPDATE 쿼리를 실행한다.

```
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;

@Configuration
public class JpaBasicConfig {
    private EntityManager em;
    private EntityTransaction tx;

    @Bean
    public CommandLineRunner testJpaBasicRunner(EntityManagerFactory emFactory) {
        this.em = emFactory.createEntityManager();
        this.tx = em.getTransaction();

        return args -> {
             example04();
        };
    }

    private void example04() {
       tx.begin();
       em.persist(new Member("hgd1@gmail.com"));    // (1)
       tx.commit();    // (2)


       tx.begin();
       Member member1 = em.find(Member.class, 1L);  // (3)
       member1.setEmail("hgd1@yahoo.co.kr");       // (4)
       tx.commit();   // (5)
    }
}
```

-   영속성 컨텍스트와 테이블의 엔티티 삭제
    -   (1)에서 Member 클래스의 객체를 영속성 컨텍스트의 1차 캐시에 저장한다.
    -   (2)에서 `tx.commit()`을 호출해서 영속성 컨텍스트의 쓰기 지연 SQL 저장소에 등록된 INSERT 쿼리를 실행한다.
    -   (2)에서 테이블에 저장된 Member 클래스의 객체를 (3)과 같이 영속성 컨텍스트의 1차 캐시에서 조회한다.
    -   (4)에서 `em.remove(member)`을 통해 영속성 컨텍스트의 1차 캐시에 있는 엔티티의 제거를 요청한다.
    -   (5)에서 `tx.commit()`을 실행하면 영속성 컨텍스트의 1차 캐시에 있는 엔티티를 제거하고, 쓰기 지연 SQL 저장소에 등록된 DELETE 쿼리가 실행된다.
-   `EntityManager`의 `flush()` API
    -   `tx.commit()` 메서드가 호출되면 JPA 내부적으로 `em.flush()` 메서드가 호출되어 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영한다

```
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;

@Configuration
public class JpaBasicConfig {
    private EntityManager em;
    private EntityTransaction tx;

    @Bean
    public CommandLineRunner testJpaBasicRunner(EntityManagerFactory emFactory) {
        this.em = emFactory.createEntityManager();
        this.tx = em.getTransaction();

        return args -> {
            example05();
        };
    }

    private void example05() {
        tx.begin();
        em.persist(new Member("hgd1@gmail.com"));  // (1)
        tx.commit();    //(2)

        tx.begin();
        Member member = em.find(Member.class, 1L);   // (3)
        em.remove(member);     // (4)
        tx.commit();     // (5)
    }
}
```
