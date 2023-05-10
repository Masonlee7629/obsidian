태그: #Java 
연결문서: [Java 심화 - 3](Java%20심화%20-%203.md)

# Java 심화

## 람다(Lambda)

> 함수형 프로그래밍 기법을 지원하는 자바의 문법 요소이다.
> 
> 메서드를 하나의 '식(expression\_'으로 표현한 것으로 코드를 매우 간결하면서 명확하게 표현할 수 있다는 큰 장점이 있다.
> 
> 자바는 JDK 1.8 이후 람다식 등 함수형 프로그래밍 문법 요소를 도입하며 기존의 객체지향 프로그래밍과 함수형 프로그래밍을 혼합하는 방식으로 보다 효율적인 프로그래밍을 할 수 있게 되었다.

### 람다식의 기본 문법

-   기본적으로 반환타입과 이름을 생략 가능
-   익명 함수(annonymous function)라 부르기도 함
-   메서드 바디에 문장이 실행문이 하나만 존재할 때 중괄호와 return문을 생략 가능하며 이 경우 세미콜론까지 생략해야 함
-   매개변수 타입을 함수형 인터페이스를 통해 유추할 수 있는 경우 매개변수의 타입 생략 가능

```
// 기존 방식
int sum(int num1, int num2) {
    return num1 + num2;
}

// 람다식
(int num1, int num2) -> {
    return num1 + num2;
}
```

```
// 기존 방식
void example1() {
    System.out.println(3);
}

// 람다식
() -> {System.out.println(3);}
```

```
// 기존 방식
int example2() {
    return 5;
}

// 람다식
() -> {return 5;}
```

```
/ 기존 방식
void example3(String str1) {
    System.out.println(str1);
}

// 람다식
(String str1) -> {    System.out.println(str1);}
```

### 함수형 인터페이스

-   기존의 인터페이스 문법을 활용하여 람다식을 다루는 것
-   람다식과 인터페이스의 메서드가 1:!로 매칭되어야 하기 떄문에 함수형 인터페이스에는 단 하나의 추상 메서드만 선언될 수 있음

```
public class LamdaExample1 {
    public static void main(String[] args) {
        ExampleFunction exampleFunction = (num1, num2) -> num1 + num2;
        System.out.println(exampleFunction.sum(5,10));
}

@FunctionalInterface // 컴파일러가 인터페이스가 바르게 정의되었는지 확인
interface ExampleFunction {
        int sum(int num1, int num2);
}

// 출력 결과
15
```

#### 매개변수와 리턴값이 없는 람다식

-   예시 1

```
@FunctionalInterface
public interface MyFunctionalInterface {
    void accept();
}

MyFunctionalInterface example = () -> { ... };

// example.accept();
```

-   예시 2

```
@FunctionalInterface
interface MyFunctionalInterface {
    void accept();
}

public class MyFunctionalInterfaceExample {

        public static void main(String[] args) throws Exception {

                MyFunctionalInterface example = () -> System.out.println("accept()");
                example.accept();

        }
}

// 출력값
accept()
```

#### 매개변수가 있는 람다식

-   예시 1

```
@FunctionalInterface
public interface MyFunctionalInterface {
    void accept(int x);
}

public class MyFunctionalInterfaceExample {

    public static void main(String[] args) throws Exception {

        MyFunctionalInterface example;
        example = (x) -> {
            int result = x * 2;
            System.out.println(result);
        };
        example.accept(2);

        example = (x) -> System.out.println(x * 2);
        example.accept(2);
    }
}

// 출력값
4
4
```

#### 리턴값이 있는 람다식

-   예시 1

```
@FunctionalInterface
public interface MyFunctionalInterface {
    int accept(int x, int y);
}

public class MyFunctionalInterfaceExample {

    public static void main(String[] args) throws Exception {

        MyFunctionalInterface example;

        example = (x, y) -> {
            int result = x + y;
            return result;
        };
        int result1 = example.accept(2, 3);
        System.out.println(result1);


        example = (x, y) -> { return x + y; };
        int result2 = example.accept(2, 3);
        System.out.println(result2);


          example = (x, y) ->  x + y;
                //return문만 있을 경우, 중괄호 {}와 return문 생략가능
        int result3 = example.accept(2, 3);
        System.out.println(result3);


        example = (x, y) -> sum(x, y);
                //return문만 있을 경우, 중괄호 {}와 return문 생략가능
        int result4 = example.accept(2, 3);
        System.out.println(result4);

    }

    public static int sum(int x, int y){
        return x + y;
    }
}

//출력값
5
5
5
5
```

### 메서드 레퍼런스(메서드 참조)

> 람다식에서 불필요한 매개변수를 제거할 때 주로 사용한다.

```
// 클래스이름::메서드이름

Math :: max

// 클래스 :: 메서드
// 참조변수 :: 메서드
```

-   예시 1

```
public class Calculator {
  public static int staticMethod(int x, int y) {
                        return x + y;
  }

  public int instanceMethod(int x, int y) {
   return x * y;
  }
}
```

```
import java.util.function.IntBinaryOperator;

public class MethodReferences {
  public static void main(String[] args) throws Exception {
    IntBinaryOperator operator;

    /*정적 메서드
        클래스이름::메서드이름
    */
    operator = Calculator::staticMethod;
    System.out.println("정적 메서드 결과 : " + operator.applyAsInt(5, 5));

    /*인스턴스 메서드
        인스턴스명::메서드명
    */
    Calculator calculator = new Calculator();
    operator = calculator::instanceMethod;
    System.out.println("인스턴스 메서드 결과 : "+ operator.applyAsInt(5, 5));
  }
}

// 출력 결과
정적메서드 결과 : 10
인스턴스 메서드 결과 : 25
```

#### 생성자 참조

-   예시 1

```
public class Member {
  private String name;
  private String id;

  public Member() {
    System.out.println("Member() 실행");
  }

  public Member(String id) {
    System.out.println("Member(String id) 실행");
    this.id = id;
  }

  public Member(String name, String id) {
    System.out.println("Member(String name, String id) 실행");
    this.id = id;
    this.name = name;
  }

  public String getName() {
    return name;
  }

public String getId() {
    return id;
  }
```

```
import java.util.function.BiFunction;
import java.util.function.Function;

public class ConstructorRef {
  public static void main(String[] args) throws Exception {
    Function<String, Member> function1 = Member::new;
    Member member1 = function1.apply("Mason");

    BiFunction<String, String, Member> function2 = Member::new;
    Member member2 = function2.apply("Mason", "메이슨");
  }
}

// 출력 결과
Member(String id) 실행
Member(String name, String id) 실행
```