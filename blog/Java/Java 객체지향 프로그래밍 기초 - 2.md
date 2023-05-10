태그: #Java 
연결문서: [Java 객체지향 프로그래밍 심화 - 1](Java%20객체지향%20프로그래밍%20심화%20-%201.md)

## Java의 생성자

### 생성자(Constructor)

> 인스턴스 변수를 초기화하는 데 사용되는 메서드이다.
> 
> > -   생성자와 메서드의 차이점
> >     -   생성자의 이름은 반드시 클래스의 이름과 같아야 함
> >     -   생성자는 리턴 타입이 없음

-   기본 생성자(Default Constructor)
    -   모든 클래스에는 반드시 하나 이상의 생성자가 존재
    -   생성자가 클래스 안에 없을 경우 자바 컴파일러는 기본 생성자를 자동으로 추가
    -   생성자가 있을 경우 기본 생성자가 아닌 추가한 생성자를 기본으로 사용

```
클래스명(){}      // 기본 생성자

DefConst(){}      // DefConst 클래스의 기본 생성자
```

-   매개변수가 있는 생성자
    -   메서드처럼 매개변수를 통해 호출 시 해당 값을 받아 인스턴스를 초기화
    -   인스턴스마다 다른 값을 가지고 초기화가 가능하여 고유한 특성을 지닌 인스턴스를 반복 생성시 유용함

```
public class ConstructorExam {
  public static void main(String[] args) {
    Computer com = new Computer("Windows", "하얀색");
    System.out.println("제 PC의 운영체제는 " + com.getOsName() + "이고, 컬러는 " + com.getColor() + "입니다.");
  }
}

class Computer {
  private String osName;
  private String color;

  public Car(String osName, color) {
    this.osName = osName;
    this.color = color;
  }

  public String getOsName() {
    return osName;
  }

  public String getColor() {
    return color;
  }
}

// 출력 결과
// 제 PC의 운영체제는 Windows이고, 컬러는 하얀색입니다.
```

-   this()
    -   자신이 속한 클래스에서 다른 생성자를 호출할 때 사용
    -   this() 메서드를 사용하기 위한 두 가지 문법 요소
        -   반드시 생성자의 내부에서만 사용 가능
        -   반드시 생성자의 첫 줄에 위치

```
pubilc class TestCase {
  public static void main(String[] args) {
    Exam exam = new Exam();
    Exam exam = new Exam(1);
  }
}

class Exam {
  public Exam() {
    System.out.println("Exam의 기본 생성자");
  }

  public Exam(int x) {
    this();
    System.out.println("Exam의 두 번째 생성자");
  }
}

// 출력 결과
// Exam의 기본 생성자
// Exam의 기본 생성자
// Exam의 두 번째 생성자
```

-   this 키워드
    -   자신이 포함된 클래스의 객체를 가리키는 참조 변수
    -   인스턴스의 필드명과 지역변수를 구분하기 위해 사용

---

## Java의 내부 클래스

> 클래스 내에 선언된 클래스로 외부 클래스와 내부 클래스가 서로 연관되어 있을 때 사용한다.
> 
> 외부 클래스로의 접근이 용이하고 코드의 복잡성이 감소한다.
> 
> 캡슐화(Encapsulation)에 용이하다.
> 
> 인스턴스 내부 클래스, 정적 내부 클래스 , 지역 내부 클래스로 구분한다.

```
class OuterClass {    // 외부 클래스

  class InnerClass {
    // 인스턴스 내부 클래스
  }

  static class StaticInnerClass {
    // 정적 내부 클래스
  }

  void play(){
    // 지역 내부 클래스
  }
}
```

|   종류   |   선언 위치   |   사용 가능한 변수   |
| :-: | :-: | :-: |
|   인스턴스 내부 클래스   (instance inner class)   |   외부 클래스의 멤버변수 선언 위치에 선언(멤버 내부 클래스)   |   외부 인스턴스, 외부 전역변수   |
|   정적 내부 클래스   (static inner class)   |   외부 클래스의 멤버변수 선언 위치에 선언(멤버 내부 클래스)   |   외부 전역변수   |
|   지역 내부 클래스   (local inner class)   |   외부 클래스의 메서드나 초기화 블럭 안에 선언   |   외부 인스턴스, 외부 전역변수   |
|   익명 내부 클래스   (anonymous inner class)   |   클래스의 선언과 객체의 생성을 동시에 하는 일회용 익명 클래스   |   외부 인스턴스, 외부 전역변수   |

### 멤버 내부 클래스

-   인스턴스 내부 클래스
    -   객체 내부에 멤버의 형태로 존재
    -   외부 클래스의 모든 접근 지정자의 멤버에 접근 가능
-   정적 내부 클래스
    -   내부 클래스가 외부 클래스와 무관하게 정적 변수를 사용할 수 있도록 사용
    -   인스턴스 내부 클래스와 비슷하지만 static 키워드를 사용한다는 것이 차이점
-   지역 내부 클래스
    -   클래스의 멤버가 아닌 메서드 내에서 정의되는 클래스
    -   메서드 안에서 선언 후 바로 객체를 생성하여 사용