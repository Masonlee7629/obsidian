태그: #Spring
연결문서: [Spring MVC - Spring Data JPA 4](Spring%20MVC%20-%20Spring%20Data%20JPA%204.md)

# Spring Data JPA

## 엔티티 간의 연관 관계 매핑

### 연관 관계 매핑이란

연관 관계 매핑이란 엔티티 클래스 간의 관계를 만들어주는 것을 말한다.  
  

연관 관계 매핑은 참조하는 방향성을 기준으로 생각했을 때 **단방향 연관 관계**와 **양방향 연관 관계**로 구분할 수 있다.  
  

그리고, 엔티티 간에 참조할 수 있는 객체의 수에 따라서 **일대다(1:N)**, **다대일(N:1)**, **다대다(N:N)**, **일대일(1:1)** 의 연관 관계로 나눌 수 있다.

#### 단방향 연관 관계

[##_Image|kage@UvQ6w/btsaRxmw6k5/ksSRnB2oMkLg0OfuGKpnPK/img.png|CDM|1.3|{"originWidth":1080,"originHeight":560,"style":"alignCenter","width":null}_##]

한쪽 클래스만 다른 쪽 클래스의 참조 정보를 가지고 있는 관계를 **단방향 연관 관계**라고 한다.  
  

위 그림은 Order 클래스가 Member 객체를 가지고 있으므로 Member 클래스를 참조할 수 있다.  
  

하지만 Member 클래스는 Order 클래스에 대한 참조 값이 없으므로 Member 클래스는 Order 클래스의 정보를 알 수 없다.

#### 양방향 연관 관계

[##_Image|kage@dFHVkC/btsaROO4Afd/w9GmAFK3llHpPwIqpWLNAK/img.png|CDM|1.3|{"originWidth":1080,"originHeight":560,"style":"alignCenter","width":null}_##]

양쪽 클래스가 서로의 참조 정보를 가지고 있는 관계를 **양방향 연관 관계**라고 한다.  
  

위 그림에선 Member 클래스가 Order 객체를 원소로 포함하고 있는 List 객체를 가지고 있고, Order 클래스를 참조할 수 있다.  
  

Order 클래스는 Member 객체를 가지고 있으므로 Member 클래스를 참조할 수 있다.  
  

두 클래스가 서로의 객체를 참조할 수 있으므로 서로의 정보를 알 수 있다.

-   JPA는 단방향 연관 관계와 양방향 연관 관계를 모두 지원하지만, **Spring Data JDBC는 단방향 연관 관계만 지원**한다.

#### 일대다 단방향 연관 관계

[##_Image|kage@EAIpg/btsaRwnBzDG/xaZmBK7HYSUKb1Ke3KHol1/img.png|CDM|1.3|{"originWidth":1080,"originHeight":560,"style":"alignCenter","width":null}_##]

일대다 관계란 **일(1)에 해당하는 클래스가 다에 해당하는 객체를 참조할 수 있는 관계**를 의미한다.  
  

테이블 간의 관계에서는 일대다 중에서 '다'에 해당하는 테이블에서 '일'에 해당하는 테이블의 기본키를 외래키로 가진다.  
  

하지만 클래스 상에서는 **테이블 관계에서 외래키에 해당하는 클래스의 참조값을 가지고 있지 않기 때문에 일반적인 테이블 간의 관계를 정상적으로 표현하지 못한다.**  
  

따라서 일대다 단방향 매핑 하나만 사용하는 경우는 드물고, **다대일 단방향 매핑을 먼저 한 후에, 필요한 경우, 일대다 단방향 매핑을 추가해서 양방향 연관 관계를 만드는 것이 일반적이다.**

#### 다대일 연관 관계

[##_Image|kage@lJ9WO/btsatVP00w0/Ccx0YbKGvt4ma7V6QRTT5k/img.png|CDM|1.3|{"originWidth":1080,"originHeight":560,"style":"alignCenter","width":null}_##]

다대일 관계란 **다(N)에 해당하는 클래스가 일(1)에 해당하는 객체를 참조할 수 있는 관계**를 의미한다.  
  

위 그림은 다대일 단방향 연관 관계를 가지고 있다.  
  

테이블 관계에서 ORDERS 테이블이 MEMBER 테이블의 member\_id를 외래키로 가지듯이 Order 클래스가 Member 객체를 외래키처럼 가지고 있다.  
  

다대일 단방향 매핑은 테이블 간의 관계처럼 자연스러운 매핑 방식이기 때문에 JPA의 엔티티 연관 관계 중에서 가장 기본으로 사용되는 매핑 방식이다.

#### 다대일 연관 관계 매핑 방법

1.  (1)과 같이 `@ManyToOne` 애너테이션으로 다대일의 관계를 명시한다.
2.  (2)와 같이 `@JoinColumn` 애너테이션으로 ORDERS 테이블에서 외래키에 해당하는 컬럼명을 입력한다.

```
@NoArgsConstructor
@Getter
@Setter
@Entity(name = "ORDERS")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long orderId;

    ...
    ...

    @ManyToOne   // (1)
    @JoinColumn(name = "MEMBER_ID")  // (2)
    private Member member;

    ...
    ...
}
```

#### 다대일 매핑에 일대다 매핑 추가하기

다대일 매핑이 되어 있는 상태에서 필요에 의해 **일대다 매핑을 추가해 양방향 관계를 만든다.**

1.  (1)의 `@OneToMany` 는 일대다의 관계를 명시한다.
2.  `mappedBy` 애프리뷰트를 사용한다.
    -   일대다 단방향 매핑의 경우, `mappedBy` 애트리뷰트의 값이 필요없음
        -   `mappedBy` 는 참조할 대상이 필요하지만 일대다 단방향 매핑은 참조할 대상이 없음
    -   `mappedBy` 의 값은 관계를 소유하고 있는 필드를 지정
        -   Order 클래스에서 외래키의 역할을 하는 것이 member 필드라서 member를 지정

```
@NoArgsConstructor
@Getter
@Setter
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long memberId;

    ...
    ...

        // (1)
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    ...
    ...
```

#### 다대다 연관 관계

테이블 설계 시 다대다의 관계는 중간에 테이블을 하나 추가해서 두 개의 일대다 관계를 만들어주는 것이 일반적이다.  
  

**일대다 단방향 매핑을 외래키를 포함하지 않기 때문에 자주 사용되지 않는 매핑 방법**이다.  
  

따라서 두 개의 다대일 매핑을 사용하고, 객체 그래프 탐색으로 원하는 객체를 조회할 수 없다면 일대다 양방향 매핑을 추가하면 된다.

#### 일대일 연관 관계

일대일 연관 관계 매핑은 다대일 단방향 연관 관계 매핑과 매핑 방법이 동일하다.  
  

차이점은 `@ManyToOne` 애너테이션이 아닌 `@OneToOne` 애너테이션을 사용한다는 것이다.  
  

일대일 단방향 매핑에 양방향 매핑을 추가하는 방법도 다대일에 일대다 매핑을 추가하는 방식과 동일하며 `@ManyToOne` 애너테이션이 아닌 `@OneToOne` 애너테이션을 사용한다.

### 엔티티 간의 연관 관계 매핑 권장 방법

-   일대다 매핑은 사용하지 않는다.
-   우선적으로 다대일 단방향 매핑부터 적용한다.
-   다대일 단방향 매핑을 통해 객체 그래프 탐색으로 조회할 수 없는 정보가 있을 경우, 양방향 매핑을 적용한다.