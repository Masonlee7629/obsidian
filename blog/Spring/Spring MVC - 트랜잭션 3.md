태그: #Spring
연결문서: [Spring MVC - 테스팅 1](Spring%20MVC%20-%20테스팅%201.md)

# 트랜잭션

## JTA를 이용한 분산 트랜잭션 적용

서로 다른 데이터소스를 사용하는 한 개 이상의 데이터베이스를 하나의 트랜잭션으로 묶어서 처리하는 것을 **분산 트랜잭션**이라고 한다.  
  

H2 인메모리 DB의 경우 분산 트랜잭션이 정상적으로 동작하지 않는다.  
  

분산 트랜잭션은 하나의 PC에 물리적으로 데이터베이스의 인스턴스 두 개를 실행히키는 것이 아니라 'create dababase' 명령어로 두 개의 데이터베이스를 생성해서 각각 별도의 데이터소스를 사용하는 구조이다.

### 분산 트랜잭션 구조

[##_Image|kage@cATmLW/btsayVIYZar/NXMvuemaNXPqekSKRwbMYK/img.png|CDM|1.3|{"originWidth":1080,"originHeight":800,"style":"alignCenter","width":null}_##]

위 그림은 MySQL을 사용할 경우, 분산 트랜잭션의 적용 예시이다.  
  

백업용 회원 정보는 기존 회원 정보의 백업 데이터 역할을 하며, 분산 트랜잭션의 적용을 확인하기 위한 임시 테이블이라고 가정한다.  
  

백업을 위해 특정 데이터베이스의 데이터를 다른 데이터베이스로 복제하는 방법은 여러가지가 존재한다.  
  

같은 종류의 데이터베이스일 경우, 복제(Replication) 기능을 이용해서 데이터를 백업할 수 있다.  
  

다른 종류의 데이터베이스 간에 사용할 수 있는 방법은 애플리케이션의 **스케쥴링 기능**을 통해 주기적으로 원본 데이터베이스의 데이터를 다른 데이터베이스로 백업하는 기능을 구현할 수 있으며, 이런 기능들을 기본적으로 지원하는 **Apache NiFi** 같은 오픈 소스 기술을 사용할 수도 있다.

### 분산 트랜잭 적용을 위한 사전 준비(MySQL)

#### MySQL 설치

분산 트랜잭션을 적용하기 위해서는 데이터베이스가 준비되어 있어야 한다.

#### 두 개의 데이터베이스 생성

분산 트랜잭션을 적용하기 위해서는 두 개의 database와 user가 필요하다.

-   기존의 커피 주문 정보를 위한 데이터베이스 정보

```
mysql> create database coffee_order;
mysql> create user guest@localhost identified by 'guest';
mysql> grant all privileges on *.* to guest@localhost;
mysql> flush privileges;
```

-   백업 회원 정보를 위한 데이터베이스 정보

```
mysql> create database backup_data;
mysql> create user backup@localhost identified by 'backup';
mysql> grant all privileges on *.* to backup@localhost;
mysql> flush privileges;
```

#### 의존 라이브러리 추가

MySQL DB를 사용하기 때문에 MySQL 커넥터와 분산 트랜잭션 오픈 소스인 atomikos를 의존 라이브러리로 추가한다.

```
dependencies {
    ...
    ...
    implementation 'mysql:mysql-connector-java'
    implementation 'org.springframework.boot:spring-boot-starter-jta-atomikos'
}
```

### 두 개의 데이터베이스 설정

두 개의 데이터베이스에 분산 트랜잭션을 적용하려면 두 개의 데이터베이스에 다음과 같은 설정이 필요하다.

1.  해당 데이터베이스에 맞는 데이터소스(Datasource)를 생성한다.
2.  각 데이터소스를 JPA의 EntityManager가 인식하도록 설정한다.
3.  JTA TransactionManager 설정
4.  JTA Platform 설정

#### 커피 주문을 위한 데이터베이스(`XaCoffeeOrderConfig`) 설정

```
import com.atomikos.jdbc.AtomikosDataSourceBean;
import com.mysql.cj.jdbc.MysqlXADataSource;
import org.hibernate.engine.transaction.jta.platform.internal.AtomikosJtaPlatform;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

// (1) JpaRepository 활성화
@EnableJpaRepositories(
        basePackages = {"com.codestates.member",
                "com.codestates.stamp",
                "com.codestates.order",
                "com.codestates.coffee"},
        entityManagerFactoryRef = "coffeeOrderEntityManager"
)
@Configuration
public class XaCoffeeOrderConfig {
        // (2) 데이터소스 생성
    @Primary
    @Bean
    public DataSource dataSourceCoffeeOrder() {
        MysqlXADataSource mysqlXADataSource = new MysqlXADataSource();
        mysqlXADataSource.setURL("jdbc:mysql://localhost:3306/coffee_order" +
                "?allowPublicKeyRetrieval=true" +
                "&characterEncoding=UTF-8");
        mysqlXADataSource.setUser("guest");
        mysqlXADataSource.setPassword("guest");

        AtomikosDataSourceBean atomikosDataSourceBean = new AtomikosDataSourceBean();
        atomikosDataSourceBean.setXaDataSource(mysqlXADataSource);
        atomikosDataSourceBean.setUniqueResourceName("xaCoffeeOrder");

        return atomikosDataSourceBean;
    }

        // (3) EntityManagerFactoryBean 설정
    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean coffeeOrderEntityManager() {
        LocalContainerEntityManagerFactoryBean emFactoryBean =
                new LocalContainerEntityManagerFactoryBean();
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setDatabase(Database.MYSQL);
        Map<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto", "create");
        properties.put("hibernate.show_sql", "true");
        properties.put("hibernate.format_sql", "true");

                // (4)
        properties.put("hibernate.transaction.jta.platform", 
                                             AtomikosJtaPlatform.class.getName());
        properties.put("javax.persistence.transactionType", "JTA");

        emFactoryBean.setDataSource(dataSourceCoffeeOrder());
        emFactoryBean.setPackagesToScan(new String[]{
                "com.codestates.member",
                "com.codestates.stamp",
                "com.codestates.order",
                "com.codestates.coffee"
        });
        emFactoryBean.setJpaVendorAdapter(vendorAdapter);
        emFactoryBean.setPersistenceUnitName("coffeeOrderPersistenceUnit");
        emFactoryBean.setJpaPropertyMap(properties);

        return emFactoryBean;
    }
}
```

위 코드는 커피 주문을 위한 데이터베이스 설정 코드이다.  
  

로컬 트랜잭션의 경우, Spring Boot의 자동 구성을 이용했기 때문에 개발자가 해야할 것이 별로 없지만 분산 트랜잭션의 경우, 별도의 추가 설정이 필요하다.  
  

서블릿 컨테이너 환경에서 분산 트랜잭션을 적용하기 위해서는 별도의 JTA 트랜잭션 매니저가 필요하다. 예시에서는 가장 많이 사용되는 오픈 소스 JTA 트랜잭션 매니저 플램폼인 Atomikos를 사용한다.

-   (1)에서 데이터베이스를 사용하기 위한 `JpaRepository`가 위치한 패키지와 `entityManagerFactory` 빈에 대한 참조를 설정한다.
    -   `basePackages`
        -   해당 Repository가 있는 패키지 경로를 적어준다.
    -   `entityManagerFactory`
        -   (3)의 bean 생성 메서드 명을 적어준다.
-   (2)에서 데이터베이스에 대한 데이터소스를 생성하기 위해 데이터베이스 접속 정보들을 설정한다.
-   (3)에서 JPA의 `EntityManager`를 얻기 위해 `LocalContainerEntityManagerFactoryBean` 을 사용한다.
    -   `LocalContainerEntityManagerFactoryBean` 에서 사용하는 어댑터 중에서 현재 사용하는 `HibernateJpaVendorAdapter`를 설정해주고, Hibernate에서 필요한 설정 정보를 Map으로 설정해준다.
    -   `EntityManager`가 사용할 `Entity` 클래스가 위치한 패키지 경로를 지정해준다.
-   (4)와 같이 JTA Platform의 이름을 추가해준다.

#### 백업용 회원 정보 데이터베이스(`XaBackupConfig`) 설정

```
import com.atomikos.jdbc.AtomikosDataSourceBean;
import com.mysql.cj.jdbc.MysqlXADataSource;
import org.hibernate.engine.transaction.jta.platform.internal.AtomikosJtaPlatform;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

// (1)
@EnableJpaRepositories(
        basePackages = {"com.codestates.backup"},
        entityManagerFactoryRef = "backupEntityManager"
)
@Configuration
public class XaBackupConfig {
    @Bean
    public DataSource dataSourceBackup() {
                // (2)
        MysqlXADataSource mysqlXADataSource = new MysqlXADataSource();
        mysqlXADataSource.setURL("jdbc:mysql://localhost:3306/backup_data" +
                "?allowPublicKeyRetrieval=true" +
                "&characterEncoding=UTF-8");
        mysqlXADataSource.setUser("backup");
        mysqlXADataSource.setPassword("backup");

        AtomikosDataSourceBean atomikosDataSourceBean = new AtomikosDataSourceBean();
        atomikosDataSourceBean.setXaDataSource(mysqlXADataSource);
        atomikosDataSourceBean.setUniqueResourceName("xaMySQLBackupMember");

        return atomikosDataSourceBean;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean backupEntityManager() {
        LocalContainerEntityManagerFactoryBean emFactoryBean =
                new LocalContainerEntityManagerFactoryBean();
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setDatabase(Database.MYSQL);
        Map<String, Object> properties = new HashMap<>();
        properties.put("hibernate.hbm2ddl.auto", "create");
        properties.put("hibernate.show_sql", "true");
        properties.put("hibernate.format_sql", "true");
        properties.put("hibernate.transaction.jta.platform",  
                                             AtomikosJtaPlatform.class.getName());
        properties.put("javax.persistence.transactionType", "JTA");

        emFactoryBean.setDataSource(dataSourceBackup());

                // (3)
        emFactoryBean.setPackagesToScan(new String[]{"com.codestates.backup"});
        emFactoryBean.setJpaVendorAdapter(vendorAdapter);
        emFactoryBean.setPersistenceUnitName("backupPersistenceUnit");
        emFactoryBean.setJpaPropertyMap(properties);

        return emFactoryBean;
    }
}
```

위 코드는 분샌 트랜잭션 적용을 위한 백업용 회원 정보 데이터베이스를 위한 설정 코드이다.

-   데이터베이스에서 사용할 `JpaRepository`가 위치한 패키지 경로를 (1)과 같이 변경한다.
-   (2)와 같이 MySQL 접속 정보를 입력한다.
-   (3)과 같이 `EntityManager`가 사용할 `Entity` 클래스가 위치한 패키지 경로를 변경한다.

#### JTA TransactionManager 설정

```
import javax.transaction.TransactionManager;
import javax.transaction.UserTransaction;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.transaction.jta.JtaTransactionManager;

import com.atomikos.icatch.jta.UserTransactionImp;
import com.atomikos.icatch.jta.UserTransactionManager;

@Configuration
public class JtaConfig {
    // (1)
    @Bean(name = "userTransaction")
    public UserTransaction userTransaction() throws Throwable {
        UserTransactionImp userTransactionImp = new UserTransactionImp();
        userTransactionImp.setTransactionTimeout(10000);
        return userTransactionImp;
    }

    @Bean(name = "atomikosTransactionManager")
    public TransactionManager atomikosTransactionManager() throws Throwable {
    // (2)
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        userTransactionManager.setForceShutdown(false);

    // (3)
        AtomikosJtaPlatform.transactionManager = userTransactionManager;

        return userTransactionManager;
    }

    @Bean(name = "transactionManager")
    @DependsOn({ "userTransaction", "atomikosTransactionManager" })
    public PlatformTransactionManager transactionManager() throws Throwable {
        UserTransaction userTransaction = userTransaction();

        AtomikosJtaPlatform.transaction = userTransaction;

        TransactionManager atomikosTransactionManager = atomikosTransactionManager();

    // (4)
        return new JtaTransactionManager(userTransaction, atomikosTransactionManager);
    }
}
```

JTA TransactionManager를 사용하기 위한 설정 코드이다.  
  

Atomikos와 관련된 두 개의 Bean은 `userTransaction`과 `atomikosTransactionManager`이며, 이 두 개의 Bean을 (4)와 같이 `JtaTransactionManager`의 생성자로 넘겨주면 Atomikos의 분산 트랜잭션을 사용할 수 있다.

-   (1)의 `UserTransaction`은 애플리케이션이 트랜잭션 경계에서 관리되는 것을 명시적으로 정의한다.
-   (2)와 같이 `UserTransaction`을 관리하는 `UserTransactionManager`를 생성한 후에 (3)과 같이 `AtomikosJtaPlatform`의 트랜잭션 매니저로 설정한다.

#### JTA Platform(`AtomikosJtaPlatform`) 설정

```
public class AtomikosJtaPlatform  extends AbstractJtaPlatform {
    static TransactionManager transactionManager;
    static UserTransaction transaction;

    @Override
    protected TransactionManager locateTransactionManager() {
        return transactionManager;
    }

    @Override
    protected UserTransaction locateUserTransaction() {
        return transaction;
    }
}
```

`AbstractJtaPlatform`을 상속한 후에 트랜잭션 매니저의 위치와 `UserTransaction`의 위치를 지정한다.  
  

이 작업은 `JTA TransactionManager(JtaConfig)` 설정에서 이루어진다.

#### 회원 백업 정보 엔티티 클래스 정의

```
package com.codestates.backup.entity;

import com.codestates.audit.Auditable;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import javax.persistence.*;

@NoArgsConstructor
@Getter
@Setter
@Entity
public class BackupMember extends Auditable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberId;

    @Column(nullable = false, updatable = false, unique = true)
    private String email;

    @Column(length = 100, nullable = false)
    private String name;

    @Column(length = 13, nullable = false, unique = true)
    private String phone;

    @Enumerated(value = EnumType.STRING)
    @Column(length = 20, nullable = false)
    private MemberStatus memberStatus = MemberStatus.MEMBER_ACTIVE;

    public BackupMember(String email) {
        this.email = email;
    }

    public BackupMember(String email, String name, String phone) {
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

전체적으로 `Member` 클래스와 동일하며, **다른 엔티티 클래스와의 연관 관계를 제거**한다.

#### 회원 백업 정보 저장을 위한 Repository 인터페이스

```
import com.codestates.backup.entity.BackupMember;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BackupMemberRepository extends JpaRepository<BackupMember, Long> {
}
```

회원 백업 정보 저장을 위한 `BackupMemberRepository`는 `Member`가 `BackupMember`로 변경되었다는 것 외에 `MemberRepository`와 동일하다.

#### MemberService에서 회원 정보와 회원 백업 정보 등록

```
@Transactional
@Service
public class MemberService {
    private final BackupMemberService backupMemberService;
    private final MemberRepository memberRepository;
    private final BackupMemberRepository backupMemberRepository;

    public MemberService(BackupMemberService backupMemberService, // (1)
                         MemberRepository memberRepository) {
        this.backupMemberService = backupMemberService;
        this.memberRepository = memberRepository;
        this.backupMemberRepository = backupMemberRepository;
    }

    @Transactional
    public Member createMember(Member member) {
        verifyExistsEmail(member.getEmail());
        Member savedMember = memberRepository.save(member);

                // (2)
        backupMemberService.createBackupMember(new BackupMember(member.getEmail(),
                member.getName(), member.getPhone()));

        return savedMember;
    }
        ...
        ...
}
```

MemberService의 createMember() 메서드를 위 코드와 같이 데이터베이스에서 회원 정보를 등록할 떄, 백업용 회원 정보 데이터베이스에 추가로 회원 정보를 저장하도록 구현한다.

-   (1)에서 백업용 회원 정보를 저장할 `BackupMemberService`를 DI 받는다.
-   회원 정보를 등록한 후, (2)에서 백업용 회원 정보를 저장한다.

#### MemberService에서 회원 정보와 회원 백업 정보 등록

```
import com.codestates.backup.entity.BackupMember;
import com.codestates.backup.repository.BackupMemberRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class BackupMemberService {
    private final BackupMemberRepository backupMemberRepository;

    public BackupMemberService(BackupMemberRepository backupMemberRepository) {
        this.backupMemberRepository = backupMemberRepository;
    }

    @Transactional
    public void createBackupMember(BackupMember backupMember) {
        backupMemberRepository.save(backupMember);

                // (1)
        throw new RuntimeException("multi datasource rollback test");
    }
}
```

실질적으로 백업용 회원 정보 저장 작업을 수행하는 코드이다.  
  

회원 정보 저장 중에 예외 발생을 시뮬레이션하기 위해 (1)과 같이 RuntimeException을 발생시켰다.

#### 테스트 결과

애플리케이션을 실행시키고, Postman으로 `postMember()`에 요청을 전송하면 두 개의 데이터베이스에 각각 회원 정보를 저장하기 위한 작업을 하다가 `BackupMemberService`에서 예외가 발생하기 때문에 두 개의 데이터베이스에는 모두 데이터가 저장되지 않는다.  
  

이처럼 분산 트랜잭션을 적용하면 서로 **다른 리소스에 대한 작업을 수행하더라도 하나의 작업처럼 관리하기 때문에 원자성을 보장할 수 있다.**