태그: #Spring
연결문서: [Spring MVC - 트랜잭션 1](Spring%20MVC%20-%20트랜잭션%201.md)

# Spring Data JPA

## Spring Data JPA를 통한 데이터 액세스 계층 구현

### Spring Data JPA란?

Spring Data JPA는 Spring Data 패밀리 기술 중 하나로써, JPA 스펙을 구현한 구현체의 API를 조금 더 쉽게 사용할 수 있도록 해주는 모듈이다.  
  

Spring에서는 애플리케이션이 **특정 기술에 결합되지 않도록 Spring이 추구하는 PSA를 통해 개발자는 일관된 코드 구현 방식을 유지하도록 하고, 기술의 변경이 필요할 때 최소한의 변경만을 하도록 지원한다.** (Spring Data JDBC -> Spring Data JPA)

#### Spring Data JPA 적용 순서

1.  엔티티 클래스를 Spring Data JPA에 맞게 수정
2.  리포지토리(Repository) 인터페이스 구현
3.  서비스 클래스 구현
4.  기타 기능 추가로 인한 코드 수정

#### 리포지토리 인터페이스 구현

```
public interface CoffeeRepository extends JpaRepository<Coffee, Long> { // (1)
    Optional<Coffee> findByCoffeeCode(String coffeeCode);

// @Query(value = "FROM Coffee c WHERE c.coffeeId = :coffeeId") // (2-1)
// @Query(value = "SELECT * FROM COFFEE WHERE coffee_Id = :coffeeId", nativeQuery = true) // (2-2)
    @Query(value = "SELECT c FROM Coffee c WHERE c.coffeeId = :coffeeId") // (2-3)
    Optional<Coffee> findByCoffee(long coffeeId);
}
```

(1)과 같이 Spring Data JDBC와 비교했을 때, `CrudRepository` 를 상속하는 대신 `JpaRepository` 를 상속한다.  
  

JPA에서는 복잡한 검색 조건을 지정하기 위한 몇 가지 방법을 제공한다.

-   **JPQL을 통한 객체 지향 쿼리 사용**
    -   JPA에선 JPQL이라는 객체 지향 쿼리를 통해 데이터베이스 내의 테이블의 조회 가능
    -   JPQL은 엔티티 클래스의 객체를 대상으로 객체를 조회하는 방법
    -   JPQL의 문법을 사용해서 객체를 조회하면 JPA가 내부적으로 JPQL을 분석해서 적절한 SQL을 만든 후에 데이터베이스를 조회하고, 조회한 결과를 엔티티 객체로 매핑한 뒤에 반환
    -   (2-3)은 JPQL을 사용해서 `coffeeId`에 해당하는 커피 정보를 조회하는데, SQL과 유사하지만 차이점이 있음
        -   JPQL은 **객체를 대상으로 한 조회**이기 때문에 COFFEE 테이블이 아니라 `Coffee` 클래스라는 객체를 지정해야하고, coffee\_id라는 컬럼이 아닌 coffeeId 필드를 지정해야 함
        -   (2-3)의 “`SELECT c FROM Coffee c WHERE c.coffeeId = :coffeeId`”에서 `Coffee`는 클래스명이고, `coffeeId`는 `Coffee` 클래스의 필드명
        -   ‘`c`’는 `Coffee` 클래스의 별칭이기 때문에 “`SELECT c FROM~`” 와 같이 SQL에서 사용하는 ‘`*`’이 아니라 ‘`c`’로 모든 필드를 조회
        -   (2-3)은 (2-1)과 같이 ‘`SELECT c`’를 생략한 형태로 사용이 가능
-   **네이티브 SQL을 통한 조회**
    -   Spring Data JDBC와 마찬가지로 JPA 역시 네이티브 SQL 쿼리를 작성해서 사용 가능
    -   (2-2)의 `nativeQuery` 애트리뷰트 값을 `true`로 설정하면 `value` 애트리뷰트에 적용한 SQL 쿼리가 적용

> #### 참고 사항
> 
> `Spring Data JDBC의 @Query` vs `Spring Data JPA의 @Query`  
> Spring Data JDBC와 Spring Data JPA에서 사용하는 `@Query` 애너테이션은 이름은 같지만 패키지 자체가 다르다.  
> 만약, Starter 모듈이 둘 다 의존 라이브러리에 포함되어 있는 경우, 패키지 경로를 혼동하지 않도록 주의해야 한다.
> 
> -   `Spring Data JDBC의 @Query` 애너테이션 패키지 경로
>     -   `import org.springframework.data.jdbc.repository.query.Query`
> -   Spring Data JPA의 @Query\` 애너테이션 패키지 경로
>     -   `org.springframework.data.jpa.repository.Query`