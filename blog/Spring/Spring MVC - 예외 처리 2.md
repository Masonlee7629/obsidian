태그: #Spring
연결문서: [Spring MVC - 예외 처리 3](Spring%20MVC%20-%20예외%20처리%203.md)

# Spring MVC에서의 예외 처리

## @RestControllerAdvice를 이용한 예외처리

### @RestControllerAdvice를 사용한 예외 처리 공통화

`@RestControllerAdvice` 애너테이션을 추가한 클래스를 이용하면 **예외 처리를 공통화할 수 있다.**

-   특정 클래스에 `@RestControllerAdvice` 애너테이션을 추가하면 여러개의 Controller 클래스에서 `@ExceptionHandler`, `@InitBinder` 또는 `@ModelAttribute` 가 추가된 메서드를 공유해서 사용할 수 있다.

#### @RestControllerAdvice 적용 순서

1.  기존의 Controller 클래스에서 사용한 `@ExceptionHandler`가 추가된 메서드는 특정 클래스에서 공통으로 처리되므로 모두 제거한다.
2.  ExceptionAdvice 클래스 정의
    -   Controller 클래스에서 발생하는 예외들을 공통으로 처리할 클래스 정의
3.  Exception 핸들러 메서드 구현
4.  ErrorResponse 수정
5.  Exception 핸들러 메서드 수정

#### @RestControllerAdvice 적용 방식

1.  ExectionAdvice 클래스 정의

```
@RestControllerAdvice
public class GlobalExceptionAdvice {

}
```

2.  Exception 핸들러 메서드 구현  
    handleConstraintViolationException() 메서드는 ErrorResponse 클래스가 ConstraintViolationException 에 대한 Error Response를 생성할 수 없기에 ErrorResponse 클래스의 추가 수정 필요하다.

```
@RestControllerAdvice
public class GlobalExceptionAdvice {
  @ExceptionHandler
  public ResponseEntity handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
    final List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();

    List<ErrorResponse.FieltError> errors = 
          fieldErrors.stream()
                  .map(error -> new ErrorResponse.FieldError(
                    error.getField(),
                    error.getRejectedValue(),
                    error.getDefaultMessage()))
                  .collect(Collectors.toList());

    return new ResponseEntity<>(new ErrorResponse(errors), HttpStatus.BAD_REQUEST);
  }

  @ExceptionHandler
  public ResponseEntity handleConstraintViolationException(ConstraintViolationException e) {
    return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
  }
}
```

3.  ErrorResponse 수정

(1) DTO 멤버 변수 필드의 유효성 검증 실패로 발생한 에러 정보를 담는 멤버 변수

(2) URI 변수 값의 유효성 검증에 실패로 발생한 에러 정보를 담는 멤버 변수

(3) ErrorResponse 클래스의 생성자

-   private 접근 제한자를 지정
    -   new ErrorResponse 방식으로 객체 생성 불가능
    -   (4), (5)와 같이 of() 메서드로 ErrorResponse 의 객체 생성 가능
        -   ErrorResponse 의 객체를 생성함과 동시에 역할을 명확히 함

(4) MethodArgumentNtoValidException 에 대한 ErrorResponse 객체를 생성

-   MethodArgumentNotValidException 에서 에러 정보를 얻기 위해 필요한 것은 BindingResult
-   of() 메서드를 호출하는 쪽에서 BindingResult 객체를 파라미터로 전달
    -   BindingResult 객체로 에러 정보를 추출, 가공하는 것은 FieldError 클래스에게 위임

(5) ConstraintViolationException 에 대한 ErrorResponse 객체를 생성

-   ConstraintViolationException 에서 에러 정보를 얻기 위해 필요한 것은 Set<ConstraintViolation<?>> 객체
-   of() 메서드를 호출하는 쪽에서 Set<ConstraintViolation<?>> 객체를 파라미터로 전달
    -   Set<ConstraintViolation<?>> 객체로 에러 정보를 추출, 가공하는 것은 ConstraintViolationError 클래스에게 위임

(6) 필드(DTO 클래스의 멤버 변수)의 유효성 검증에서 발생하는 에러 정보를 생성

(7)URI 변수 값에 대한 에러 정보를 생성

```
@Getter
public class ErrorResponse {
  private List<FieldError> fieldErrors;   // (1)
  private List<ConstraintViolationError> violationErrors;   // (2)

  // (3)
  private ErrorResponse(final List<FieldError> fieldErrors,
                        final List<ConstraintViolationError> violationErrors) {
    this.fieldErrors = fieldErros;
    this.violationErrors = violationErrors;
  }

  // (4) BindinResult에 대한 ErrorResponse 객체 생성
  public static ErrorResponse of(BindingResult bindingResult) {
    return new ErrorResponse(FieldError.of(bindingResult), null);
  }

  // (5) Set<ConstraintViolation<?>> 객체에 대한 ErrorResponse 객체 생성
  public static ErrorResponse of(Set<ConstraintViolation<?>> violation) {
    return new ErrorResponse(null, ConstraintViolationError.of(violation));
  }

  // (6) Field Error 가공
  @Getter
  public static class FieldError {
    private String field;
    private Object rejectedValue;
    private String reason;

    private FieldError(String field, Object rejectedValue, String reason) {
      this.field = field;
      this.rejectedValue = rejectedValue;
      this.reason = reason;
    }

    private static List<FieldError> of(BindingResult bindingResult) {
      final List<org.springframework.validation.FieldError> fieldErrors = 
              bindingResult.getFieldErrors();

      return fieldErrors.stream()
          .map(error -> new FieldError(
            error.getField(),
            error.getRejectedValue() == ? "" : error.getRejectedValue().toString(),
            error.getDefaultMessage()))
          .collect(Collectors.toList());
    }
  }

  // (7) ConstraintViolation Error 가공
  @Getter
  public static class ConstraintViolationError {
    private String propertyPath;
    private Object rejectedValue;
    private String reason;

    private ConstraintViolationError(String propertyPath,
                                     Object rejectedValue,
                                     String reason) {
      this.propertyPath = propertyPath;
      this.rejectedValue = rejectedValue;
      this.reason = reason;
    }

    public static List<ConstraintViolationError> of(
            Set<ConstraintViolation<?>> constraintViolations) {
      return constraintViolations.stream()
              .map(constraintViolation -> new ConstraintViolationError(
                constraintViolation.getPropertyPath().toString(),
                constraintViolation.getInvalidValue().toString(),
                constraintViolation.getMessage()))
              .collect(Collectors.toList());
    }
  }
}
```

#### Exception 핸들러 메서드 수정

```
@RestControllerAdvice
public class GlobalExceptionAdvice {
  @ExceptionHandler
  @ResponseStTUS(HttpStatus.BAD_REQUEST)
  public ErrorResponse handleMethodArgumentNotValidException(
    MethodArgumentNotValidException e) {
     final ErrorResponse response = ErrorResponse.of(e.getBindingResult());

     return response;
  }

  @ExceptionHandler
  @ResponseStatus(HttpStatus.BAD_REQEUST)
  public ErrorResponse handleConstraintViolationException(
    ConstraintViolationException e) {
      final ErrorResponse response = ErrorResponse.of(e.getConstraintViolations());

      return response;
  }
}
```

### @RestControllerAdvice vs @ControllerAdvice

**`@RestControllerAdvice` = `@ControllerAdvice` + `@ResponseBody`**  
  

Spring MVC 4.3 버전 이후부터 @RestControllerAdvice 애너테이션을 지원한다.  
  

`@RestControllerAdvice` 애너테이션은 `@ControllerAdvice` 과 `@ResponseBody`의 기능을 포함하고 있다.  
  

따라서 **JSON 형식의 데이터를 Response Body로 전송하기 위해서는 ResponseEntity로 데이터를 래핑할 필요가 없다.**