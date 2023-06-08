태그: #Spring 
연결문서: [Spring Security 기본 7](Spring%20Security%20기본%207.md)

# Spring Security

## DelegatingPasswordEncoder
- DelegatingPasswordEncoder는 Spring Security에서 지원하는 PasswordEncoder 구현 객체를 생성해 주는 컴포넌트로써 DelegatingPasswordEncoder를 통해 애플리케이션에서 사용할 PasswordEncoder를 결정하고, 결정된 PasswordEncoder로 사용자가 입력한 패스워드를 단방향으로 암호화를 해줌

<br>

### DelegatingPasswordEncoder 도입 전 문제점
- 스프링 시큐리티 5.0 이전 버전에서는 평문 텍스트(Plain Text) 패스워드를 그대ㅑ로 사용하는 NoOpPasswordEncoder가 디폴트 PasswordEncoder로 고정되어 있었지만, 다음과 같은 문제를 해결하기 위해 DelegatingPasswordEncoder를 도입해서 조금 더 유연한 구조로 PasswordEncoder를 사용할 수 있게 됨
    1. 패스워드 인코딩 방식을 마이그레이션 하기 쉽지 않은 오래된 방식을 사용하는 경우
        - 패스워드 단방향 암호화에 사용되는 hash 알고리즘은 시간이 지나면서 보다 더 안전한 hash 알고리즘이 지속적으로 고안되고 있기 때문에 항상 고정된 암호화 방식을 사용하는 것은 바람직한 사용 방식이 아님
        - 보안에 취약한 오래된 방식의 암호화 방식을 고수하는 애플리케이션은 해커의 좋은 타깃이 될 수 있음
    2. 스프링 시큐리티는 프레임워크이기 때문에 하위 호환성을 보장하지 않는 업데이트를 자주 할 수 없음
        - 오래된 하위 버전의 기술이 언젠가 Deprecated 되는 것처럼 보안에 취약한 오래된 방식의 암호화 알고리즘 역시 언제까지 관리 대상이 되지는 않음

<br>

### DelegatingPasswordEncoder의 장점
- DelegatingPasswordEncoder를 사용해 다양한 방식의 암호화 알고리즘을 적용할 수 있는데, 사용하고자 하는 암호화 알고리즘을 특별히 지정하지 않는다면 Spring Security에서 권장하는 최신 암호화 알고리즘을 사용하여 패스워드를 암호화할 수 있도록 해줌
- 패스워드 검증에 있어서 레거시 방식의 암호화 알고리즘으로 암호화된 패스워드의 검증은 지원함
- Delegating이라는 표현에서도 DelegatingPasswordEncoder의 특징이 잘 드러나듯이 나중에 암호화 방식을 변경하고 싶다면 언제든지 암호화 방식을 변경할 수 있음
    - 이런 경우, 기존에 암호화되어 저장된 패스워드에 대한 마이그레이션 작업이 진행되어야 함

<br>

### DelegatingPasswordEncoder를 이용한 PasswordEncoder 생성
- DelegatingPasswordEncoder를 사용해 PasswordEncoder를 생성하는 방법
    - PasswordEncoderFactories로 만들 수 있음
    - PasswordEncoderFactories.createDelegatingPasswordEncoder();를 통해 DelegatingPasswordEncoder의 객체를 생성하고, 내부적으로 DelegatingPasswordEncoder가 다시 적절한 PasswordEncoder 객체를 생성함

```java
PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

<br>

### Custom DelegatingPasswordEncoder 생성
- Spring Security에서 지원하는 PasswordEncoderFactories 클래스를 이용하면 기본적으로 Spring Security에서 권장하는 PasswordEncoder를 사용할 수 있지만 필요한 경우, DelegatingPasswordEncoder로 직접 PasswordEncoder를 지정해서 Custom DelegatingPasswordEncoder를 사용할 수 있음
- Map encoders에 원하는 유형의 PasswordEncoder를 추가하여 DelegatingPasswordEncoder의 생성자로 넘겨주면 디폴트로 지정(idForEncode)한 PasswordEncoder를 사용할 수 있음

```java
String idForEncode = "bcrypt";
Map encoders = new HashMap<>();
encoders.put(idForEncode, new BCryptPasswordEncoder());
encoders.put("noop", NoOpPasswordEncoder.getInstance());
encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
encoders.put("scrypt", new SCryptPasswordEncoder());
encoders.put("sha256", new StandardPasswordEncoder());

PasswordEncoder passwordEncoder = new DelegatingPasswordEncoder(idForEncode, encoders);
```

<br>

### 암호화된 Password Format
- 암호화된 패스워드의 포맷
    - {id}encodedPassword
- BCryptPasswordEncoder로 암호화할 경우
    - {bcrypt}&#36;2a&#36;10&#36;dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
        - PasswordEncoder id는 “bcrypt”
        - encodedPassword는 "&#36;2a&#36;10&#36;dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG"
- Pbkdf2PasswordEncoder로 암호화할 경우
    - {pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc
        - PasswordEncoder id는 “pbkdf2”
        - encodedPassword는 “5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc”
- SCryptPasswordEncoder로 암호화할 경우
    - {scrypt}&#36;e0801&#36;8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw&#61;&#61;&#36;OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=
        - PasswordEncoder id는 “scrypt”
        - encodedPassword는 “&#36;e0801&#36;8bWJaSu2IKSn9Z9kM+TPXfOc/9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw&#61;&#61;&#36;OAOec05+bXxvuu/1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=”
- StandardPasswordEncoder로 암호화할 경우
    - {sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0
        - PasswordEncoder id는 “sha256”
        - encodedPassword는 “97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0”