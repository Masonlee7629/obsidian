태그: #Java 
연결문서: [Java 객체지향 프로그래밍 심화 - 2](Java%20객체지향%20프로그래밍%20심화%20-%202.md)

## Java의 상속

### 상속

> 기존의 클래스를 재활용하여 새로운 클래스를 작성하는 자바의 문법 요소이다.

-   상위 클래스의 멤버를 하위 클래스와 공유하는 것을 의미
-   extends 키워드를 사용하여 정의
-   단일 상속만을 허용

```
class Parent {
  // 상위 클래스
}

class Child extends Parent {
  // 하위 클래스, Parent 클래스를 상속받는 Child 클래스
}
```

### 포함 관계

> 포함(composite)은 클래스의 멤버로 다른 클래스 타입의 참조변수를 선언하는 것을 의미한다.

-   클래스 간의 관계가 '-은 -이다(IS-A)' 관계는 상속, '-은 -을 가지고 있다(HAS-A)' 관계는 포함
-   단위별 여러 개의 클래스는 코드가 작게 나눠 작성되어 있으므로 코드를 관리하는데 용이함

```
class Dot {
  int a;
  int b;
}

class Circle {      // Circle 클래스에 int a, b가 있을 경우 Dot을 재사용함
  Dot d = new Dot();
  int c;
}
```

### 메서드 오버라이딩

> 상위 클래스에서 상속 받은 메서드의 내용을 변경하는 것이다.

-   메서드 오버라이딩의 사용 조건
    -   메서드의 선언부(메서드명, 매개변수, 반환타입)가 상위 클래스와 일치해야 함
    -   접근 제어자의 범위가 상위 클래스의 메서드보다 크거나 동일해야 함
    -   예외는 상위 클래스의 메서드보다 많이 선언할 수 없음
-   모든 객체를 상위 클래스 타입으로 선언하면 상위 클래스에서 배열로 선언하여 관리 가능

```
class Transportation {
  void go(){
    System.out.println("I'm going");
  }
}

public class Bike extends Transportation {
  void go(){
    System.out.println("I ride a bike");
  }

  public static void main(String[] args){
    Bike bike = new Bike();
    bike.run();
  }
}

// 출력 결과
// "I ride a bike"
```

### SUPER 키워드

> 상위 클래스의 객체를 호출하는 것이다.

-   상위 클래스와 변수의 이름이 같을 때 두 변수를 구분하기 위해 사용
-   this 키워드와 비슷하지만 상위 클래스의 멤버와 자신의 멤버를 구별하는 것이 차이점

```
class SuperExam {
  public static void main(String[] args){
    Child child = new Child();
    child.test();
  }
}

class Parent {
  int a = 5;
}

class Child extends Parent {
  int a = 10;

  void test(){
    System.out.println("a = " + a);
    System.out.println("this.a = " + this.a);
    System.out.println("super.a = " + super.a);
  }
}

// 출력 결과
// a = 10
// this.a = 10
// super.a = 5
```

### super 메서드

> 상위 클래스의 생성자를 호출하는 것이다.

-   생성자 안에서만 사용 가능하며 모든 생성자의 첫 줄에 반드시 this() 또는 super()를 선언
-   상위 클래스에 기본 생성자가 없으면 에러를 발생

```
public class Exam {
  public static void main(String[] args){
    Teacher t = new Teacher();
  }
}

class Human {
  Human(){
    System.out.println("Human 클래스 생성자");
  }
}

class Teacher extends Human{    // Human 클래스로 확장
  Teacher(){
    super();    // Human 클래스의 생성자 호출, 없을 경우 컴파일러가 생성자의 첫 줄에 자동으로 super()를 삽입
    System.out.println("Teacher 클래스 생성자");
  }
}

// 출력 결과
// Human 클래스 생성자
// Teacher 클래스 생성자
```

### Object 클래스

> 자바의 클래스 상속계층도에서 최상위에 위치한 클래스이다

-   다른 클래스로부터 아무 상속을 받지 않는 클래스는 자동적으로 Object 클래스를 상속

```
class Parent {    // 컴파일러가 "extends Object'를 자동적으로 추가
}

class Child extends Parent {
}
```

### Object 클래스의 대표적인 메서드

|   메서드명   |   반환타입   |   메서드 특징   |
| :-: | :-: | :-: |
|   toString()   |   String   |   객체 정보를 문자열로 출력   |
|   equal(Object obj)   |   boolean   |   등가 비교 연산과 동일하게 스택 메모리값을 비교   |
|   hashCode()   |   int   |   객체의 위치정보 관련. HashTable 또는 HashMap에서 동일 객체 여부를 판단   |
|   wait()   |   void   |   현재 쓰레드 일시정지   |
|   notify()   |   void   |   일시정지 중인 쓰레드를 재동작   |
