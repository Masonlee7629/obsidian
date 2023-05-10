태그: #Java 
연결문서: [Java 컬렉션 - 2](Java%20컬렉션%20-%202.md)

# Java 컬렉션

## Java 열거형

### 열거형

> 여러 상수들을 보다 편리하게 선언할 수 있도록 만들어진 자바의 문법 요소이다.
> 
> 상수명의 중복을 피하고 타입에 대한 안정성을 보장한다.
> 
> 간결하고 가독성이 좋은 코드를 작성할 수 있으며 switch문에서 작동이 가능하다.

-   **열거형의 사용 방법**
    -   대소문자 모두 사용 가능하지만 관례적으로 대문자를 사용
    -   값을 따로 지정하지 않아도 자동적으로 0부터 시작하는 정수값이 할당

```
enum 열거형이름 {상수명1, 상수명2, 상수명3, ...}

// 열거형 사용 예시
enum Numbering {
  FIRST,
  SECOND,
  THIRD
}

public class Main {
  public static void main(String[] args) {
    Numbering num = Numbering.THIRD;

    switch(num) {
      case FIRST:
        System.out.println("First Number");
        break;
      case SECOND:
        System.out.println("Second Number");
        break;
      case THIRD:
        System.out.println("Third Number");
        break;
    }
  }
}

// 출력 결과
// Third Number
```

-   **열거형에서 사용 가능한 메서드**
    -   해당 메서드는 모두 열거형의 모상인 java.lang.Enum에 정의

|   리턴타입   |   메서드(매개변수)   |   설명   |
| :-: | :-: | :-: |
|   String   |   name()   |   열거 객체가 가진 문자열을 리턴   리턴되는 문자열은 열거 타입을 정의할 때 사용한 상수명과 동일   |
|   int   |   ordinal()   |   열거 객체의 순번(0부터 시작)을 리턴   |
|   int   |   compareTo(비교값)   |   주어진 매개값과 비교하여 순번 차이를 리턴   |
|   열거타입   |   valueOf(String name)   |   주어진 문자열의 열거 객체를 리턴   |
|   열거배열   |   values()   |   모든 열거 객체들을 배열로 리턴   |

---

## Java의 제네릭

> 타입을 추후에 지정할 수 있도록 일반화해두는 것이다.
> 
> 작성한 클래스 또는 메서드의 코드가 특정 데이터 타입에 얽매이지 않게 해둔 것이다.

-   **제네릭 클래스**
    -   제네릭이 사용된 클래스
    -   클래스 변수(static)에는 타입 매개변수의 사용이 불가능

```
class 클래스명<타입 매개변수> {}

// 타입 매개변수는 임의의 문자로 지정 가능
// T = Type, K = Key, V = Value, E = Element, N = Number, R = Result 를 주로 사용
```

-   제네릭 클래스의 사용 방법
    -   제네릭 클래스를 인스턴스화할 때는 의도하는 타입을 지정해야 함
    -   타입 매개변수에 치환될 타입으로 기본 타입의 지정이 불가능하며 Integer, Double과 같은 래퍼 클래스를 활용해야 함
    -   제네릭 클래스 사용 시 다형성 적용 가능

```
Box<String> box1 = new Box<String>("Java");
Box<Integer> box2 = new Box<Integer>(5);
Box<Double> box3 = new Box<Double>(8.15);

// new Box<...> 는 구체적인 타입의 생략이 가능, 찹조변수의 타입으로 유추
```

-   **제한된 제네릭 클래스**
    -   제네릭 클래스를 인스턴스화할 때 타입으로 특정 하위 클래스 또는 인터페이스를 구현한 클래스만 지정하도록 제한하는 것
    -   extends 키워드를 사용
    -   상속과 인터페이스를 동시에 지정하려면 &를 사용하며 클래스를 인터페이스보다 앞에 위치시켜야 함

```
class Toy {...}
class Doll extends Toy {...}
class Snack {...}

// 제네릭 클래스 정의
class Box<T extends Toy> {
  private T item;

  public T getItem() {
    return item;
  }

  public T setItem(T item) {
    this.item = item;
  }
}

public static void main(String[] args) {
  // 인스턴스화
  Box<Toy> toyBox = new Box<>();
  Box<Snack> snackBox = new Box<>(); // 에러
}
```

-   **제네릭 메서드**
    -   클래스 내부의 특정 메서드만 제네릭으로 선언한 것
    -   제네릭 메서드의 타입 매개변수는 제네릭 클래스의 타입 매개변수와 별개로 다른 타입 매개변수로 간주함
    -   제네릭 클래스는 클래스가 인스턴스화될 때이며 제네릭 메서드는 메서드가 호출될 때 타입 지정
    -   static 메서드에도 선언하여 사용 가능
    -   제네릭 메서드를 정의하는 시점에 실제 타입을 알 수 없으므로 length()와 같은 String 클래스의 메서드는 사용 불가
    -   Object 클래스의 메서드는 사용 가능

```
class Box<T> {  //제네릭 클래스의 타입 매개변수
  ...
  public <T> void sum(T item) {  //  제네릭 매서드의 타입 매개변수
    System.out.println(item.length());    // 사용 불가
    System.out.println(item.equals("Mason Lee"));    // 사용 가능
  }
}
```

-   **와일드카드**
    -   제네릭에서 어떠한 타입으로든 대체될 수 있는 타입 파라미터를 의미하며 '?' 기호로 사용
    -   일반적으로 extends와 super 키워드를 조합하여 사용

```
<? extends T>     // 와일드카드에 상한 제한을 두는 것
                  // T와 T를 상속받는 하위 클래스 타입만 타입 파라미터로 받을 수 있도록 지정

<? super T>       // 와일드카드에 하한 제한을 두는 것
                  // T와 T의 상위 클래스만 타입 파라미터로 받을 수 있도록 지정

<?>               // <? extends Object>와 동일하며 모든 클래스 타입을 타입 파라미터로 받을 수 있음
```

---

## Java의 예외 처리

> 프로그램의 비정상적인 종료를 방지하고 정상적인 실행 상태를 유지하기 위한 것이다.
> 
> 자바에서는 발생 시점에 따라 에러를 컴파일 에러와 런타임 에러로 구분한다.

### 컴파일 에러(Compile Time Error)

-   컴파일할 때 발생하는 에러
-   주로 세미콜론 생략, 오탈자 등 문법적 문제인 신택스(syntax) 오류로부터 발생하므로 신택스 에러(Syntax Error)라고 부르기도 함
-   자바 컴파일러가 오류를 감지하여 사용자에게 알려주어 상대적으로 쉽게 발견하고 수정 가능

### 런타임 에러(Runtime Error)

-   런타임 시에 발생하는 에러
-   개발자가 컴퓨터가 수행할 수 없는 작업을 요청할 때 주로 발생
-   자바 가상 머신(JVM)에 의해 감지

### 에러와 예외

-   자바에서 코드 실행(run-time) 시 잠재적으로 발생할 수 있는 오류를 에러와 예외로 구분
-   에러는 복구하기 어려운 수준의 심각한 오류를 의미하며 메모리 부족, 스택오버플로우 등이 있음
-   예외는 잘못된 사용 또는 코딩으로 인한 상대적으로 미약한 수준의 오류로 코드 수정 등을 통해 수습이 가능한 오류
-   예외의 최고 상위 클래스는 Exception 클래스이며 일반 예외 클래스와 실행 예외 클래스로 구분

-   **일반 예외 클래스(Exception)**
    -   런타임 시 발생하는 RuntimeException 클래스와 그 하위 클래스를 제외한 모든 Exception 클래스와 그 하위 클래스를 지칭
    -   컴파일러가 코드 실행 전에 예외 처리 코드 여부를 검사하여 checked 예외라고도 부름
    -   주로 잘못된 클래스명이나 데이터 형식 등 사용자 편의 실수로 발생

-   **실행 예외 클래스(Runtime Exception)**
    -   RuntimeException 클래스와 그 하위 클래스를 지칭
    -   컬파일러가 예외 처리 코드 여부를 검사하지 않아 unchecked 예외라고 부름
    -   개발자의 실수에 의해 발생하는 경우가 많으며 자바 문법 요소와 관련

### try-catch문

-   자바에서 예외 처리는 try-catch문을 통해 구현 가능
-   '예외 발생 - 첫 번째 catch문 - 두 번째 catch문 - finally문'의 과정으로 진행

```
try {
  // 예외가 발생할 가능성이 있는 코드 삽입
}
catch (ExceptionType1 except1) {
  // ExceptionType1 유옇의 예외 발생 시 실행할 코드
}
catch (ExceptionType2 except2) {
  // ExceptionType2 유옇의 예외 발생 시 실행할 코드
}
finally {
  // 예외 발생 여부와 상관없이 항상 실행
  // finally 블럭은 옵셔널
}
```

### 예외 전가

-   예외를 호출한 곳으로 다시 예외를 떠넘기는 방법
-   메서드의 선언부 끝에 throws 키워드와 발생할 수 있는 예외들을 쉼표로 구분하여 나열
-   throw 키워드를 사용하여 의도적으로 예외를 발생시킬 수 있음

-   **예외 전가 사용 방법**

```
반환타입 메서드명(매개변수, ...) throws 예외클래스 1, 예외클래스2... {}

// 모든 종류의 예외가 발생할 가능성이 있는 경우
void ExamMethod() throws Exception {
}
```

-   **예외를 의도적으로 발생시키는 경우**

```
public class ThrowExam {
  public static void main(String[] args) {
    try {
      Exception intendedException = new Exception("의도적인 예외");
      throw intendedExcepton;
    } catch (Exception e) {
      System.out.println("의도적인 예외 발생 성공");
    }
  }
}

// 출력 결과
// 의도적인 예외 발생 성공
```
