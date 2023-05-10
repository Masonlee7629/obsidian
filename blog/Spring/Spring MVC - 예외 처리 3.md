태그: #Spring
연결문서: [Spring MVC - Spring Data JPA 1](Spring%20MVC%20-%20Spring%20Data%20JPA%201.md)

# 비즈니스 로직에 대한 예외 처리

## 비즈니스적인 예외 던지기(throw) 및 예외 처리

### 체크 예외와 언체크 예외

애플리케이션에서 발생하는 예외(Exception)는 크게 체크 예외(Check Exception)와 언체크 예외(Unchecked Exception)로 구분할 수 있다.

-   **체크 예외**
    -   예외를 잡아서(catch) 체크한 후에 해당 예외를 복구 또는 회피 등의 어떤 구체적인 처리를 해야 하는 예외
    -   Ex) ClassNotFoundException
-   **언체크 예외**
    -   예외를 잡아서(catch) 해당 예외에 대한 어떤 처리를 할 필요가 없는 예외
    -   Ex) NullPointerException, ArrayIndexOutOfBoundsException

### 개발자가 의도적으로 예외를 던질 수 있는 상황

-   **백엔드 서버와 외부 시스템과의 연동에서 발생하는 에러 처리**
-   **시스템 내부에서 조회하려는 리소스가 없는 경우**

### 의도적인 예외 던지기/받기(throw/catch)

서비스 계층의 메서드는 API 계층인 Controller의 핸들러 메서드가 이용하므로 서비스 계층에서 던져진 예외는 Controller의 핸들러 메서드 쪽에서 잡아서 처리할 수 있다.  
  

그러나 이미 Controller에서 발생하는 예외를 Exception Advice에서 처리하도록 공통화 해두었다면 서비스 계층에서 던진 예외 역시 Exception Advice에서 처리하면 된다.

#### 서비스 계층에서 예외 던지기

throw 키워드를 사용하여 RuntimeException 객체에 적절한 예외 메시지를 포함한 후에 메서드 밖으로 던진다.

```
@Service
public class MemberService {
    ...
        ...

    public Member findMember(long memberId) {

                // (1)
        throw new RuntimeException("Not found member");
    }

        ...
        ...
}
```

#### ExceptionAdvice 예외 잡기

Controller의 핸들러 메서드에 요청을 보내면 Service에서 RuntimeException을 던지고, ExceptionAdvice의 handleResourceNotFoundException() 메서드가 이 RuntimeException을 잡아서 예외 메시지를 출력할 수 있다.

```
@RestControllerAdvice
public class GlobalExceptionAdvice {
    ...
        ...

        // (1)
    @ExceptionHandler
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFoundException(RuntimeException e) {
        System.out.println(e.getMessage());


        return null;
    }
}
```

### 사용자 정의 예외(Custom Exception)

예외 발생 시 RuntimeException과 같은 추상적인 예외가 아닌 InsufficentBalanceException 같은 해당 예외를 조금 더 구체적으로 표현할 수 있는 Custom Exception을 만들어서 예외 던지기가 가능하다.

-   서비스 계층에서 RuntimeException을 던지고 Exception Advice에서 그대로 잡는 것은 예외의 의도가 명확하지 않고 구체적으로 어떤 예외가 발생했는지에 대한 예외 정보를 얻기 어려움

#### 사용자 정의 예외 순서

1.  서비스 계층에서 던질 Custom Exception에 사용할 ExceptionCode를 enum으로 정의
    -   ExceptionCode를 enum으로 정의하면 비즈니스 로직에서 발생하는 다양한 유형의 예외를 enum에 추가하여 사용 가능

```
import lombok.Getter;

public enum ExceptionCode {
    MEMBER_NOT_FOUND(404, "Member Not Found");

    @Getter
    private int status;

    @Getter
    private String message;

    ExceptionCode(int status, String message) {
        this.status = status;
        this.message = message;
    }
}
```

2.  서비스 계층에서 사용할 Custom Exception 정의
    -   추상적인 RuntimeException 예외를 구체화할 경우 RuntimeException을 상속하며 ExceptionCode를 멤버 변수로 지정하여 생성자를 통해 구체적인 예외 정보를 제공
    -   예외 메시지는 상위 클래스인 RuntimeException의 생성자(super)로 전달
    -   CustomException은 서비스 계층에서 개발자가 의도적으로 예외를 던져야 하는 여러 상황에서 ExceptionCode 정보만 바꿔가며 던질 수 있음

```
import lombok.Getter;

public class BusinessLogicException extends RuntimeException {
    @Getter
    private ExceptionCode exceptionCode;

    public BusinessLogicException(ExceptionCode exceptionCode) {
        super(exceptionCode.getMessage());
        this.exceptionCode = exceptionCode;
    }
}
```

3.  서비스 계층에 Custom Exception 적용

```
@Service
public class MemberService {
    ...
        ...

    public Member findMember(long memberId) {

                // (1)
        throw new BusinessLogicException(ExceptionCode.MEMBER_NOT_FOUND);
    }

    ...
        ...
}
```

4.  Exception Advice에서 Custom Exception 처리

```
@RestControllerAdvice
public class GlobalExceptionAdvice {
    ...
        ...

    @ExceptionHandler
    public ResponseEntity handleBusinessLogicException(BusinessLogicException e) {
        System.out.println(e.getExceptionCode().getStatus());
        System.out.println(e.getMessage());

        return new ResponseEntity<>(HttpStatus.valueOf(e.getExceptionCode().getStatus()));
    }
}
```
