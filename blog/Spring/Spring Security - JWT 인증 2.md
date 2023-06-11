태그: #Spring 
연결문서: [Spring Security - JWT 인증 3](Spring%20Security%20-%20JWT%20인증%203.md)

#  JWT(Json Web Token) 개요

## JWT 생성 및 검증 테스트

### JWT 생성 및 검증 테스트를 위한 프로젝트 설정
- Spring Boot 기반의 템플릿 프로젝트 생성 및 의존 라이브러리 추가
    - JWT 생성 및 검증 테스트를 수행하기 위해 필요한 JWT 라이브러리
    - JWT를 위한 대표적인 라이브러리에는 jjwt와 Java JWT가 있으며, Java 진영에서는 jjwt를 가장 많이 사용함
<br>

```java
dependencies {
  // (1)
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'

  // (2)
	implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
	runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
	runtimeOnly	'io.jsonwebtoken:jjwt-jackson:0.11.5'
}
```

<br>

### JWT 생성

#### JWT 생성 기능 구현
- JWT(JSON Web Token)를 생성하고 검증하는 역할을 수행하는 클래스 및 정의하고, JWT 생성 메서드 추가
<br>

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.io.Encoders;
import io.jsonwebtoken.security.Keys;

import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Date;
import java.util.Map;

public class JwtTokenizer {
    // (1)
    public String encodeBase64SecretKey(String secretKey) {
        return Encoders.BASE64.encode(secretKey.getBytes(StandardCharsets.UTF_8));
    }

    // (2)
    public String generateAccessToken(Map<String, Object> claims,
                                      String subject,
                                      Date expiration,
                                      String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey); // (2-1)

        return Jwts.builder()
                .setClaims(claims)          // (2-2)
                .setSubject(subject)        // (2-3)
                .setIssuedAt(Calendar.getInstance().getTime())   // (2-4)
                .setExpiration(expiration)  // (2-5)
                .signWith(key)              // (2-6)
                .compact();                 // (2-7)
    }

    // (3)
    public String generateRefreshToken(String subject, Date expiration, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        return Jwts.builder()
                .setSubject(subject)
                .setIssuedAt(Calendar.getInstance().getTime())
                .setExpiration(expiration)
                .signWith(key)
                .compact();
    }
    
    ...

    // (4)
    private Key getKeyFromBase64EncodedKey(String base64EncodedSecretKey) {
        byte[] keyBytes = Decoders.BASE64.decode(base64EncodedSecretKey);  // (4-1)
        Key key = Keys.hmacShaKeyFor(keyBytes);    // (4-2)

        return key;
    }
}
```

<br>

- (1)의 encodeBase64SecretKey() 메서드는 Plain Text 형태인 Secret Key의 byte[]를 Base64 형식의 문자열로 인코딩해줌
    - jjwt가 버전업 되면서 Plain Text 자체를 Secret Key로 사용하는 것은 암호학(cryptographic)적인 작업에 사용되는 Key가 항상 바이너리(byte array)라는 사실과 맞지 않는 것을 감안하여 Plain Text 자체를 Secret Key로 사용하는 것을 권장하지 않고 있음
- (2)의 generateAccessToken()은 인증된 사용자에게 JWT를 최초로 발급해 주기 위한 JWT 생성 메서드
    - (2-1)에서는 Base64 형식 Secret Key 문자열을 이용해 Key(java.security.Key) 객체를 얻음
    - (2-2)의 setClaims()에는 JWT에 포함시킬 Custom Claims를 추가합니다. Custom Claims에는 주로 인증된 사용자와 관련된 정보를 추가함
    - (2-3)의 setSubject()에는 JWT에 대한 제목을 추가함
    - (2-4)의 setIssuedAt()에는 JWT 발행 일자를 설정하며 파라미터 타입은 java.util.Date 타입
    - (2-5)의 setExpiration()에는 JWT의 만료일시를 지정하며, 파라미터 타입은 java.util.Date 타입
    - (2-6)의 signWith()에 서명을 위한 Key(java.security.Key) 객체를 설정함
    - (2-7)의 compact()를 통해 JWT를 생성하고 직렬화함
- (3)의 generateRefreshToken() 메서드는 Access Token이 만료되었을 경우, Access Token을 새로 생성할 수 있게 해주는 Refresh Token을 생성하는 메서드
    - Refresh Token의 경우 Access Token을 새로 발급해 주는 역할을 하는 Token이기 때문에 별도의 Custom Claims는 추가할 필요가 없음
- (4)의 getKeyFromBase64EncodedKey() 메서드는 JWT의 서명에 사용할 Secret Key를 생성함
    - (4-1)의 Decoders.BASE64.decode() 메서드는 Base64 형식으로 인코딩 된 Secret Key를 디코딩한 후, byte array를 반환함
    - (4-2)의 Keys.hmacShaKeyFor() 메서드는 key byte array를 기반으로 적절한 HMAC 알고리즘을 적용한 Key(java.security.Key) 객체를 생성함

<br>

#### JWT 생성 기능 테스트

```java
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInstance;

import java.util.*;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.Matchers.notNullValue;

@TestInstance(TestInstance.Lifecycle.PER_CLASS) // 테스트 학습이 핵심이 아니므로 검색을 통한 학습이 필요함
public class JwtTokenizerTest {
    private static JwtTokenizer jwtTokenizer;
    private String secretKey;
    private String base64EncodedSecretKey;

    // (1)
    @BeforeAll
    public void init() {
        jwtTokenizer = new JwtTokenizer();
        secretKey = "kevin1234123412341234123412341234";  // encoded "a2V2aW4xMjM0MTIzNDEyMzQxMjM0MTIzNDEyMzQxMjM0"

        base64EncodedSecretKey = jwtTokenizer.encodeBase64SecretKey(secretKey);
    }

    // (2)
    @Test
    public void encodeBase64SecretKeyTest() {
        System.out.println(base64EncodedSecretKey);

        assertThat(secretKey, is(new String(Decoders.BASE64.decode(base64EncodedSecretKey))));
    }

    // (3)
    @Test
    public void generateAccessTokenTest() {
        Map<String, Object> claims = new HashMap<>();
        claims.put("memberId", 1);
        claims.put("roles", List.of("USER"));

        String subject = "test access token";
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.MINUTE, 10);
        Date expiration = calendar.getTime();

        String accessToken = jwtTokenizer.generateAccessToken(claims, subject, expiration, base64EncodedSecretKey);

        System.out.println(accessToken);

        assertThat(accessToken, notNullValue());
    }

    // (4)
    @Test
    public void generateRefreshTokenTest() {
        String subject = "test refresh token";
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.HOUR, 24);
        Date expiration = calendar.getTime();

        String refreshToken = jwtTokenizer.generateRefreshToken(subject, expiration, base64EncodedSecretKey);

        System.out.println(refreshToken);

        assertThat(refreshToken, notNullValue());
    }
}
```

<br>

- (1)에서 테스트에 사용할 Secret Key를 Base64 형식으로 인코딩한 후, 인코딩 된 Secret Key를 각 테스트 케이스에서 사용함
- (2)에서는 Plain Text인 Secret Key가 Base64 형식으로 인코딩이 정상적으로 수행이 되는지 테스트함
    - Base64 형식으로 인코딩 된 Secret Key를 디코딩한 값이 원본 Plain Text Secret Key가 일치하는지를 테스트함
- (3)에서는 JwtTokenizer가 Access Token을 정상적으로 생성하는지 테스트함
    - JWT는 생성할 때마다 그 값이 바뀌기 때문에 우선 생성된 Access Token이 null이 아닌지 여부만 테스트함
    - 생성 과정에서 Exception이 발생하지 않았기 때문에 정상적으로 생성이 되었다고 봐도 무방하며, 더 정확한 테스트는 JWT의 서명 검증에서 확인할 수 있음
- (4)에서는 JwtTokenizer가 Refresh Token을 정상적으로 생성하는지 테스트함
    - Custom Claims가 필요하지 않다는 것 외에는 Access Token과 테스트 과정은 동일함

<br>

### JWT 검증

#### JWT 검증 기능 구현
- JWT는 JWT에 포함된 Signature를 검증함으로써 JWT의 위/변조 여부를 확인할 수 있음
- jjwt에서는 JWT를 생성할 때 서명에 사용된 Secret Key를 이용해 내부적으로 Signature를 검증한 후, 검증에 성공하면 JWT를 파싱 해서 Claims를 얻을 수 있음
<br>

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.io.Encoders;
import io.jsonwebtoken.security.Keys;

import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Calendar;
import java.util.Date;
import java.util.Map;

public class JwtTokenizer {
    ...

    public void verifySignature(String jws, String base64EncodedSecretKey) {
        Key key = getKeyFromBase64EncodedKey(base64EncodedSecretKey);

        Jwts.parserBuilder()
                .setSigningKey(key)     // (1)
                .build()
                .parseClaimsJws(jws);   // (2)
    }

    ...
}
```

<br>

- (1)의 setSigningKey() 메서드로 서명에 사용된 Secret Key를 설정함
- (2)의 parseClaimsJws() 메서드로 JWT를 파싱해서 Claims를 얻음
    - verifySignature() 메서드는 Signature를 검증하는 용도로 Claims를 리턴할 필요는 없음
    - 파라미터로 사용한 jws는 Signature가 포함된 JWT라는 의미이

<br>

#### JWT 검증 기능 테스트

```java
import io.jsonwebtoken.ExpiredJwtException;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInstance;

import java.util.*;
import java.util.concurrent.TimeUnit;

import static org.junit.jupiter.api.Assertions.assertDoesNotThrow;
import static org.junit.jupiter.api.Assertions.assertThrows;

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class JwtTokenizerTest {
    private static JwtTokenizer jwtTokenizer;
    private String secretKey;
    private String base64EncodedSecretKey;

    ...

    // (1)
    @DisplayName("does not throw any Exception when jws verify")
    @Test
    public void verifySignatureTest() {
        String accessToken = getAccessToken(Calendar.MINUTE, 10);
        assertDoesNotThrow(() -> jwtTokenizer.verifySignature(accessToken, base64EncodedSecretKey));
    }

    // (2)
    @DisplayName("throw ExpiredJwtException when jws verify")
    @Test
    public void verifyExpirationTest() throws InterruptedException {
        String accessToken = getAccessToken(Calendar.SECOND, 1);
        assertDoesNotThrow(() -> jwtTokenizer.verifySignature(accessToken, base64EncodedSecretKey));

        TimeUnit.MILLISECONDS.sleep(1500);

        assertThrows(ExpiredJwtException.class, () -> jwtTokenizer.verifySignature(accessToken, base64EncodedSecretKey));
    }
    
    ...

    private String getAccessToken(int timeUnit, int timeAmount) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("memberId", 1);
        claims.put("roles", List.of("USER"));

        String subject = "test access token";
        Calendar calendar = Calendar.getInstance();
        calendar.add(timeUnit, timeAmount);
        Date expiration = calendar.getTime();
        String accessToken = jwtTokenizer.generateAccessToken(claims, subject, expiration, base64EncodedSecretKey);

        return accessToken;
    }
}
```

<br>

- (1)에서는 우리가 구현한 JwtTokenizer의 verifySignature() 메서드가 Signature를 잘 검증하는지 테스트함
    - 생성된 JWT를 verifySignature()로 전달해서 Exception이 발생하지 않는다면 Signature에 대한 검증이 잘 수행된 것으로 볼 수 있음
- (2)에서는 JWT 생성 시 지정한 만료일시가 지나면 JWT가 정말 만료되는지를 테스트함
    - 생성되는 JWT의 만료 주기를 아주 짧게 준 후에 첫 번째 Signature 검증을 수행하고, 만료일시가 지나도록 지연시간을 준 뒤, 두 번째 Signature 검증을 수행했을 경우 ExpiredJwtException이 발생하면 JWT가 정상적으로 만료된다고 볼 수 있음