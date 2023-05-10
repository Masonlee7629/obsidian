태그: #Spring
연결문서: [Spring MVC - 슬라이스 테스트 1](Spring%20MVC%20-%20슬라이스%20테스트%201.md)

# 테스팅(Testing)

## Hamcrest를 사용한 Assertion

### Hamcrest란?

Hamcrest는 JUnit 기반의 단위 테스트에서 사용할 수 있는 Assertion Framework이다.  
  

JUnit에서도 Assertion 을 위한 다양한 메서드를 지원하지만 Hamcrest는 다음과 같은 이유로 JUnit에서 지원하는 Assertion 메서드보다 더 많이 사용된다.

-   Assertion을 위한 **매쳐(Matcher)**가 자연스러운 문장으로 이어지므로 **가독성이 향상** 된다.
-   **테스트 실패 메시지를 이해하기 쉽다**
-   **다양한 Matcher를 제공한다.**

### 단위 테스트에 Hamcrest Assertion 적용 예시

```
// JUnit에서의 Assertion
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class HelloJunitTest {
    @DisplayName("Hello Junit Test")
    @Test
    public void assertionTest1() {
        String actual = "Hello, JUnit";
        String expected = "Hello, JUnit";

        assertEquals(expected, actual); // (1)
    }
}
```

```
// Hamcrest에서의 Assertion
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.equalTo;
import static org.hamcrest.Matchers.is;

public class HelloHamcrestTest {

    @DisplayName("Hello Junit Test using hamcrest")
    @Test
    public void assertionTest1() {
        String expected = "Hello, JUnit";
        String actual = "Hello, JUnit";

        assertThat(actual, is(equalTo(expected)));  // (1)
    }
}
```

위 코드는 Unit의 Assertion 메서드를 사용한 HelloJunitTest 클래스와 Hamcrest의 매쳐(Matcher)를 사용한 HelloHamcrestTest 클래스이다.

#### JUnit Assertion vs Hamcrest의 매쳐(Matcher)

-   **JUnit Assertion 기능 이용**
    -   `assertEquals(expected, actual);`
        -   파라미터로 입력된 값의 변수 이름을 통해 대략적으로 어떤 검증을 하려는지 알 수 있으나 구체적인 의미는 유추하는 과정이 필요하다.
-   **Hamcrest의 매쳐(Matcher) 이용**
    -   `assertThat(actual, is(equalTo(expected)));`
        -   (1)의 Assertion 코드 한 줄은 `assert that actual is equal to expected` 라는 하나의 영어 문장으로 자연스럽게 읽혀진다.
            -   한글로 번역하자면, ‘`결과 값(actual)이 기대 값(expected)과 같다는 것을 검증(Assertion)한다.`’ 정도로 해석할 수 있다.
        -   `assertThat()` 메서드의 파라미터
            -   첫 번째 파라미터는 테스트 대상의 실제 결과 값이다.
            -   두 번째 파라미터는 기대하는 값이다.