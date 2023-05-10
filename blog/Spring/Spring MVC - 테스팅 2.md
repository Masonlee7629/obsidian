태그: #Spring
연결문서: [Spring MVC - 테스팅 3](Spring%20MVC%20-%20테스팅%203.md)

# 테스팅(Testing)

## JUnit을 사용한 단위 테스트

### JUnit이란?

JUnit은 Java 언어로 만들어진 애플리케이션을 테스트하기 위한 오픈 소스 테스트 프레임워크이며, Spring Boot의 디폴트 테스트 프레임워크이다.

### JUnit 기본 작성법

Spring Boot Initializr에서 Gradle 기반의 Spring Boot 프로젝트를 생성하고 오픈하면 기본적으로 `'src/test'` 디렉토리가 생성된다.  
  

Spring Boot Intializr를 이용해서 프로젝트를 생성하면 기본적으로 `testImplementation >'org.springframework.boot:spring-boot-starter-test'` 스타터가 포함되며, JUnit도 포함이 되어 있다.  
  

따라서 별다른 설정없이 JUnit을 사용할 수 있다.

#### JUnit을 사용한 테스트 케이스의 기본 구조

```
import org.junit.jupiter.api.Test;

public class JunitDefaultStructure {
        // (1)
    @Test
    public void test1() {
        // 테스트 하고자 하는 대상에 대한 테스트 로직 작성
    }

        // (2)
    @Test
    public void test2() {
        // 테스트 하고자 하는 대상에 대한 테스트 로직 작성
    }

        // (3)
    @Test
    public void test3() {
        // 테스트 하고자 하는 대상에 대한 테스트 로직 작성
    }
}
```

위 코드는 테스트 케이스에 JUnit을 적용하는 기본 구조이다.  
  

(1), (2), (3)과 같이 애플리케이션에서 테스트 하고자 하는 대상(Target)이 있으면 `public void test1(){…}` 같은 void 타입의 메서드 하나 만들고, `@Test` 애너테이션을 추가해준다.  
  

그리고 그 내부에 테스트 하고자 하는 대상 메서드에 대한 테스트 로직을 작성한다.

#### Assertion 메서드 사용하기

JUnit에서는 Assertion과 관련된 다양한 메서드를 사용해서 테스트 대상에 대한 Assertion을 진행할 수 있다.

-   `assertEquals()`

```
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class HelloJUnitTest {
    @DisplayName("Hello JUnit Test")  // (1)
    @Test
    public void assertionTest() {
        String expected = "Hello, JUnit";
        String actual = "Hello, JUnit";

        assertEquals(expected, actual); // (2)
    }
}
```

`assertEquals()` 메서드를 사용하면 기대하는 값과 실제 결과 값이 같은지를 검증할 수 있다.  
  

(1)은 테스트 케이스를 실행시켰을 때, 실행 결과 창에 표시되는 이름을 지정하는 부분이다.  
(2)에서는 기대하는 문자열(expected)과 실제 결과 값(autual)이 일치하는지를 검증한다.

-   `assertNotNull()`

```
import com.codestates.CryptoCurrency;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertNotNull;

public class AssertionNotNullTest {

    @DisplayName("AssertionNull() Test")
    @Test
    public void assertNotNullTest() {
        String currencyName = getCryptoCurrency("ETH");

                // (1)
        assertNotNull(currencyName, "should be not null");
    }

    private String getCryptoCurrency(String unit) {
        return CryptoCurrency.map.get(unit);
    }
}
```

`assertNotNull()` 메서드를 사용하면 테스트 대상 객체가 `null` 이 아닌지를 테스트할 수 있다.  
  

`assertNotNull()` 메서드의 첫 번째 파라미터는 테스트 대상 객체이고, 두 번째 파라미터는 테스트에 실패했을 때, 표시할 메시지이다.

-   `assertThrows()`

```
import com.codestates.CryptoCurrency;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class AssertionExceptionTest {

    @DisplayName("throws NullPointerException when map.get()")
    @Test
    public void assertionThrowExceptionTest() {
        // (1)
        assertThrows(NullPointerException.class, () -> getCryptoCurrency("XRP"));
    }

    private String getCryptoCurrency(String unit) {
        return CryptoCurrency.map.get(unit).toUpperCase();
    }
}
```

`assertThrows()`를 사용하면 호출한 메서드의 동작 과정 중에 예외가 발생하는지 테스트할 수 있다.  
  

(1)에서 `assertThrows()`의 첫 번째 파라미터에는 발생이 기대되는 예외 클래스를 입력하고, 두 번째 파라미터인 람다 표현식에서는 테스트 대상 메서드를 호출한다.  
  

`assertThrows()` 를 사용해서 예외를 테스트 하기 위해서는 예외 클래스의 상속 관계를 이해한 상태에서 테스트 실행 결과를 예상해야 된다.

#### 테스트 케이스 실행 전 전처리

테스트 케이스를 실행하기 전에 어떤 객체나 값에 대한 초기화 작업 등의 전처리 과정을 해야할 경우가 많다. 이 경우 JUnit에서 사용할 수 있는 애너테이션이 `@BeforeEach`와 `@BeforeAll()`이다.

-   `@BeforeEach`
    -   `@BeforeEach` 애너테이션을 추가한 메서드는 테스트 케이스가 각각 실행될 때 마다 테스트 케이스 실행 직전에 먼저 실행되어 초기화 작업 등을 진행할 수 있다.

```
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

// (1)
public class BeforeEach1Test {

    @BeforeEach
    public void init() {
        System.out.println("Pre-processing before each test case");
    }

    @DisplayName("@BeforeEach Test1")
    @Test
    public void beforeEachTest() {

    }

    @DisplayName("@BeforeEach Test2")
    @Test
    public void beforeEachTest2() {

    }
}
```

-   `@BeforeAll()`
    -   `@BeforeAll()`은 클레스 레벨에서 테스트 케이스를 한꺼번에 실행 시키면 테스트 케이스가 실행되기 전에 딱 한번만 초기화 작업을 할 수 있도록 해주는 애너테이션이다.

```
public class BeforeAllTest {
    private static Map<String, String> map;

    @BeforeAll
    public static void initAll() {
        map = new HashMap<>();
        map.put("BTC", "Bitcoin");
        map.put("ETH", "Ethereum");
        map.put("ADA", "ADA");
        map.put("POT", "Polkadot");
        map.put("XRP", "Ripple");

        System.out.println("initialize Crypto Currency map");
    }

    @DisplayName("Test case 1")
    @Test
    public void beforeEachTest() {
        assertDoesNotThrow(() -> getCryptoCurrency("XRP"));
    }

    @DisplayName("Test case 2")
    @Test
    public void beforeEachTest2() {
        assertDoesNotThrow(() -> getCryptoCurrency("ADA"));
    }

    private String getCryptoCurrency(String unit) {
        return map.get(unit).toUpperCase();
    }
}
```

#### 테스트 케이스 실행 후, 후처리

JUnit에서는 테스트 케이스 실행이 끝난 시점에 후처리 작업을 할 수 있는 `@AfterEach`, `@AfterAll` 같은 애너테이션도 지원한다.  
  

이 애너테이션은 `@BeforeEach` , `@BeforeAll` 과 동작 방식은 같고, 호출되는 시점만 반대이다.

#### Assumption을 이용한 조건부 테스트

Junit 5에는 Assumption이라는 기능이 추가되었다.  
  

Assumption은 ‘~라고 가정하고’ 라는 표현을 쓸 때의 ‘가정’에 해당한다.  
  

JUnit 5의 Assumption 기능을 사용하면 특정 환경에만 테스트 케이스가 실행 되도록 할 수 있다.

```
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

public class AssumptionTest {
    @DisplayName("Assumption Test")
    @Test
    public void assumptionTest() {
        // (1)
        assumeTrue(System.getProperty("os.name").startsWith("Windows"));
//        assumeTrue(System.getProperty("os.name").startsWith("Linux")); // (2)
        System.out.println("execute?");
        assertTrue(processOnlyWindowsTask());
    }

    private boolean processOnlyWindowsTask() {
        return true;
    }
}
```

(1)에서 `assumeTrue()` 메서드는 파라미터로 입력된 값이 true이면 나머지 아래 로직들을 실행한다.  
  

테스트 케이스를 실행하는 PC의 운영체제(OS)가 윈도우즈(Windows)라면 `assumeTrue()` 메서드의 파라미터 값이 `true`가 될 것이므로 `assumeTrue()` 아래 나머지 로직들이 실행이 될 것이고, 여러분들의 PC 운영체제(OS)가 윈도우즈(Windows)가 아니라면 `assumeTrue()` 아래 나머지 로직들이 실행되지 않을 것이다.  
  

이처럼, `assumeTrue()`는 특정 OS 환경 등의 특정 조건에서 선택적인 테스트가 필요하다면 유용하게 사용할 수 있는 JUnit 5의 API이다.