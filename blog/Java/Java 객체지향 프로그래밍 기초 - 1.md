태그: #Java 
연결문서: [Java 객체지향 프로그래밍 기초 - 2](Java%20객체지향%20프로그래밍%20기초%20-%202.md)

# Java의 객체지향 프로그래밍 기초

## 객체지향 프로그래밍

> 모든 실재하는 어떤 대상을 프로그래밍 언어에서는 객체(Object)라고 부른다. 객체지향 프로그래밍(OOP, Object Oriented Programming)의 개념은 이 객체에서 시작된다. 객체지향이론의 핵심 개념은 실제 세계가 객체들로 구성되어 있고 발생하는 모든 사건들은 객체들의 상호작용을 통해 발생한다는 것을 전제로 한다.
> 
> 객체지향 프로그래밍은 "프로그래밍에서 필요한 데이터를 한 데 모아 추상화시켜 상태와 행위를 가진 객체를 만들고 그 객체들 간의 유기적인 상호작용을 통해 특정 기능을 구성"하는 프로그래밍 방법론을 지칭한다.
> 
> 즉, 실제 사물의 속성(state)과 기능(behavior)을 분석한 후 이것을 프로그래밍의 변수와 함수로 정의함으로 컴퓨터 프로그래밍에 반영하고 실제 세계와 유사한 가상 세계를 구현하고자 하는 것이다.

-   객체지향언어의 특징

1.  코드의 재사용성이 높음
    -   새로운 코드를 작성할 때 기존 코드를 활용하여 쉽게 작성 가능
2.  코드의 관리가 쉬움
    -   코드간의 관계를 이용해 쉽게 코드의 변경 가능
3.  프로그래밍의 신뢰성이 높음
    -   제어자와 메서드를 이용해 데이터를 보호하고 올바른 값을 유지하도록 함
    -   코드의 중복을 제거하여 오작동을 방지

## Java의 클래스와 객체

### Java의 클래스

> 클래스(Class)란 객체를 정의한 설계도 또는 틀이라고 정의할 수 있다.
> 
> 클래스를 통해 생성된 객체를 인스턴스라 부르며 클래스로 객체를 만드는 과정을 객체의 인스턴스화라 일컫는다.

-   클래스의 구성 요소와 기본 문법

1.  필드 : 클래스의 속성을 나타내는 변수
2.  메서드 : 클래스의 기능을 타나내는 함수
3.  생성자 : 클래스의 객체를 생성하는 역할
4.  이너 클래스 : 클래스 내부의 클래스

```
public class Example{
  int num = 5;          // 필드
  void printX() {...}   // 메서드
  Example {...}         // 생성자
  class Example2 {...}  // 이너 클래스
}
```

### Java의 객체

> 프로그래밍에서 객체는 클래스에 정의된 내용대로 메모리에 생성된 것을 의미한다.
> 
> > 객체의 정의 : 실제로 존재하는 사물 또는 개념  
> > 객체의 용도 : 객체가 가지고 있는 기능과 속성에 따라 다름

-   객체의 구성 요소
    -   속성과 기능이라는 두 가지 구성 요소로 이뤄짐
    -   속성과 기능은 클래스에서 필드와 메서드로 정의되며 일반적으로 하나의 객체는 다양한 기능의 집합으로 이뤄져 있으며 이런 속성과 기능을 객체의 멤버라고 부름
    -   클래스로 객체를 생성하면 클래스에 정의된 속성과 기능을 가진 객체가 생성
    -   속성(property) : 멤버변수(member variable), 특성(attribute), 필드(field), 상태(state)
    -   기능(function) : 메서드(method), 함수(function), 행위(behavior)

-   객체의 생성과 활용

```
// 객체의 생성
클래스명 참조_변수명;        // 인스턴스를 참조하기 위한 참조변수 선언
참조_변수명 = new 생성자();  // 인스턴스 생성 후, 객체의 주소를 참조변수에 저장

Computer com;             // Computer클래스의 참조변수 com을 선언
com = new Computer();     // Computer인스턴스 생성하고 생성된 인스턴스의 주소를 com에 저장

Computer com = new Computer();  // 간략화

// 객체의 활용
참조_변수명.필드명      // 필드값 불러오기
참조_변수명.메서드명()  // 메서드 호출
```

---

## Java의 필드와 메서드

### Java의 필드

> 필드는 클래스에 포함된 변수를 의미하며 객체의 속성읠 정의할 때 사용하며 클래스 변수와 인스턴스 변수로 구분한다.
> 
> 변수는 선언된 위치에 따라 종류가 결정되며 각자 다른 유효 범위(scope)를 가진다.

```
class Example{        // 클래스 영역
    int iv;           // 인스턴스 변수
    static int cv;    // 클래스 변수

    void method{      // 메서드 영역
          int lv = 5;   // 지역 변수 - 메서드 영역 안에서만 유효
    }
}
```

-   인스턴스 변수 (iv, instance variable)
    -   인스턴스가 가지는 고유한 속성을 저장하기 위한 변수
    -   static키워드가 없이 선언된 변수
    -   new 생성자를 통해 인스턴스가 생성될 때 만들어짐
    -   힙 메모리의 독립적 공간에 저장되며 객체의 고유한 개별성을 지님
    -   Ex) 성별, 이름, 나이

-   클래스 변수 (cv, class variable)
    -   공통된 저장공간을 공유하는 변수
    -   static키워드와 함께 선언된 변수
    -   특정 클래스에서 생성되는 모든 인스턴스가 특정한 값을 공유해야할 때 사용
    -   인스턴스를 생성하지 않아도 '클래스.클래스변수명'을 통해 사용 가능
    -   Ex) 눈의 개수, 심장의 개수

-   지역 변수
    -   메서드 내에 선언되며 메서드 내에서만 사용 가능한 변수
    -   멤버 변수(cv, iv)와 달리 스텍 메모리에 저장되며 메서드 종료 시 소멸
    -   일정 기간 사용하지 않으면 가상 머신에 의해 자동으로 삭제
    -   멤버 변수와 달리 직접 초기화하지 않으면 값을 출력할 때 오류가 발생

### static 키워드

> static은 클래스의 멤버(필드, 메서드, 이너 클래스)에 사용되는 키워드이다.
> 
> static 키워드가 붙어있는 멤버를 '정적 멤버(static member)'라고 부른다.
> 
> static 키워드로 선언된 정적 멤버는 객체 생성없이 바로 사용 가능하다.

```
public class StaticCase {
  public static void main(String[] args){
    StaticExam staticExam = new StaticExam();
    System.out.println("iv = " + staticExam.first);   // 인스턴스 변수
    System.out.println("cv = " + StaticExam.second);  // 클래스 변수 - 인스턴스 생성 없이 사용
  }
}

class StaticExam {
  int first = 10;
  static int second = 20;
}

// 출력 결과
iv = 10
cv = 20
```

### Java의 메서드

> 메서드는 특정 작업을 수행하는 일련의 명령문들의 집합을 의미한다.
> 
> 메서드 시그니처(method signature)와 메서드 바디(method body)로 구분한다.
> 
> 메서드의 반환타입이 void가 아니면 메서드 바디에는 반드시 return문이 존재해야 한다.

```
// 자바제어자 반환타입 메서드명(매개 변수)
public static int multi(int a, int b){    // 메서드 시그니처

  // 메서드 내용 = 메서드 바디
  int result = a * b;
  return result;
}
```

### 메서드 오버로딩

> 하나의 클래스 안에 같은 이름의 메서드를 여러 개 정의하는 것이다.
> 
> 같은 이름의 메서드명을 사용하며, 매개변수의 개수나 타입이 다르게 정의되어야 한다.
> 
> 하나의 메서드로 여러 경우의 수를 해결할 수 있다.
> 
> Ex) println 메서드

```
public class MethodOverloading {
  public static void main(String[] args){
    Sum sum = new Sum();  // 객체 생성

    sum.add();            // 메서드 호출
    sum.add(5, 5);
    sum.add(5, 5L);
    sum.add(5L, 5);
    sum.add(5L, 5L);
  }
}

class Sum {
  public void add(){      // 동일한 이름의 메서드 5개를 정의
    System.out.println("입력된 값이 없습니다.");
  }
  public void add(int a, int b){
    System.out.println("int a + int b = " + int a + int b);
  }
  public void add(int a, long b){
    System.out.println("int a + long b = " + int a + long b);
  }
  public void add(long a, int b){
    System.out.println("long a + int b = " + long a + int b);
  }
  public void add(long a, long b){
    System.out.println("long a + long b = " + long a + long b);
  }
}

// 출력 결과는 모두 10으로 같으나 사용된 메서드의 타입이 다름
입력받은 값이 없습니다.
int a + int b = 10
int a + long b = 10
long a + int b = 10
long a + long b = 10
```
