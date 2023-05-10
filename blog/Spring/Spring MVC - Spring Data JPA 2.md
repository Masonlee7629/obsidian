태그: #Spring
연결문서: [Spring MVC - Spring Data JPA 3](Spring%20MVC%20-%20Spring%20Data%20JPA%203.md)

# Spring Data JPA

## 엔티티 매핑

JPA를 이용해 데이터베이스의 테이블과 상호 작용(데이터 저장, 수정, 조회, 삭제 등)하기 위해 가장 우선되는 작업은 데이터베이스의 테이블과 엔티티 클래스 간의 매핑 작업니다.  
  

엔티티 매핑 작업은 크게 객체와 테이블 간의 매핑, 기본키 매핑, 필드(멤버 변수)와 컬럼 간의 매핑, 엔티티 간의 연관 관계 매핑 등으로 나눌 수 있다.

### 엔티티 테이블 간의 매핑

```
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity(name = "USERS") // (1)
@Table(name = "USERS") // (2)
public class Member {
    @Id
    private Long memberId;
}
```

#### `@Entity` 애너테이션

-   애트리뷰트
    -   name
        -   엔티티 이름을 설정할 수 있다.
        -   name 애트리뷰트를 설정하지 않으면 기본값으로 클래스 이름을 엔티티 이름으로 사용한다.

#### `@Table` 애너테이션

-   애트리뷰트
    -   name
        -   테이블 이름을 설정할 수 있다.
        -   name 애트리뷰트를 설정하지 않으면 기본값으로 클래스 이름을 테이블 이름으로 사용한다.
        -   `@Table` 애너테이션은 옵션이며, 추가하지 않을 경우 클래스 이름을 테이블 이름으로 사용한다.
        -   주로 테이블 이름이 클래스 이름과 달라야 할 경우에 추가한다.

#### 주의사항

-   `@Table` 애너테이션은 옵션이지만 `@Entity`와 `@ID` 애너테이션은 필수이다.
-   `@Entity`와 `@ID` 애너테이션은 함께 사용한다.
    -   `@Entity` 애너테이션만 추가하고 식별자 역할을 하는 필드(멤버 변수)에 `@ID` 애너테이션을 추가하지 않으면 다음과 같은 에러가 발생한다.
        -   `Caused by: org.hibernate.AnnotationException: No identifier specified for entity:...`
-   파라미터가 없는 기본 생성자는 필수로 추가한다.
    -   Spring Data Jpa의 기술을 적용할 때 기본 생성자가 없는 경우 에러가 발생하는 경우가 있기 때문에 기본 생성자는 습관적으로 추가하는 것이 좋다.
-   중복되는 엔티티 클래스가 없고, 테이블 이름이 클래스의 이름과 같을 경우에는 `@Entity`와 `@Table` 애너테이션에 `name` 애트리뷰트를 지정하지 않고, 클래스 이름으로 사용하는 것을 권장한다.

### 기본키 매핑

데이터베이스의 테이블에 기본키 설정은 필수라고 할 수 있다.  
  

JPA에서는 기본적으로 `@ID` 애너테이션을 추가한 필드가 기본 키 컬럼이 되는데, JPA에서는 기본키를 어떤 방식으로 생성해 줄지에 대한 다양한 전략을 지원한다.

-   기본키 직접 할당
    -   애플리케이션 코드 상에서 기본키를 직접 할당해주는 방식
-   기본키 자동 생성
    -   IDENTITY
        -   기본키 생성을 데이터베이스에 위임하는 전략
        -   데이터베이스에서 기본키를 생성해주는 대표적인 방식은 MySQL의 AUTO\_INCREMENT 기능을 통해 자동 증가 숫자를 기본키로 사용하는 방식이 있음
    -   SEQUENCE
        -   데이터베이스에서 제공하는 시퀀스를 사용해서 기본키를 생성하는 전략
    -   TABLE
        -   별도의 키 생성 테이블을 사용하는 전략

#### 기본키 직접 할당 전략

-   기본키를 직접 할당해서 엔티티를 저장한다.

```
@NoArgsConstructor
@Getter
@Entity
public class Member {
    @Id   // (1)
    private Long memberId;

    public Member(Long memberId) {
        this.memberId = memberId;
    }
}
```

#### IDENTITY 전략

-   IDENTITY 전략은 데이터베이스에서 기본키를 대신 생성해 준다

```
@NoArgsConstructor
@Getter
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // (1)
    private Long memberId;

    public Member(Long memberId) {
        this.memberId = memberId;
    }
}
```

#### SEQUENCE 전략

-   SEQUENCE 전략은 데이터베이스 시퀀스를 이용한다.

```
@NoArgsConstructor
@Getter
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)  // (1)
    private Long memberId;

    public Member(Long memberId) {
        this.memberId = memberId;
    }
}
```

#### AUTO 전략

-   JPA가 데이터베이스의 Dialect에 따라서 적절한 전략을 자동으로 선택한다.
    -   Dialect는 표준 SQL 등이 아닌 특정 데이터베이스에 특화된 고유한 기능을 의미하며, 만일 JPA가 지원하는 표준 문법이 아닌 특정 데이터베이스에 특화된 기능을 사용할 경우 Dialect가 처리해준다.

```
@NoArgsConstructor
@Getter
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)  // (1)
    private Long memberId;

    public Member(Long memberId) {
        this.memberId = memberId;
    }
}
```

### 필드(멤버 변수)와 컬럼 간의 매핑

#### Member 엔티티 클래스 필드와 컬럼 간의 매핑 예시

-   (1)에서 사용한 `@Column` 애너테이션은 필드와 컬럼을 매핑해주는 애너테이션이다. 만약, `@Column` 애너테이션이 없고, 필드만 정의되어 있다면 JPA는 기본적으로 이 필드가 테이블의 컬럼과 매핑되는 필드라고 간주하게 된다. 또한, `@Column` 애너테이션이 사용되는 애트리뷰트의 값은 디폴트 값이 모두 적용된다.
    -   nullable
        -   컬럼에 `null` 값을 허용할 지 여부를 지정
        -   디폴트 값은 `true`
        -   email 주소는 일반적으로 ID로 많이 사용되며, 따라서 필수 항목이기 때문에 nullable 값을 `false`로 지정함
    -   updatable
        -   컬럼 데이터를 수정할 수 있는지 여부를 지정
        -   디폴트 값을 `true`
        -   email 주소가 사용자 ID 역할을 한다고 가정할 때, 한 번 등록되면 수정이 불가능하도록 하기 위해 updatable 값을 `false`로 지정
    -   unique
        -   하나의 컬럼에 unique 유니크 제약 조건을 설정
        -   디폴트 값은 `false`
        -   email의 경우 고유한 값이어야 하므로 unique 값을 `true`로 지정
-   `@Column` 애너테이션이 생략되었거나 애트리뷰트가 기본값을 사용할 경우 주의사항
    -   int나 long 같은 원시 타입일 경우, `@Column` 애너테이션이 생략되면 기본적으로 `nullable=false`이다.  
        만약 개발자가 `int price not null` 이라는 조건으로 컬럼을 설정하길 원하는데 nullable에 대한 명시적인 설정 없이 `@Column` 애너테이션만 추가하면 `nullable=true`가 기본값이 되기 때문에 테이블에는 `int price` 와 같이 설정될 것이다.  
        따라서 개발자가 의도하는 바가 `int price not null`일 경우, `@Column(nullable=false)`라고 명시적으로 지정하거나 `@Column` 애너테이션 자체를 사용하지 않는 것이 권장된다.

```
@NoArgsConstructor
@Getter
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberId;

        // (1)
    @Column(nullable = false, updatable = false, unique = true)
    private String email;

        ...
        ...

    public Member(String email) {
        this.email = email;
    }
}
```

### 엔티티와 테이블 매핑 권장 사용 방법

-   클래스 이름 중복 등의 특별한 이유가 없다면 `@Entity`와 `@Id` 애너테이션만 추가한다.
    -   엔티티 클래스가 테이블 스키마 명세의 역할을 하길 바란다면 @Table 애너테이션에 테이블명을 지정해줄 수 있다.
-   기본키 생성 전략은 데이터베이스에서 지원해주는 AUTO\_INCREMENT 또는 SEQUENCE를 이용할 수 있도록 IDENTITY 또는 SEQUENCE 전략을 사용하는 것이 좋다.
-   `@Column` 정보를 명시적으로 모두 지정하는 것은 번거롭긴하지만 다른 누군가가 엔티티 클래스 코드를 확인하더라도 테이블 설계가 어떤식으로 되어 있는지 한눈에 알 수 있다는 장점이 있다.
-   엔티티 클래스 필드 타입이 Java의 원시타입일 경우, @Column 애너테이션을 생략하지 말고, 최소한 nullable=false 설정을 하는게 좋다.
-   `@Enumerated` 애너테이션을 사용할 때 `EnumType.ORDINAL` 을 사용할 경우, enum의 순서가 뒤바뀔 가능성도 있으므로 처음부터 `EnumType.ORDINAL` 대신에 `EnumType.STRING` 을 사용하는 것이 좋습니다.
-   `@Transient` 애너테이션은 필드에 추가하면 테이블 컬럼과 매핑하지 않겠다는 의미로 JPA가 인식한다.
    -   데이터베이스에 저장하지 않고, 조회할 때도 매핑되지 않는다.
    -   주로 임시 데이터를 메모리에서 사용하기 위한 용도로 사용한다.