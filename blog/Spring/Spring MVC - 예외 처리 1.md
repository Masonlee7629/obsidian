태그: #Spring
연결문서: [Spring MVC - 예외 처리 2](Spring%20MVC%20-%20예외%20처리%202.md)

# Spring MVC에서의 예외 처리

## @ExceptionHandler를 이용한 예외 처리

Spring에서의 예외는 애플리케이션에 문제가 발생할 경우, 이 문제를 알려서 처리하는 것 뿐만 아니라 유효성 검증에 실패했을 때와 같이 이 실패를 하나의 예외로 간주하여 이 예외를 던져서(throw) 예외 처리를 유도한다.

### @ExceptionHandler를 이용한 Controller 레벨에서의 예외 처리

#### Controller 클래스에 @ExceptionHandler 적용

```
@RestController
@RequestMapping("/v1/members")
public class Controller {
  // 유효성 검증 과정에 실패할 떄의 예외 처리

  @ExceptionHandler
  public ResponseEntity handleException(MethodArgumentNotValidException e) {
      final List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();

      return new ResponseEntity><>(fieldErrors, HttpStatus.BAD_REQUEST);
  }
}
```

#### 유효하지 않은 이메일 주소를 포함한 POST 요청 전송에 대한 응답 메시지

```
// 현재는 Response Body 전체 정보를 전달
[
    {
        "codes": [
            "Email.memberPostDto.email",
            "Email.email",
            "Email.java.lang.String",
            "Email"
        ],
        "arguments": [
            {
                "codes": [
                    "memberPostDto.email",
                    "email"
                ],
                "arguments": null,
                "defaultMessage": "email",
                "code": "email"
            },
            [],
            {
                "arguments": null,
                "defaultMessage": ".*",
                "codes": [
                    ".*"
                ]
            }
        ],
        "defaultMessage": "올바른 형식의 이메일 주소여야 합니다",
        "objectName": "memberPostDto",
        "field": "email",
        "rejectedValue": "hgd@",
        "bindingFailure": false,
        "code": "Email"
    }
]
```

요청 전송 시 Request Body의 JSON 프로퍼티 중 문제가 된 프로퍼티와 에러 메시지만 전달하는 것이 가능하다.

-   **에러 정보를 기반으로 한 Error Response 클래스를 만들어 필요한 정보만 담은 후 클라이언트에 전달**

#### ErrorResponse 클래스 적용

한 개 이상의 유효성 검증에 실패한 필드의 에러 정보를 담기 위해 List 객체를 이용한다.  
  

하나의 필드 에러 정보는 FieldError 라는 별도의 static class를 ErrorResponse 클래스의 멤버 클래스로 정의한다.

```
@Getter
@AllArgsConstructor
public class ErrorResponse {
  private List<FieldError> fieldErrors;

  @Getter
  @AllArgsConstructor
  public static class FieldError {
    private String field;
    private Object rejectedValue;
    private String reason;
  }
}
```

#### ErrorResponse를 사용하도록 Controller의 handleException() 메서드 수정

```
public class Controller {
  @ExceptionHandler
  public ResponseEntity handleException(MethodArgumentNotValidException e) {
    final List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();

    List<ErrorResponse.FieldError> errors =
            fieldErrors.stream()
                    .map(error -> new ErrorResponse.FieldError(
                      error.getField(),
                      error.getRejectedValue(),
                      error.getDefaultMessage()))
                    .collect(Collectors.toList());

    return new ResponseEntity<>(new ErrorResponse(errors), HttpStatus.BAD_REQUEST);
  }
}
```

### @ExceptionHandler의 단점

-   각각의 Controller 클래스에서 `@ExceptionHandler` 애너테이션을 사용하셔 Request Body에 대한 유효성 검증 실패에 대한 에러 처리를 해야하므로 각 Controller 클래스마다 코드 중복이 발생
-   Controller에서 처리해야하는 예외가 유효성 검증 실패에 대한 예외만 있는 것이 아니므로 Controller 클래스 내에서 `@ExceptionHandler`를 추가한 에러 처리 핸들러 메서드가 증가함