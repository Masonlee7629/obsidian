태그: #Java 
연결문서: [Java 심화 - 2](Java%20심화%20-%202.md)

# Java 심화(Effective)

## 애너테이션(Annotation)

> 애너테이션이란 소스 코드가 컴파일되거나 실행될 때에 컴파일러 및 다른 프로그램에게 필요한 정보를 전달해주는 문법 요소이다.
> 
> @로 시작하며 클래스, 인터페이스, 필트, 메서드 등에 붙여서 사용할 수 있다.
> 
> 애너테이션은 java.lang.annotation 인터페이스를 상속받기 때문에 다른 클래스나 인터페이스를 상속받을 수 없다.

### 에너테이션의 종류

-   JDK에서 기본적으로 제공하는 애너테이션
    -   표준 애너테이션 : JDK에 내장되어 있는 일반적인 애너테이션
    -   메타 애너테이션 : 다른 애너테이션을 정의할 때 사용하는 애너테이션

### 표준 애너테이션

-   **@Override**
    -   메서드 앞에만 붙일 수 있는 애너테이션
    -   선언한 메서드가 상위 클래스의 메서드를 오버라이딩하거나 추상 메서드를 구현하는 메서드라는 것을 컴파일러에게 알려주는 역할
-   **@Ovreride 사용 방법**
    -   컴파일러가 @Override를 발견하면 상위클래스(또는 인터페이스)에 동일한 이름의 메서드가 존재하는지 검사하고, 만약, 동일한 이름의 메서드를 찾을 수 없다면 컴파일 에러 발생
    -   @Override를 붙이지 않으면 컴파일러는 새로운 메서드를 정의하는 것으로 간주하고 에러를 발생시키지 않으며, 컴파일 에어 없이 코드가 그대로 실행되어 실행 시에 런타임 에러 발생

```
class SuperClass {
    public void example() {
        System.out.println("example() of SuperClass");
    }
}

class SubClass extends SuperClass {

    // 올바른 예시
    @Override
    public void example() {
        System.out.println("example() of SubClass");
    }

    // 잘못된 예시 -> 런타임 에러 발생
    public void exapmle() { // 메서드 이름에 오타가 있습니다. 
        System.out.println("example() of SubClass");
    }
}
```

-   **@Deprecated**
    -   기존에 사용하던 기술이 다른 기술로 대체되어 기존 기술을 적용한 코드를 더 이상 사용하지 않도록 유도하는 경우에 사용
    -   기존의 코드를 다른 코드와의 호환성 문제로 삭제하기 곤란해 남겨두어야만 하지만 더 이상 사용하는 것을 권장하지 않을 때 사용

-   **SuppressWarnings**
    -   컴파일 경고 메시지가 나타나지 않도록 하며 경우에 따라 경고를 발생할 것이 충분히 예상됨에도 묵인해야할 때 사용
    -   애너테이션 뒤에 괄호를 붙이고 그 안에 억제하고자 하는 경고 메시지를 지정 가능
    -   중괄호에 여러 개의 경고 유형을 나열하며 여러 개의 경고를 한번에 묵인 가능

|   애너테이션   |   설명   |
| :-: | :-: |
|   @SuppressWarnings("all")   |   모든 경고를 억제   |
|   @SuppressWarnings("deprecation")   |   Deprecated 메서드를 사용한 경우에 발생하는 경고를 억제   |
|   @SuppressWarnings("fallthrough")   |   switch문에서 break 구문이 없을 때 발생하는 경고를 억제   |
|   @SuppressWarnings("finally")   |   finally와 관련된 경고를 억제   |
|   @SuppressWarnings("null")   |   null과 관련된 경고를 억제   |
|   @SuppressWarnings("unchecked")   |   검증되지 않은 연산자와 관련된 경고를 억제   |
|   @SuppressWarnings("unused")   |   사용하지 않는 코드와 관련된 경고를 억제   |

-   **FunctionalInterface**
    -   함수형 인터페이스를 선언할 때 컴파일러가 함수형 인터페이스의 선언이 바르게 선언되었는지 확인하도록 하며 바르지 않을 경우 에러를 발생

### 메타 애너테이션

> 메타 애너테이션은 애너테이션을 정의하는 데에 사용되는 애너테이션으로 애너테이션의 적용 대상 및 유지 기간을 지정하는 데에 사용된다.

-   **@Target**
    -   애너테이션을 적용할 '대상'을 지정
    -   java.lang.annotation.ElementType 이란 열거형에 정의

|   대상 타입   |   적용범위   |
| :-: | :-: |
|   ANNOTATION\_TYPE   |   애너테이션   |
|   CONSTRUCTOR   |   생성자   |
|   FIELD   |   필드(멤버변수, 열거형 상수)   |
|   LOCAL\_VARIABLE   |   지역변수   |
|   METHOD   |   메서드   |
|   PACKAGE   |   패키지   |
|   PARAMETER   |   매개변수   |
|   TYPE   |   타입(클래스, 인터페이스, 열거형)   |
|   TYPE\_PARAMETER   |   타입 매개변수   |
|   TYPE\_USE   |   타입이 사용되는 모든 대상   |

-   **@Documented**
    -   애너테이션에 대한 정보다 javadoc으로 작성한 문서에 포함되도록 하는 애너테이션 설정
    -   자바에서 제공하는 표준 애너테이션과 메타 애너텡션 중 @Override와 @SuppressWarnungs를 제외하고는 모두 @Documented가 적용되어 있음

-   **@Inherited**
    -   하위 클래스가 애너테이션을 상속받도록 하는 애너테이션

```
@Inherited // @SupAnnotatio이 하위 클래스까지 적용
@interface SupAnnotation{ }

@SupAnnotation
class Super { }

class Sub extends Super{ } // Sub에 애너테이션이 붙은 것으로 인식
```

-   **@Retention**
    -   특정 애너테이션의 지속 시간을 결정하는 데에 사용
    -   애너테이션과 관련된 유지 정책(retention policy)
        -   SOURCE : 소스 파일에 존재, 클래스 파일에는 존재하지 않음
            -   CLASS : 클래스 파일에 존재, 실행 시에 사용 불가, 기본값
            -   RUNTIME : 클래스 파일에 존재, 실행 시에 사용 가능

-   **@Repeatable**
    -   애너테이션을 여러 번 붙일 수 있도록 하용한다는 의미를 가짐
    -   같은 이름의 애너테이션들을 하나로 묶어주는 애너테이션을 별도로 작성해야 함

```
// Work 애너테이션을 여러 번 반복해서 쓸 수 있게 함
@Repeatable(Works.class)
@interface Work{  
    String value();  
}

@Work("코드 업데이트")  
@Work("메서드 오버라이딩")  
class Main{  
    ... 생략 ...
}
```

```
// 여러개의 Work애너테이션을 담을 컨테이너 애너테이션 Works
@interface Works {
    Work[] value(); 
}

@Repeatable(Works.class) // 컨테이너 애너테이션 지정 
@interface Work {
    String value();
}
```

### 사용자 정의 애너테이션

> 사용자가 직접 애너테이션을 정의해서 사용하는 것을 의미한다.

```
// 인터페이스 앞에 @기호만 붙이면 애너테이션을 정의할 수 있음
@interface 애너테이션명 { 
    타입 요소명(); // 애너테이션 요소를 선언
}
```
