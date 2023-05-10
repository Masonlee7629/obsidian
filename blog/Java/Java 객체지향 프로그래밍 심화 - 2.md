태그: #Java 
연결문서: [Java 객체지향 프로그래밍 심화 - 3](Java%20객체지향%20프로그래밍%20심화%20-%203.md)

## Java의 캡슐화

### 캡슐화

> 특정 객체 안에 관련된 속성과 기능을 하나의 캡슐로 만들어 데이터를 외부로부터 보호하는 것

-   데이터 보호화 내부적으로 사용되는 데이터의 불필요한 외부 노출을 방지
-   유지보수와 코드 확장 시 오류의 범위를 최소화
-   캡슐화의 핵심 수단으로는 접근제어자(Access Modifier), getter와 setter 메서드가 존재

### 패키지(Package)

> 특정한 목적을 공유하는 클래스와 인터페이스의 묶음이다.

-   패키지는 물리적인 디렉토리이고, 클래스나 인터페이스 파일은 해당 패키지에 속함
-   패키지가 있을 경우 소스 코드의 첫 번째 줄에 반드시 패키지명이 표시되어야 함
-   클래스의 충돌을 방지해주는 기능을 보유
-   Ex) java.lang, java.util, java.io 등

### import문

> 다른 패키지 내의 클래스를 사용하기 위해 사용한다.

```
// test1 패키지
package packageexam.test1;

public class ImportExam {
  public int a = 0;
  public void print(){
    System.out.println("Import문 출력");
  }
}

// test2 패키지
package packageexam.test2;

import packgeexam.test1.ImportExam;    // Import문 작성

public class PackegeExam{
  public static void main(String[] args){
    ImportExam imp = new ImportExam();  //  test패키지의 class를 활용
  }
}
```

### 제어자(Modifier)

> 클래스, 필드, 메서드 ,생성자 등에 부가적인 의미를 부여하는 키워드이다.

-   접근 제어자와 기타 제어자로 구분
-   하나의 대상에 여러 제어자를 사용할 수 있지만 접근 제어자는 단 한 번만 사용 가능

-   접근 제어자(Access Modifier)
    -   클래스 외부로의 불필요한 데이터 노출과 데이터가 임의로 변경되지 않도록 방지
    -   private, default, protected, public으로 분류
    -   변수명 앞에 접근 제어자가 없는 경우 접근 제어자는 자동으로 default

|   접근 제어자   |   접근 제한 범위   |   클래스 내   |   패키지 내   |   다른 패키지의 하위 클래스   |   패키지 외   |
| :-: | :-: | :-: | :-: | :-: | :-: |
|   private   |   동일 클래스에서만 접근 가능   |   O   |   X   |   X   |   X   |
|   default   |   동일 패키지 내에서만 접근 가능   |   O   |   O   |   X   |   X   |
|   protected   |   동일 패키지와 다른 패키지의 하위 클래스에서   접근 가능   |   O   |   O   |   O   |   X   |
|   public   |   접근 제한 없음   |   O   |   O   |   O   |   O   |

```
packge packagetest;   // 패키지명 packagetest

// 파일명 : Parent.java
class Exam {    // 접근 제어자 : default
  public static void main(String[] args){
    Parent p = new Parent();

    // System.out.println(p.a);    // 동일 클래스가 아니므로 에러 발생
    System.out.println(p.b);
    System.out.println(p.c);
    System.out.println(p.d);
  }
}

public class Parent {   // 접근 제어자 : public
  private String a = "Private";
  String b = "Default";
  protected String c = "Protected";
  public String d = "Public";

  public void print2(){     // 동일 클래스로 모두 정상 출력
    System.out.println(p.a);
    System.out.println(p.b);
    System.out.println(p.c);
    System.out.println(p.d);
  }
}

// test 클래스 출력 결과
// "Default"
// "Protected"
// "Public"
```

### getter와 setter 메서드

> 대표적으로 private 접근 제어자가 포함된 객체의 변수의 데이터 값을 수정해야할 때 사용한다.

-   setter 메서드는 외부에서 메서드에 접근하여 조건에 맞을 경우 데이터 값을 변경
-   getter 메서드는 설정한 변수 값을 읽어오는 데 사용
-   메서드명에 get-, set- 을 붙여서 정의

```
public class GetterSetter {
  public static void main(){
    Tester t = new Tester();
    t.setName("Mason");
    t.setAge(32);
    t.setId("masonlee7629");

    String name = t.getName();
    System.out.println("테스터의 이름은 " + name);
    int age = t.getAge();
    System.out.println("테스터의 나이는 " + age);
    String id = t.getId();
    System.out.println("테스터의 ID는 " + id);
  }
}

class Tester {
  private String name;
  private int age;
  private String id;

  public String getName(){    // 멤버변수의 값
    return name;
  }

  public void setName(String name){   // 멤버변수의 값 변경
    this.name = name;
  }

  public int getAge(){
    return age;
  }

  public void setAge(int age){
    this.age = age;
  }

  public String getId(){
    return id;
  }

  public void setId(String id){
    this.id = id;
  }
}

// 출력 결과
// 테스터의 이름은 Mason
// 테스터의 나이는 32
// 테스터의 ID는 masonlee7629
```